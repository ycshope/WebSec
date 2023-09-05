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

# 任务26：AD 枚举（二）；

## 半自动信息收集工具

### powerview

#### 概述

该工具是个kali默认集成的powershell脚本,全名为`powerview.ps1`,该脚本能提供常见的域信息收集功能。

#### 使用教程

**导入模块**

1.关闭本地防病毒软件检测（很重要,不然没法导入模块）

`windows security`->`virus&threat portection`->turn off`virus&threat portection settings`

2.导入powerview

​	2.1.关闭保护(很重要)： `powershell -ep bypass`

​	2.2.导入模块：`Import-Module .\powerview.ps1`

**常用信息收集命令**

查询域当前所有的用户：`Get-NetUser`

查询域当前所有的组：`Get-NetGroup`

获取域内的管理员组：`Get-NetGroup "domain admins" | select member`

查询域内host名：`Get-NetComputer | select dnshostname`

查询域当中提供的服务：`Get-NetUser -SPN | select serviceprincipalname`

[其他参考](https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview)

### psloggedon64

该工具是微软自己提供的,常用的方法是查看设备上是否有高权限账户

## 全自动化信息收集工具&分析工具

这款工具在域渗透中十分重要，`sharphound`是信息收集工具，信息收集完后会生成一个json文件；`bloodhound`是一个分析工具，利用`sharphound`收集到的信息为攻击者提供攻击路径

### sharphound

工具位置:`/usr/share/metasploit-framework/data/post/powershell/SharpHound.ps1`

**使用步骤:**

**导入模块**

1.关闭本地防病毒软件检测（很重要,不然没法导入模块）

`windows security`->`virus&threat portection`->turn off`virus&threat portection settings`

2.导入powerview

​	2.1.关闭保护(很重要)： `powershell -ep bypass`

​	2.2.导入模块：`Import-Module .\powerview.ps1`

3.信息收集

​	启动`sharphound`进行信息收集：`Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\Public` 

4.回传到kali

​	4.1.kali本地启动smb服务：`impacket-smbserver s /home/kali/s -smb2support` 

​		该命令表示开启将`/home/kali/s`映射出去,所以需要提前创建好该目录才能进行文件传输

​	4.2.windows通过smb将信息收集的压缩包回传给kali:

​		用`ctrl+R`调度出命令,输入`\\$kali_ip`,将文件放该目录下即可

### bloodhound

**安装**

该软件默认没集成,需要进行安装

**导入数据**

1.启动数据库,并修改密码:

​	1.1.输入`sudo neo4j start`后会启动一个服务，点击该url并进入服务，默认账号密码均为`neo4j`

​	1.2.只需要输入username和password即可

2.打开bloodhound,并上传数据

​	2.1.输入`bloodhound`启动程序,并将`1.1`的密码输入

​	2.2.将`sharphound`信息收集结果的压缩包解压,里面有一堆json文件

​	2.3.点击`upload data`（右边第四个选择）上传所有json

**使用**

常用的分析工具都在左边的`analysis`中

常用的分析能力:

1.找到所有成员：`find all domain admins`

2.最短攻击路径：该功能需要标记一个节点作为已获得的权限才能使用

​	

## 工具模式补充(待验证)

一些情况下域控可能会在自己本地修改域成员的密码,这个过程中可能密码的哈希会保存在域控的共享文件中,通过破解这个hash(目前windows有已经泄露的域控密钥)就能进行密码窃取

步骤：

1.访问管理员的日志文件

用`ctrl+R`调度出命令,输入`\\$domain_admin_ip`。访问`SYSVOL`下的文件

2.通过信息搜索找到修改密码的文件

一般文本会包含`cpassword`

3.利用工具进行密码破解

`gpp-decrypt "$password_hash"`。该工具默认集成了泄露的密钥

