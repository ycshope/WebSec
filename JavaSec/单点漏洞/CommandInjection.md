# 漏洞原理

```
操作系统命令注入（也称为 shell 注入）是一种 Web 安全漏洞，允许攻击者在运行应用程序的服务器上执行任意操作系统 (OS) 命令，通常会完全破坏应用程序及其所有数据。通常，攻击者可以利用操作系统命令注入漏洞来危害托管基础设施的其他部分，利用信任关系将攻击转向组织内的其他系统。
```

# 漏洞危害

# 漏洞场景

## 涉及函数

原生的包括`Runtime.getRuntime.exec`、`ProcessBuilder.start	`

### Runtime

```java
//http://localhost:8080/rce/exec?cmd=id
@GetMapping("/runtime/exec")
public String CommandExec(String cmd) {
    Runtime run = Runtime.getRuntime();
    StringBuilder sb = new StringBuilder();

    try {
      //未过滤直接执行
        Process p = run.exec(cmd);
        BufferedInputStream in = new BufferedInputStream(p.getInputStream());
        BufferedReader inBr = new BufferedReader(new InputStreamReader(in));
        String tmpStr;

        while ((tmpStr = inBr.readLine()) != null) {
            sb.append(tmpStr);
        }

        if (p.waitFor() != 0) {
            if (p.exitValue() == 1)
                return "Command exec failed!!";
        }

        inBr.close();
        in.close();
    } catch (Exception e) {
        return e.toString();
    }
    return sb.toString();
}
```

### ProcessBuilder

```java
//  http://localhost:8080/rce/ProcessBuilder?cmd=id
@GetMapping("/ProcessBuilder")
public String processBuilder(String cmd) {

    StringBuilder sb = new StringBuilder();

    try {
        String[] arrCmd = {"/bin/sh", "-c", cmd};
        ProcessBuilder processBuilder = new ProcessBuilder(arrCmd);
      //执行命令
        Process p = processBuilder.start();
        BufferedInputStream in = new BufferedInputStream(p.getInputStream());
        BufferedReader inBr = new BufferedReader(new InputStreamReader(in));
        String tmpStr;

        while ((tmpStr = inBr.readLine()) != null) {
            sb.append(tmpStr);
        }
    } catch (Exception e) {
        return e.toString();
    }

    return sb.toString();
}
```

### GroovyShell

需要本地38080端口开启http服务来验证,本地命令行 `python3 -m http.server 38080`

```java
//http://localhost:8080/rce/groovy?content=java.lang.Runtime.getRuntime().exec("curl 127.0.0.1:38080")
@GetMapping("groovy")
public void groovyshell(String content) {
    GroovyShell groovyShell = new GroovyShell();
    groovyShell.evaluate(content);
}
```

# 漏洞利用

# 漏洞防御

# 安全bypass

# 案例学习

# 小结