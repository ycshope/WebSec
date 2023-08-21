# 任务17：信息收集；Windows提权练习；

本章主要是学习windows的命令行本地信息收集的常用命令,但我们一定要注意人**工的信息收集是不能被代替的**,典型的是用户名密码藏在某个地方

## 用户信息类

`whoami`:查看当前用户

`whoami /user`:查看当前用户以及用户ID(SID),通过SID的最后四位我们可以知道谁是管理员

`whoami /priv`:查看当前用户的权限

`whoami /groups`:查看当前用户所在的组

`net user`:查看计算机有多少用户，在win10之后`Administrator`被默认禁用，当前用户主要是通过将用户加入管理员组实现管理员权限

`net user $user`:查看当前用户信息,我们关系的是当前用户所在的用户组

`Get-LocalUser`:powershell脚本,查看计算机有多少用户

`Get-LocalGroup`:powershell脚本,查看计算机有多少用户组,**确定提权的目标**

## 日志信息类

`Get-History`:powershell脚本,windows版的`history`

`(Get-PSReadLineOption).HistorySavePath`:查看历史命令保存的文件

## 文件信息类

`Get-ChildItem -Path 'C:\' -Include *password*.txt -File -Recurse -ErrorAction SilentlyContinue`:powershell脚本,windows版的find

## 系统信息类

`systeminfo`:获取系统的硬件、版本等信息,其中版本信息可能用于提权

## 进程信息类

`tasklist`:查看系统进程

`Get-Process`:powershell版本的tasklist

## 其他

`runas /user:$user calc`：以`$user`运行某个程序

# 任务18: Windows提权练习；Linux 提权练习

## windows提权手段

1. 替换高权限的EXE
2. 替换DLL
3. 服务路径未引号闭合(重点)
4. 替换周期性执行的可执行文件

