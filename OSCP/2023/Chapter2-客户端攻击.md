# 任务8：了解攻击目标 ；HTA 程序；嵌入对象；

## 了解攻击目标

- 客户端指纹信息

	1. 浏览器供能特性丰富复杂，这是双刃剑，给攻击则提供更多攻击面，是首先的攻击目标
	1. 诱骗被攻击者访问攻击者搭建的网站，以此了解目标OS，浏览器版本信息
	1. 利用javascipt收集客户端信息(fingerpringjs2)
	1. 在线获取浏览器信息的网站[canarytokens](https://canarytokens.org/)

## HTA程序

**介绍：**

HTA 是指 HTML Application 的缩写，是一种特殊的文件格式，其后缀名为 .hta。HTA 文件是一种 Microsoft Windows 系统下的应用程序文件，使用 HTML、CSS 和 JavaScript 来创建图形用户界面（GUI）应用程序。

HTA 文件运行在 Internet Explorer 浏览器引擎（Trident）之上，但它**不受 Internet Explorer 的安全限制和网页环境的限制**。这使得 HTA 文件可以访问本地系统资源、执行本地命令、与系统进行交互，以及利用 ActiveX 控件等功能。

由于 HTA 文件可以执行本地代码，因此需要小心使用，避免从未知来源下载和运行 HTA 文件，以免遭受恶意代码攻击。

**攻击方式:**

利用msfvenom生成木马后诱骗目标点击

## 宏攻击

原理：运行vbscript,执行powshell

注意点：

1. 文件是doc不是docx。docx只能在本机攻击成功,传输后payload不会保存。
2. 宏代码可能需要拆分成多行链接在一起。

# 任务9：利用 Office 攻击；

目前office会**默认禁用宏**。但少量新文件类型(比如.pub)可能默认执行。

**office插入对象**

插入->对象->由文件创建->浏览(选择一个恶意文件,比如bat文件)->显示为图标->更改图标(把图标改成excel更具迷惑性)->题注(修改显示名的后缀更有迷惑性)

## Windows库文件攻击(new2023新增内容,实验未作)

**库文件**

简单点说就是一个快捷方式远程连接到另一个文件(服务端常用的服务是webdav服务),利用这个供能链接到一个恶意的powshell实现反弹shell。



