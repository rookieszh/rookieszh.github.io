---
title: Cookie
date: 2019-07-05 09:34:57
categories:
    - Internet
tags:
    - Cookie
    - Session
description: "Cookie是什么？他用来做什么？"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

#### Cookie 是什么
1. Cookie 是浏览器访问服务器后，服务器传给浏览器的一段数据。
2. 浏览器需要保存这段数据，不得轻易删除。
3. 此后每次浏览器访问该服务器，都必须带上这段数据。

#### 使用 Cookie
Cookie 一般有两个作用
###### 第一个作用是识别用户身份

比如用户 A 用浏览器访问了 http://a.com，那么 http://a.com 的服务器就会立刻给 A 返回一段数据「uid=1」（这就是 Cookie）。当 A 再次访问 http://a.com 的其他页面时，就会附带上「uid=1」这段数据。

同理，用户 B 用浏览器访问 http://a.com 时，http://a.com 发现 B 没有附带 uid 数据，就给 B 分配了一个新的 uid，为2，然后返回给 B 一段数据「uid=2」。B 之后访问 http://a.com 的时候，就会一直带上「uid=2」这段数据。

借此，http://a.com 的服务器就能区分 A 和 B 两个用户了。

###### 第二个作用是记录历史
假设 http://a.com 是一个购物网站，当 A 在上面将商品 A1 、A2 加入购物车时，JS 可以改写 Cookie，改为「uid=1; cart=A1,A2」，表示购物车里有 A1 和 A2 两样商品了。

这样一来，当用户关闭网页，过三天再打开网页的时候，依然可以看到 A1、A2 躺在购物车里，因为浏览器并不会无缘无故地删除这个 Cookie。

借此，就达到里记录用户操作历史的目的了。
