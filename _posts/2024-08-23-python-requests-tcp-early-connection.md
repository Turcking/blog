---
layout: post
title: python requests 库 tcp 提前连接
date: 2024-08-23
Author: Turcking
categories: 
tags: [python, requests]
comments: false
toc: false
---

**注意：本文使用 requests 版本 2.32.3，urllib3 版本 1.26.19，python 版本 3.12.4，运行环境 archlinux**

网上搜抢课脚本的时候看到 [https://www.zhihu.com/question/268200514](https://www.zhihu.com/question/268200514) 一个回答说有可能服务器 tcp 握手会握不上，所以为了将来写脚本就研究了下。

首先要知道 requests 的 session 会在允许的情况下保存 tcp 连接，就是 http/1.1 长连接。
当头部都拥有 `Connection: keep-alive` 时，tcp 连接可以不断开，客户端可以在这个连接上再次发送请求，比短连接少了 tcp 握手阶段。
所以 session 为了保持长连接，提高爬虫运行效率，会管理建立的连接。

我的目标就是在我的代码发送请求之前，先建立好 tcp 连接。等到需要发送请求的时候，再使用这条连接发送请求。
但是我在网上没有找到实现这个目标的方法，也没在 requests 里找到什么简单的方法可以直接做到~~（要是有就是我眼瞎）~~。

实际上要在介绍长连接的地方也说了，只要双方都有 `Connection: keep-alive` tcp 连接就不会被 session 断开。
所以**如果没有特殊需求的话，可以直接放一个不搭嘎的请求**就能提前建立连接了。

还有，**该解决方法只建议短期使用**~~（应该说因为它使用了当前 requests 版本的特性吗）~~。

接下来是正文，就是研究记录：

首先看 `requests/session.py` 中 `Session` 的 `request()` 方法：
```python
def request(...):
    # Create the Request.
    req = Request(...)
    prep = self.prepare_request(req)

    proxies = proxies or {}

    settings = self.merge_environment_settings(
        prep.url, proxies, stream, verify, cert
    )

    # Send the request.
    send_kwargs = {
        "timeout": timeout,
        "allow_redirects": allow_redirects,
    }
    send_kwargs.update(settings)
    resp = self.send(prep, **send_kwargs)

    return resp
```

注意 `merge_environment_settings()` 以及 `requests.session.Session` 的构造函数：
```python
class Session(SessionRedirectMixin):
    ...
    def __init__(self):
        ...
        self.verify = True
        ...
    def merge_environment_settings(self, url, proxies, stream, verify, cert):
        ...
        # Merge all the kwargs.
        proxies = merge_setting(proxies, self.proxies)
        stream = merge_setting(stream, self.stream)
        verify = merge_setting(verify, self.verify)
        cert = merge_setting(cert, self.cert)

        return {"proxies": proxies, "stream": stream, "verify": verify, "cert": cert}
```

所以如果 `verify` 没有设置的话会变成 `True`，这里先记着，之后要用。

`request()` 最终使用 `send()` ，所以来看 `send()`：
```python
def send(...):
    ...
    # Get the appropriate adapter to use
    adapter = self.get_adapter(url=request.url)
    ...
    # Send the request
    r = adapter.send(request, **kwargs)
    ...
    return r
```

然后 `get_adapter()`：
```python
def get_adapter(self, url):
    ...
    for prefix, adapter in self.adapters.items():
        if url.lower().startswith(prefix.lower()):
            return adapter
    ...
```

这里 `self.adapters` 在构造函数最后三行：
```python
def __init__(self):
    ...
    # Default connection adapters.
    self.adapters = OrderedDict()
    self.mount("https://", HTTPAdapter())
    self.mount("http://", HTTPAdapter())
```

所以 `get_adapter()` 的 `url` 写 `"http://"` 或者 `"https://"` 就行了。
然后再来看 `get_adapter()` 取回来的 `HTTPAdapter` 的 `send()`，来自 `requests/adapters.py`：
```python
def send(...):
    ...
    try:
        conn = self.get_connection_with_tls_context(
            request, verify, proxies=proxies, cert=cert
        )
    ...
    try:
        resp = conn.urlopen(
            method=request.method,
            url=url,
            body=request.body,
            headers=request.headers,
            redirect=False,
            assert_same_host=False,
            preload_content=False,
            decode_content=False,
            retries=self.max_retries,
            timeout=timeout,
            chunked=chunked,
        )
    ...
```

于是来看 `HTTPAdapter` 的 `get_connection_with_tls_context()`：
```python
def get_connection_with_tls_context(...):
    ...
    try:
        host_params, pool_kwargs = self.build_connection_pool_key_attributes(
            request,
            verify,
            cert,
        )
    ...
    if proxy:
        ...
        conn = proxy_manager.connection_from_host(
            **host_params, pool_kwargs=pool_kwargs
        )
    else:
        # Only scheme should be lower case
        conn = self.poolmanager.connection_from_host(
            **host_params, pool_kwargs=pool_kwargs
        )

    return conn
```

还记得需要记着的 `verify` 默认为 `True` 吗，这里 `build_connection_pool_key_attributes()` 用到它了：
```python
def build_connection_pool_key_attributes(self, request, verify, cert=None):
    ...
    return _urllib3_request_context(request, verify, cert, self.poolmanager)
```

这个方法除了注释就只是调用 `_urllib3_request_context()`，`_urllib3_requests_context()` 在 `requests/adapters.py` 中：
```python
def _urllib3_request_context(
    request: "PreparedRequest",
    verify: "bool | str | None",
    client_cert: "typing.Tuple[str, str] | str | None",
    poolmanager: "PoolManager",
) -> "(typing.Dict[str, typing.Any], typing.Dict[str, typing.Any])":
    ...
    poolmanager_kwargs = getattr(poolmanager, "connection_pool_kw", {})
    has_poolmanager_ssl_context = poolmanager_kwargs.get("ssl_context")
    should_use_default_ssl_context = (
        _preloaded_ssl_context is not None and not has_poolmanager_ssl_context
    )

    cert_reqs = "CERT_REQUIRED"
    if verify is False:
        cert_reqs = "CERT_NONE"
    elif verify is True and should_use_default_ssl_context:
        pool_kwargs["ssl_context"] = _preloaded_ssl_context
    elif isinstance(verify, str):
        if not os.path.isdir(verify):
            pool_kwargs["ca_certs"] = verify
        else:
            pool_kwargs["ca_cert_dir"] = verify
    pool_kwargs["cert_reqs"] = cert_reqs
    ...
    return host_params, pool_kwargs
```

这里 `poolmanager` 如果 requests 用户没有设置，是没有 `ssl_context` 的，所以 `pool_kwargs` 会使用这个 `_preloaded_ssl_context`，它就在 `_urllib3_requests_context()` 的上方定义：
```python
try:
    import ssl  # noqa: F401

    _preloaded_ssl_context = create_urllib3_context()
    _preloaded_ssl_context.load_verify_locations(
        extract_zipped_paths(DEFAULT_CA_BUNDLE_PATH)
    )
except ImportError:
    # Bypass default SSLContext creation when Python
    # interpreter isn't built with the ssl module.
    _preloaded_ssl_context = None
```

所以最终 `pool_kwargs` 至少会设置 `{"cert_reqs": "CERT_REQUIRED", "ssl_context": requests.adapters._preloaded_ssl_context}`，之后 `connection_from_host()` 中要放进去。
这个 `pool_kwargs` 非常重要，特别是对连接有特别需求的，可以仔细分析 `_urllib3_requests_context()`~~（因为我之前没有看到这个，导致结果怎么都不对，具体为什么后面有说）~~。

回到 `HTTPAdapter().get_connection_with_tls_context()`，这里不考虑代理，`self.poolmanager` 由构造函数通过 `self.init_poolmanager()` 创建：
```python
def init_poolmanager(...):
    ...
    self.poolmanager = PoolManager(...)
```

这里的 `PoolManager` 是 `urllib3.poolmanager.PoolManager`，所以来看它的 `connection_from_host()`，来自 `urllib3/poolmanager.py`，顺带下面有几个方法一起看了：
```python
def connection_from_host(self, host, port=None, scheme="http", pool_kwargs=None):
    ...
    return self.connection_from_context(request_context)

def connection_from_context(self, request_context):
    ...
    return self.connection_from_pool_key(pool_key, request_context=request_context)

def connection_from_pool_key(self, pool_key, request_context=None):
    """
    Get a :class:`urllib3.connectionpool.ConnectionPool` based on the provided pool key.

    ``pool_key`` should be a namedtuple that only contains immutable
    objects. At a minimum it must have the ``scheme``, ``host``, and
    ``port`` fields.
    """
    with self.pools.lock:
        # If the scheme, host, or port doesn't match existing open
        # connections, open a new ConnectionPool.
        pool = self.pools.get(pool_key)
        if pool:
            return pool

        # Make a fresh ConnectionPool of the desired type
        scheme = request_context["scheme"]
        host = request_context["host"]
        port = request_context["port"]
        pool = self._new_pool(scheme, host, port, request_context=request_context)
        self.pools[pool_key] = pool

    return pool
```

可以看见最终从 `self.pools` 取一个连接池，没有就使用 `self._new_pool()` 创建一个。
`pools` 来自构造函数：
```python
from ._collections import HTTPHeaderDict, RecentlyUsedContainer
...
class PoolManager(RequestMethods):
    ...
    def __init__(self, num_pools=10, headers=None, **connection_pool_kw):
        RequestMethods.__init__(self, headers)
        self.connection_pool_kw = connection_pool_kw
        self.pools = RecentlyUsedContainer(num_pools)
```

然后 `urllib3/_collections.py` 里的 `RecentlyUsedContainer`：
```python
from collections import OrderedDict
...
class RecentlyUsedContainer(MutableMapping):
    ...
    ContainerCls = OrderedDict

    def __init__(self, maxsize=10, dispose_func=None):
        ...
        self._container = self.ContainerCls()
```

这个 `_container` 就是它的容器，是一个字典，键将是连接信息，所以之后写的代码连接信息必须一致，可以随时将其打印出来做调试。

回到 `connection_from_host()`，看看创建连接池的 `_new_pool()`：
```python
def _new_pool(self, scheme, host, port, request_context=None):
    ...
    pool_cls = self.pool_classes_by_scheme[scheme]
    ...
    return pool_cls(host, port, **request_context)
```

然后这个 `self.pool_classes_by_scheme`：
```python
from .connectionpool import HTTPConnectionPool, HTTPSConnectionPool, port_by_scheme
...
pool_classes_by_scheme = {"http": HTTPConnectionPool, "https": HTTPSConnectionPool}


class PoolManager(RequestMethods):
    ...
    def __init__(self, num_pools=10, headers=None, **connection_pool_kw):
        ...
        self.pool_classes_by_scheme = pool_classes_by_scheme
        ...
```

咱这里用 HTTP，所以最终 `HTTPAdapter().get_connection_with_tls_context()` 会返回的是 `urllib3.connectionpool.HTTPConnectionPool()`。

到这里它还只是创建了连接池，并没有连接，所以回到 `HTTPAdapter().send()`：
```python
def send(...):
    ...
    try:
        conn = self.get_connection_with_tls_context(
            request, verify, proxies=proxies, cert=cert
        )
    ...
    try:
        resp = conn.urlopen(...)
    ...
```

所以最终调用 `HTTPConnectionPool().urlopen()`，所以看来自 `urllib3/connectionpool.py` 的 `HTTPConnectionPool().urlopen()`：
```python
def urlopen(...):
    ...
    try:
        # Request a connection from the queue.
        timeout_obj = self._get_timeout(timeout)
        conn = self._get_conn(timeout=pool_timeout)
        ...
        # Make the request on the httplib connection object.
        httplib_response = self._make_request(
            conn,
            method,
            url,
            timeout=timeout_obj,
            body=body,
            headers=headers,
            chunked=chunked,
        )
        ...
    ...
    finally:
        ...
        if release_this_conn:
            # Put the connection back to be reused. If the connection is
            # expired then it will be None, which will get replaced with a
            # fresh connection during _get_conn.
            self._put_conn(conn)
    ...
```

使用 `_get_conn()` 获取了连接，使用 `_make_request()` 发送请求，最后再用 `_put_conn()` 将连接放回池中。先看 `_get_conn()`：
```python
def _get_conn(self, timeout=None):
    ...
    conn = None
    try:
        conn = self.pool.get(block=self.block, timeout=timeout)
    ...
    return conn or self._new_conn()

```

如果 `pool` 中没有连接就会通过 `_new_conn()` 创建一个，连着环境一起看 `_new_conn()`：
```python
from .connection import (
    BaseSSLError,
    BrokenPipeError,
    DummyConnection,
    HTTPConnection,
    HTTPException,
    HTTPSConnection,
    VerifiedHTTPSConnection,
    port_by_scheme,
)
...
class HTTPConnectionPool(ConnectionPool, RequestMethods):
    ...
    ConnectionCls = HTTPConnection
    ...
    def _new_conn(self):
        ...
        conn = self.ConnectionCls(
            host=self.host,
            port=self.port,
            timeout=self.timeout.connect_timeout,
            strict=self.strict,
            **self.conn_kw
        )
        return conn
```

最终返回 `urllib3.connection.HTTPConnection()`。然后来看上面 `urllib3.connectionpool.HTTPConnectionPool().urlopen()` 里提到的 `_make_request()`：
```python
def _make_request(
    self, conn, method, url, timeout=_Default, chunked=False, **httplib_request_kw
):
    ...
    # Trigger any extra validation we need to do.
    try:
        self._validate_conn(conn)
    except (SocketTimeout, BaseSSLError) as e:
        # Py2 raises this as a BaseSSLError, Py3 raises it as socket timeout.
        self._raise_timeout(err=e, url=url, timeout_value=conn.timeout)
        raise

    # conn.request() calls http.client.*.request, not the method in
    # urllib3.request. It also calls makefile (recv) on the socket.
    try:
        if chunked:
            conn.request_chunked(method, url, **httplib_request_kw)
        else:
            conn.request(method, url, **httplib_request_kw)
```

这里 `_validate_conn()` 只有个 `pass`，主要看下面的 `request()`。来自 `urllib3/connection.py` 的 `request()`：
```python3
from .packages.six.moves.http_client import HTTPConnection as _HTTPConnection
...
class HTTPConnection(_HTTPConnection, object):
    ...
    def request(self, method, url, body=None, headers=None):
        ...
        super(HTTPConnection, self).request(method, url, body=body, headers=headers)
```

这个 `urllib3.packages.six.moves` 里面我没看懂，但是推测 `urllib3.packages.six.moves.http_client` 就是 `http.client`。
所以看来自 `http/client.py` 的 `request()`：
```python
def request(self, method, url, body=None, headers={}, *,
            encode_chunked=False):
    """Send a complete request to the server."""
    self._send_request(method, url, body, headers, encode_chunked)

def _send_request(self, method, url, body, headers, encode_chunked):
    ...
    for hdr, value in headers.items():
        self.putheader(hdr, value)
    if isinstance(body, str):
        # RFC 2616 Section 3.7.1 says that text default has a
        # default charset of iso-8859-1.
        body = _encode(body, 'body')
    self.endheaders(body, encode_chunked=encode_chunked)
```

然后看 `endheaders()`：
```python
def endheaders(self, message_body=None, *, encode_chunked=False):
    ...
    self._send_output(message_body, encode_chunked=encode_chunked)
```

然后是 `_send_output()`：
```python
def _send_output(self, message_body=None, encode_chunked=False):
    ...
    self.send(msg)

    if message_body is not None:
        ...
        for chunk in chunks:
            ...
            self.send(chunk)

        if encode_chunked and self._http_vsn == 11:
            # end chunked transfer
            self.send(b'0\r\n\r\n')
```

然后看反复出现的 `send()`：
```python
def send(self, data):
    ...
    if self.sock is None:
        if self.auto_open:
            self.connect()
```

这里就可以看到最关键的 `connect()`，由于之前是从子类 `urllib3.connection.HTTPConnection` 调用过来的，所以要看子类的 `connect()`：
```python
def _prepare_conn(self, conn):
    self.sock = conn
    ...

def connect(self):
    conn = self._new_conn()
    self._prepare_conn(conn)
```

使用 `_new_conn()` 创建了一个连接，然后通过 `_prepare_conn()` 将连接放到 `self.sock` 上。

结合上面 `send()`，如果 `self.sock` 有连接就不会再创建一个连接。
所以只需要在先前调用它的 `connect()` 就可以提前建立连接了。

最后代码：
```python
#! /usr/bin/env python3

import requests
import requests.adapters
import time

if __name__ == "__main__":
    session = requests.session()

    # 这几条就是上面各个部分总结出来的代码
    adapter = session.get_adapter(url="http://")
    connection = adapter.poolmanager.connection_from_host("localhost", 8080, "http", pool_kwargs={
        "cert_reqs": "CERT_REQUIRED",
        "ssl_context": requests.adapters._preloaded_ssl_context
        })
    conn = connection._get_conn()
    conn.connect()
    connection._put_conn(conn)

    # 测试两次请求，每次关闭请求的结果不影响保持连接
    print("Send first request after 4 seconds.")
    time.sleep(4)
    session.get("http://localhost:8080/first_request").close()

    print("Send second request after 4 seconds.")
    time.sleep(4)
    session.get("http://localhost:8080/second_request").close()

    print("Close the session after 4 seconds.")
    time.sleep(4)
    # 关闭连接前打印一下所有连接池，如果有多个就代表没有成功
    for i in adapter.poolmanager.pools._container:
        print("\033[1m================================================================\033[0m")
        print(i)
    print("\033[1m================================================================\033[0m")
    session.close()
```

贴上我用来测试的服务端代码：
```python
#! /usr/bin/env python3

import socket, time, select, threading

def solve(connection, address):
    print("\033[1m%f:\033[0m" % time.time())
    print("%s: %d: Connect" % address)
    while True:
        select.select([connection], [], [])
        recvs = connection.recv(1000).decode()
        if len(recvs):
            connection.send(b"HTTP/1.1 200 Request OK\r\nConnection: keep-alive\r\nContent-Length: 0\r\n\r\n")
            print("\033[1m%f:\033[0m" % time.time())
            print(recvs)
        else:
            break
    connection.close()
    print("\033[1m%f:\033[0m" % time.time())
    print("%s: %d: Exit" % address)

if __name__ == "__main__":
    sock = socket.socket()
    sock.bind(("localhost", 8080))
    sock.listen()
    while True:
        connection, address = sock.accept()
        threading.Thread(target=solve, args=(connection, address)).start()
    sock.close()
```

最后附上运行结果，客户端：
```
We will send a request after 4 seconds.
We will send a new request after 4 seconds.
We will close the session after 4 seconds.
================================================================
PoolKey(key_scheme='http', key_host='localhost', key_port=8080, key_timeout=None, key_retries=None, key_strict=None, key_block=False, key_source_address=None, key_key_file=None, key_key_password=None, key_cert_file=None, key_cert_reqs='CERT_REQUIRED', key_ca_certs=None, key_ssl_version=None, key_ca_cert_dir=None, key_ssl_context=<ssl.SSLContext object at 0x7d28913dd050>, key_maxsize=10, key_headers=None, key__proxy=None, key__proxy_headers=None, key__proxy_config=None, key_socket_options=None, key__socks_options=None, key_assert_hostname=None, key_assert_fingerprint=None, key_server_hostname=None)
================================================================
```

服务端：
```
1724344353.871492:
127.0.0.1: 37966: Connect
1724344357.872566:
GET /first_request HTTP/1.1
Host: localhost:8080
User-Agent: python-requests/2.32.3
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive


1724344361.873776:
GET /second_request HTTP/1.1
Host: localhost:8080
User-Agent: python-requests/2.32.3
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive


1724344365.874679:
127.0.0.1: 37966: Exit
```

我发现我研究一个东西可能只需要一个下午，但是要写进博客就要花好几天的时间。就像这篇，原本 8 月 18 日就已经研究出来的，19 日去学校拿毕业证顺便和同学去商场逛了一圈，20 日开始落笔，到现在 23 日凌晨，写这句话想表达的意思就是我这种状况是否正常（otto 音）。

