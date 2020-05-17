---
title: 读《白帽子讲Web安全》——客户端脚本安全
date: 2018-08-06 12:48:52
---

​	最近在学习网络安全相关的知识，于是先从业内一本系统讲Web安全的书系统学习Web安全的相关知识。在此整理书中的知识层次，不求详尽，只求自己对整个Web安全梗概有所了解，另外记录下来以便以后温习。

​	本书总共分为四篇，作者的安全世界观，客户端脚本的安全、服务端应用的安全以及互联网公司安全运营。这一篇博客记录的是客户端脚本安全的知识，包括**安全世界观、浏览器安全、XSS跨站脚本攻击、跨站点请求劫持CSRF、点击劫持和HTML5安全**。

​	ps：阅读本书时，发现作者是年西安交通大学少年班出身，在大学期间就成立了“幻影”，后成为中国安全圈内极具影响力的组织 。算来还是学长，在此对学长的书以及学长在中国网络安全界的影响，膜一波。



<!-- more --> 

# 世界观安全

- 安全三要素 CIA

  - 机密性  Confidentiality
  - 完整性 Integrity
  - 可用性 Availability

- 安全评估

  资产等级划分=》威胁分析=》风险分析=》确认解决方案

- 互联网的核心是由用户数据驱动的

  互联网安全的核心问题，是数据安全的问题

- 威胁建模 STRIDE

  伪装，篡改，抵赖，信息泄露，拒绝服务，提升权限

- 风险模型 DREAD

- 白帽子兵法

  - Secure By Default 原则（黑白名单）
  - 最小权限原则
  - 纵深防御原则（1.多层面，多方面 2.正确的地方做正确的事）
  - 数据与代码分离原则
  - 不可预测性原则

# 浏览器安全

- ## 同源策略 Same Origin Policy

  **源**  浏览器为了不让浏览器的页面行为形成干扰，提出“源”。

  影响源的**因素**有：host、子域名、端口、协议

  对于当前页面来说，重要在加载JS的域

  在浏览器中 script、 img 、iframe 、link、 标签跨域加载资源，实际发起一次GET请求，但不能读写返回的内容

  XMLHttpRequest受到同源策略影响

- ## 浏览器沙箱

  **挂马** 在网页中插入恶意代码，利用浏览器漏洞执行任意代码

  **sandbox** 资源隔离类模块，将不受信任的代码隔离在访问区之外，通过严格合法检验的API进行访问，各个模块分隔开

  **多进程浏览器** 浏览器多进程，防止单页面崩溃导致全局崩溃

- ## 恶意网址拦截

  浏览器周期性从服务器端获得一份最新的恶意网址黑名单

  **分类** 1. 挂马网站 2. 钓鱼网站

  PhishTank提供恶意网址黑名单

  EVSSL证书（兼容X509标准）浏览器特别对待

- ## 高速发展的浏览器安全

  Firefox 第一个支持CSP（Content Security Policy）的浏览器，插入一个http返回头

# 跨站脚本攻击 （XSS）

- ## 简介

  **XSS**: Cross Site Script 

  黑客通过“HTML注入”篡改了网页，插入了恶意的脚本，从而在用户浏览网页时，控制用户浏览器的一种攻击。

  分类

  - 反射型XSS ： 将用户输入的数据“反射”给浏览器 非持久性XSS
  - 储存型XSS ： 将用户输入的数据储存在服务端
  - DOM Based XSS ： 通过修改页面的DOM节点形成的XSS

- ## XSS攻击进阶

  **XSS Payload** ： JS或者其他富客户端的脚本

  常见：读取cookie对象，发起cookie劫持。（插入一张看不见的图片，将cookie发给远程服务器）

  HttpOnly可以防止cookie劫持

  **强大的XSS Payload**

  - **构造GET和POST请求** 1.构造form表单 2.XMLHttpRequest 发送POST请求
  - **XSS钓鱼** 与用户进行交互，eg：伪造出登录框，将密码发送至服务器
  - **识别用户浏览器** UserAgent对象，用JS脚本实现
  - **识别用户安装的软件** ActiveX控件，很多第三方软件也会泄露电脑软件信息
  - **CSS History Hack** 通过CSS发现用户曾经访问过的网站
  - **获取用户真实IP地址** 通过第三方软件完成，eg：JAVA的Java Applet接口

  **常见XSS攻击平台** Attack API,BeEF, XSS-Proxy

  **XSS worm** Samy Worm 通过CSS构造出XSS

  ​                    百度空间蠕虫

  **XSS构造技巧** 

  - 利用字符编码 
  - 绕过长度限制（1将恶意代码隐藏起来 eg：location.hash不会发送 2.利用注释符绕过两个文本框从而增加长度）
  - 使用base标签：设定紧接其后的相对路径的host
  - window.name 由于window对象很多时候不受同源策略的影响

  **变废为宝 碟中谍 Mission impossible ** 

  - Apache Expect Header XSS ：JS控制的浏览器环境无法控制http头，但是利用flash发起请求可以自定义大多数HTTP头
  - Anehta的回旋镖：将要利用的反射型XSS嵌入一个储存型XSS中

  **Flash XSS** 在Flash中可以嵌入ActionScript脚本 ，如果一定要用Flash，要求转码为flv静态文件，或者配置参数

  **JS 框架的XSS漏洞** 信任了用户传入的参数，用户可能上传恶意代码 

- ## XSS的防御

  - 四两拨千斤HttpOnly：禁止JS访问带有HttpOnly属性的Cookie
  - 输入检查 ： 客户端服务端同时检查 XSS Filter
  - 输出检查 ： 编码或者转义 JavaScriptEncode() HtmlEncode() ，大部分XSS漏洞可以再模板系统中解决
  - 正确地防御XSS ， XSS 实质上是一种  HTML注入，用户数据被当成HTML代码来执行，所以要在所有XSS可能出现的场景一一解决
    - 构造script标签，执行脚本——使用HtmlEncode，以及JavaScriptEncode
    - CSS，style中出现漏洞——使用encodeForCSS()函数
    - 地址栏中——URLencode
  - 处理富文本，富文本是完整的HTML代码——使用白名单
  - 防御DOM Based XSS 

# 跨站点请求伪造（CSRF）

- ## CSRF简介

  攻击者利用用户的身份，进行http请求，从而造成破坏，不需要获得cookie直接利用用户

- ## CSRF进阶

  - 浏览器的Cookie策略

    cookie分为 Session Cookie临时cookie  和Third-party cookie 本地cookie ，IE，Safari默认拦截本地cookie的发送

  - P3P头的副作用 允许iframe，script等标签就不会拦截第三方cookie的发送

  - GET/POST的漏洞，php的$_REQUEST 和POST，创建隐形的iframe让用户发起POST请求

  - Flash CSRF （已经不能发送本地cookie）

  - CSRF Worm

- ## CSRF防御

  - 验证码，强制让用户与应用进行 交互才能发起合法的网络请求
  - Referer Check 用于检查请求是否来自合法的源
  - Anti CSRF Token
    - CSRF能够成功的原因是攻击者可以猜到重要操作的所有参数
    - 增加Token在Session或者Cookie中，提交时，需要验证表单中的Token
    - Token使用原则（根据不可预测性原则）：1.足够随机生成 2.可以考虑生成多个有效的Token解决多页面共存的问题 3.Token的保密性
    - 防止Token泄露：1.Token放在表单中 2.敏感操作由GET换为POST 3.以form表单或者AJAX的形式提交

# 点击劫持（Clickjacking）

- ## 什么是点击劫持

  视觉上的欺骗手段。攻击者通过使用一个透明的、不可见的iframe覆盖在一个网页上，然后诱使用户在该网页上进行操作，此时用户将在不知情的情况下点击透明的iframe页面。通过调整iframe的位置，可以诱使用户恰好点击iframe页面的一些功能按钮。

- ## Flash点击劫持

  在Flash游戏上覆盖一个iframe最终可以在用户不知情的情况下达到目的。

- ## 图片覆盖攻击

  在可信的网站上，通过覆盖图片 **XSIO** 方式，让用户进入钓鱼网站或者利用用户完成某些操作。

- ## 拖拽劫持与数据窃取

  浏览器拖拽事件，利用隐形的iframe来诱导用户完成需要的操作。

- ### Clickjacking3.0触屏劫持

  手机OS系统浏览器中很多时候会隐藏地址栏，从而攻击者伪造出一个iframe来欺骗用户。

- ## Clickjacking防御

  通过禁止跨域的iframe来防范。

  - frame busting 防止iframe的嵌套
  - X-Frame-Options （Http头）可以选择性决定是否加载或者是否加载不同源的iframe，属性值有DENY,  SAMEORIGIN,  ALLOW-FORM origin

# HTML5安全

- ## HTML5新标签

  - 新标签的XSS ： <video>、<audio>等新标签的XSS攻击
  - iframe的sandbox： iframe新属性sandbox能够将iframe标签加载的内容视作一个独立的源，禁止执行脚本，表单禁止提交，插件禁止被加载，指向其他浏览对象的链接也会被禁止
  - Link Types：noreferrer 浏览器请求该标签不再发送Referer，需要开发者手动添加
  - Canvas的妙用：使用Canvas在线破解验证码

- ## 其他安全问题

  - Cross-Origin Resource Sharing：jsonp，iframe合法跨域，发起请求的时候必须带上一个Origin Header（判断请求是否来自一个合法的源），服务器返回一个HTTPHeader
  - postMessage——跨窗口传递：允许每一个window向其他窗口发送文本消息，从而实现跨窗口的消息传递，不受同源策略的影响。1.在接受窗口验证Domain甚至URL 2.对消息进行检查，防止XSS攻击
  - Web Storage： 受到同源策略影响。但是当储存有敏感信息时，也有可能成为被攻击的目标。