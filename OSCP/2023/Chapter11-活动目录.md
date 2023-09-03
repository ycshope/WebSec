# 任务24：AD 理论 ；

## 需求

将WindowsServers2022作为域控并配置域环境

## 安装步骤

**1.设置域控为静态ip**

- 将ip配置为静态ip，defaultGw 为默认网关，DNS解析器为`127.0.0.1`(这里是用域控同时作为dns)

**2.修改计算机名**

- `服务器管理`->`本地服务器`->`计算机名`->`更改`：pc名为DC1

**3.安装域服务**

- `服务器管理`->`仪表板`->`添加角色和供能`,一直下一步
- `选择服务角色`->`AD域服务`,一直下一步

**4.设置本机为域服务器**

- 点击`管理`左边flag的感叹号，点击`将此服务升级为域控制器`
- 选择`部署配置`->`添加新林`->`根域名`填入`oscp.com`,剩下一直下一步

## 安装问题

1.code: UNSUPPORTED_PROCESSOR

 解决方案：镜像下载为英文版本。



# 任务25：AD 枚举（一）；

## 概念

要加入域首先要通过认证,通过认证后调用域内其他服务器的资源依旧有权限控制,不过查询所有域资源是默认允许的。

## 需求

将Windows10和Windows11加入域

## Win10加入域

### 域控为成员创建域账号

1.打开域管理界面

命令行输入`dsa.msc`

2.为指定的域加入用户

`Active Directory用户和计算机`->`oscp.com`->`Users`->`右键新建`->`用户`

其中姓和名和可以随填,用户名为`$ADuser1`(用于登录的域账户),接下来设置密码就可以



### PC1设备加入域,成为域成员

**1.设置域控为静态ip**

- 将ip配置为静态ip，defaultGw 为默认网关，DNS解析器为域控

**2.计算器加入域**

用下面链接的方法1即可,步骤3的域名填`oscp.com`,步骤4的域用户认证为**域管理员**

[参考链接](https://www.top-password.com/blog/add-windows-10-to-active-directory-domain/)

**3.登录域账户**

重启，用域账户登录,登录用户为`oscp\$ADuser1`

### 安装问题

1.code: UNSUPPORTED_PROCESSOR

 解决方案：CPU设置为单核,其他[参考方案链接](https://www.youtube.com/watch?v=ewAKsSaMPI8)

2.[激活问题](https://m.haozhuangji.com/xtjc/153923946.html)

## Win11加入域

### win11加入域

同win10

### 安装问题

1. [设备不满足要求](https://www.youtube.com/watch?v=EMuw_IN-UOU)

2. [激活问题](https://m.haozhuangji.com/xtjc/112927314.html)

## 将域用户加入到本地管理员组

**域成员关闭防火墙**

`windows security` -> `firewall & network protection`，将`domain network`,`private network`,`public network`的防火墙给关闭,这里需要输入域控的账号和密码

**通过域控给win11的域用户加入到本地管理员组**

1. 打开域管理界面

命令行输入`dsa.msc`

2. 管理指定的设备

`Active Directory用户和计算机`->`oscp.com`->`computers`->`win11`->右键`管理`

3. 将域成员加入到本地管理组

`系统工具`->`本地用户和组`->`组`->`administrators`->右键`属性`->`添加`->输入域成员名,不需要带`OSCP\`

*补充：域管理员默认在是所有域成员的本地管理组*

## 将域用户加入到域管理员组

需求：在域控中将win10加入到安全组admin中,再将admin组加入到域控中

**创建安全组admin**

1. 命令行输入`dsa.msc`
2. `Active Directory用户和计算机`->`oscp.com`->`Users`->`右键新建`->`组`->输入`admin`并选择`安全组`

**将安全组admin加入到域控**

1. 双击`Domain Users`->`成员`->`添加`->输入`admin`

**将win10加入到安全组admin中**

1. 双击`admin`->`成员`->`添加`->输入win10的域成员名

## 域的攻击思路

整体思路是攻击权限的层级

1. 通过找到高权限账户的主机,利用`Pass the Hash`(从进程`LSASS`的内存中提取缓存的密码)获得高权限用户密码,并最终找到域控的密码实现整个域的渗透

2. 也可能会通过`pass the ticket`(伪造凭证)