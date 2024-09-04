---
layout: post
title: 常熟理工 2024 入学教育考试抓包总结
date: 2024-08-31
Author: Turcking
categories: 
tags: [web-crawler]
comments: false
toc: false
---

常熟理工学院要新生搞入学教育考试，用的微信小程序，而且切出去算考试中途离开，满两次算作弊。

![常熟理工学院 2024 级新生入学教育考试考卷信息截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-08-31-cslg-2024-entrance-education-exam-preview-snapshot.png)

于是就联手 Hmi 搞个题库（不如说全是他做的，因为我这研究的没用上），把试卷的题目都爬下来。

之前没研究过小程序的抓包，结果看到 [https://anjia0532.github.io/2021/01/24/wechat-mini-program-capture/](https://anjia0532.github.io/2021/01/24/wechat-mini-program-capture/ 微信小程序抓包的三种方式)。首先我没有 ios，安卓的试了一下，能抓但不能完全抓：一加 8T，安卓 11，浏览器（这里用的 Google Chrome，版本 91.0.4472.120）是可以抓到的，但是小程序就直接访问不了内容，抓包程序也一点反应都没有（我估计就是证书问题，也懒得搞），最后就在虚拟机里装了个微信来抓。

不过这个 Windows 版的抓包，微信登陆时设的代理是不作用于小程序的，要设系统代理。虚拟机是以前作研究的一个 Windows 7，系统代理藏得挺深。这个微信登陆界面的代理还卡了我挺久，微信本身的请求都出来了，小程序的就是不知道在哪，结果系统代理一设顿时清晰明了。不过也不知道是不是只有这个考试的小程序不走微信的代理，因为看到的那篇文章用的就是微信的代理。

目标是把题目都爬下来，但是没有那么多答卷（其实不是，哪有答卷后面再说），只有自己的卷子，所以先想爬已经做好的答卷。
做完给的二维码里的网址可以直接用浏览器查看，打开开发者工具再查看答题结果就可以发现这条请求：

![答卷结果 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-answer-result-api-snapshot.png)

![burpsuite 中答卷结果 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-answer-result-api-in-burpsuite-snapshot.png)

一个非常简单的 api，这个 `answerId` 就是答卷的 id 了。

但一天只能做一次，靠每天做的答卷来收集题目太慢了，于是想抓个包看看有没有什么漏洞可以钻，然后抓到了开头那张图片的网址（是的这个页面之前我只能通过打开学校给的小程序码才能看到，抓了包才找到地址）。但是它的开始考试需要使用微信的 api，在浏览器里打开没有用，然后继续抓包，找到了这个：

![burpsuite 中创建答卷 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-08-27-create-answer-api-in-burpsuite-snapshot.png)

这条请求是在这个信息登记的地方发送的：

![信息登记界面截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-exam-info-collect-snapshot.png)

`answerCollections` 里三个 `paperCollectionId` 分别代表这三个空，`openId` 应该是用来标识我微信账号的，每个账号不同，截图中的 `openId` 我已经用 `abcdef` 换掉了，`paperId` 与 `factoryCode` 应该是用来标识这个考试的。而返回中的 `body` 其实就是用来标识答卷的 `answerId`，如果用了已经考过的 `openId` 那返回就没有 `answerId`，只有 `code` 值为 `该设备已超出最大考试次数限制，无法继续考试`，就是最终显示到网页里的提示。经过测试，同一个 `openId` 一天只能创建一个答卷，不过这只是这个考试要求每个同学一天只能考一次，别的考试可能就可以随便创建了呢。但是这个 api 并没有验证 `openId` 是否真的有一个账号（这个信息登记的网址里的 `openId` 也没有验证，不写脚本的可以从 url 改 `openId`，但是我在用火狐创答卷的时候 `openId` 不同也会创不了，猜是 cookie 之类的标识了我这一个设备（因为它说超出限制的是设备不是账号），我使用的隐私浏览模式，重启浏览器就好了），所以我写 `abcdef` 也可以创建答卷，也就是说可以通过瞎写这个 `openId` 一天创建多个试卷，这下一天可以做多个试卷，也可以收集更多题目了。

点击保存并继续，就会创建答卷并进入答题页面：

![答题界面截图截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-exam-process-snapshot.png)

开着开发者工具进入，就可以发现这个可以获取到答卷题目的 api：

![可以获取到答卷题目的 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-examing-get-answer-info-api-snapshot.png)

![可以获取到答卷题目的 api 的消息头的截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-examing-get-answer-info-api-message-head-snapshot.png)

然后选择回答点击下一题，就可以看到这个提交刚刚那题的回答的 api：

![提交回答 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-submit-exam-item-answer-api-snapshot.png)

![burpsuite 中提交回答 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-08-27-submit-exam-item-answer-api-in-burpsuite-snapshot.png)

还可以发现在前端是回不到上一题的，但是我猜使用 api 就可以重选上一题。最后回答完成，提交答卷后会有这个请求用来提交答卷：

![结束答卷 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-finish-answer-api-snapshot.png)

至此，一张答卷的生命周期（应该这样说？）就算结束了。这些截图 `answerId` 不同是因为不是同一个答卷截的，27 号截图不够 2 号补的。同一个答卷的 `answerId` 是一样的，因为它标识那份答卷。

然后就是开头说没有那么多答卷时提到有个地方有答卷，其实就是考试结果的排行榜的 api 中：

![答卷结果截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-answer-result-snapshot.png)

![排名截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-ranking-list-snapshot.png)

![排名 api 截图](https://blog.turcking.eu.org/images/2024-08-31-summary-of-packet-capture-of-cslg-2024-entrance-education-exam/2024-09-02-ranking-list-api-snapshot.png)

每一项的答卷的 `answerId` 都有给出来，而且可以通过 `pageNum` 获取更多的列表，虽然前端限制了只显示 200 项，但是其实 `pageNum` 超过 10 也是能获取到的（每一页 20 项），超过了已有的答卷数量的话 `resultRankingList` 会为 `null` 不是 `[]`。

这篇写完已经 9 月 2 号了，但还是祝初音未来生日快乐。Hmi 题库搞完做了个前端，我也用完后他把它从服务器关掉了，但是还有同学需要用，所以之后准备自己再爬一遍。

---

9 月 4 日，昨天做爬虫前网上搜了一下这个网站，16 年的公司 21 年域名过的备案，到现在网上除了他们自己发的信息是什么都没有，所以一急之下把这篇文章所有有关这个网站的词除了图片里的全删掉了，望周知。

