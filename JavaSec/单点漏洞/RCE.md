# 1. 概述

## 	1.1. 原理

Java远程代码执行漏洞是一种安全漏洞，允许攻击者在远程服务器上执行恶意Java代码。Java通常被用于构建安全的跨平台应用程序，但当不正确地配置和使用时，它可能会导致远程代码执行漏洞。远程代码执行漏洞可能发生在Java应用程序、Web应用程序、服务器或其他Java环境中。

## 	1.2. 场景

Java远程代码执行漏洞是指攻击者能够在远程服务器上执行未经授权的Java代码。这通常发生在以下情况下：

- **不安全的反序列化**：Java应用程序可能会从网络或其他不受信任的来源接收序列化的数据，攻击者可以操纵这些数据来触发反序列化漏洞，并在目标系统上执行恶意代码。
- **动态类加载**：Java应用程序允许动态加载类和代码，如果未正确验证和限制加载的代码，攻击者可以注入恶意类并执行它们。
- **自定义类加载器**：自定义类加载器可以用于绕过Java的安全机制，攻击者可能会使用它们来加载恶意类和执行远程代码。

漏洞的原理在于攻击者能够欺骗Java应用程序执行他们提供的代码，而不经过适当的验证和授权。

## 	1.3. 危害

Java远程代码执行漏洞的危害包括：

- **远程控制**: 攻击者可以完全控制目标服务器，执行任意操作，包括潜在的破坏性操作。
- **数据泄露**: 攻击者可以访问和窃取敏感数据，如数据库凭证、用户信息等。
- **系统漏洞利用**: 攻击者可以利用目标服务器上的系统漏洞，进一步扩大攻击面。
- **后门创建**: 攻击者可能在系统中创建后门，以便随时重新进入系统。

由于这些潜在威胁，Java远程代码执行漏洞是严重的安全问题，需要及时发现并修复，以防止潜在的攻击和数据泄露

# 2. 漏洞挖掘和利用

在进行Java远程代码执行漏洞的代码审计时，重要的是要了解潜在的sink点，即可能执行外部代码的地方。以下是一些常见的sink点示例以及如何使用IDEA等工具来寻找它们：

## 2.1.反序列化操作

查找在代码中使用反序列化的地方，如`ObjectInputStream`的使用。这里的sink点是在反序列化后执行的代码，例如对象的`readObject()`方法。更具体的在**反序列化**的章节补充

```java
javaCopy codeObjectInputStream ois = new ObjectInputStream(inputStream);
Object obj = ois.readObject(); // sink点
```

## 2.2.外部命令执行

查找在代码中执行外部命令的地方，如`Runtime.exec()`或`ProcessBuilder`。

### Runtime

```java
    /**
     * payload:
     * /runtime/exec?cmd=powershell%20systeminfo
     */
    @GetMapping("/runtime/exec")
    public String CommandExec(String cmd) {
        Runtime run = Runtime.getRuntime(); // sink点
        StringBuilder sb = new StringBuilder();

        try {
            Process p = run.exec(cmd);
            BufferedInputStream in = new BufferedInputStream(p.getInputStream());
            BufferedReader inBr = new BufferedReader(new InputStreamReader(in));
            String tmpStr;

            while ((tmpStr = inBr.readLine()) != null) {
                sb.append(tmpStr);
            }

            if (p.waitFor() != 0) {
                if (p.exitValue() == 1) return "Command exec failed!!";
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
    /**
     * payload:
     * /rce/ProcessBuilder?cmd=powershell%20systeminfo
     */
    @GetMapping("/ProcessBuilder")
    public String processBuilder(String cmd) {

        StringBuilder sb = new StringBuilder();

        try {
            String[] arrCmd = {"powershell", cmd};
            ProcessBuilder processBuilder = new ProcessBuilder(arrCmd); // sink点
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



## 2.3.动态类加载

查找在代码中使用动态类加载的地方，如`Class.forName()`。下列是利用`ProcessImpl`来实现exp

### Class.forName

下面用ProcessImpl和UNIXProcess来演示下该漏洞的利用过程

**ProcessImpl**

`java.lang.ProcessImpl` 是 Java 的内部类，用于处理创建和控制本机进程（Native Process）。这个类的主要作用是为 `java.lang.ProcessBuilder` 类提供底层的实现，以便在不同的操作系统上创建和管理进程。

直接调用 `java.lang.ProcessImpl` 不是一个标准的 Java API，因为它是 Java 核心库的一部分，通常不应该由应用程序代码直接访问。这是因为它包含了操作系统相关的本机代码，而且不同的操作系统实现可能会有不同的细节和行为。

正常情况下，应该使用更高级别的 API，比如 `ProcessBuilder`，来创建和执行进程。`ProcessBuilder` 抽象了进程的创建和控制，提供了更安全和可移植的方式来处理进程。

```java
    /**
     * payload:
     * /rce/ProcessImpl?obj=java.lang.ProcessImpl&cmd=powershell%20systeminfo
     */
    @GetMapping("/ProcessImpl")
    public String processImpl(String obj, String cmd) {
        String objStr = obj;
        String cmdStr = cmd;
        String retStr = "";

        if (objStr == null || cmdStr == null) return retStr = "obj and cmd cannot be Null!";

        // 获取类
        Class clazz = null;
        try {
            clazz = Class.forName(objStr); //动态类加载,sink点
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }

        // 实例化
        Object object = null;
        Constructor<?> constructor = clazz.getDeclaredConstructors()[0];
        constructor.setAccessible(true);
        try {
            object = constructor.newInstance(cmdStr.split("\\s+"), null, "./", new long[]{-1L, -1L, -1L}, false);
        } catch (InstantiationException e) {
            throw new RuntimeException(e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        } catch (InvocationTargetException e) {
            throw new RuntimeException(e);
        }

        //通过反射执行函数,获取结果
        Method inMethod = null;
        try {
            inMethod = object.getClass().getDeclaredMethod("getInputStream");
        } catch (NoSuchMethodException e) {
            throw new RuntimeException(e);
        }
        inMethod.setAccessible(true);

        // 返回结果
        InputStream in = null;
        try {
            int len = 0;
            byte[] Buffer = new byte[1024];
            String tmpBuffer = "";

            in = (InputStream) inMethod.invoke(object);
            while ((len = in.read(Buffer)) != -1) {
                tmpBuffer = new String(Buffer, 0, len);
                retStr += tmpBuffer;
            }
            in.close();
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        } catch (InvocationTargetException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        return retStr;
    }
```

**UNIXProcess(待补充)**

## 2.4.自定义类加载器(待尝试)

查找自定义类加载器的使用，因为它可以绕过标准的类加载机制。

```java
ClassLoader customClassLoader = new CustomClassLoader(); // sink点
```



# 3. 防御(待尝试)

## 3.1.限制反序列化

防御远程代码执行漏洞的一种方法是限制反序列化操作，只允许反序列化已知和受信任的类。下面是一个示例：

```java
javaCopy codeimport java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.ObjectInputStream;

public class SafeDeserialization {
    public static void main(String[] args) {
        try {
            byte[] serializedData = /* 从网络或其他来源获取的序列化数据 */;
            
            ByteArrayInputStream bis = new ByteArrayInputStream(serializedData);
            ObjectInputStream ois = new ObjectInputStream(bis);
            
            // 检查反序列化的类是否在白名单中
            String className = ois.readUTF();
            if (isAllowedClass(className)) {
                Object obj = ois.readObject();
                // 处理反序列化后的对象
            } else {
                // 拒绝反序列化，抛出异常或执行其他适当的操作
            }
        } catch (Exception e) {
            // 处理异常
        }
    }
    
    private static boolean isAllowedClass(String className) {
        // 检查类名是否在白名单中，如果是，返回true，否则返回false
        // 可以使用硬编码的白名单或配置文件中的白名单
        return className.equals("com.example.AllowedClass");
    }
}
```

## 3.2.防止动态类加载

为防止攻击者利用动态类加载来执行恶意代码，应限制加载类的路径，只允许加载受信任的类。下面是一个示例：

```java
codeClassLoader classLoader = Thread.currentThread().getContextClassLoader();
String className = "com.example.AllowedClass"; // 需要加载的类名

try {
    Class<?> clazz = classLoader.loadClass(className);
    // 处理加载后的类
} catch (ClassNotFoundException e) {
    // 类不存在或不在允许的类路径中，执行适当的操作，如拒绝加载
}
```

## 3.3.输入验证和过滤

在处理用户输入时，进行充分的验证和过滤，确保用户输入不会直接传递给危险的操作。

```java
codeString userInput = request.getParameter("userInput");
if (!isValid(userInput)) {
    // 非法输入，进行适当的处理
} else {
    // 正常处理输入
}
```

# 4. 绕过(待尝试)

## 4.1.改变类名

攻击者可能尝试更改类名以绕过白名单机制。

示例payload：

```java
String maliciousClassName = "com.example.EvilClass"; // 类名已更改
```

## 4.2.使用自定义类加载器

攻击者可以尝试使用自定义类加载器来绕过静态类加载限制。

示例payload：

```java
codeClassLoader customClassLoader = new CustomClassLoader(); // 自定义类加载器
String className = "com.example.EvilClass"; // 恶意类名

try {
    Class<?> clazz = customClassLoader.loadClass(className);
    // 尝试加载恶意类
} catch (ClassNotFoundException e) {
    // 类不存在或不在允许的类路径中，执行适当的操作，如拒绝加载
}
```

## 4.3.模糊输入

攻击者可能尝试使用特殊字符和编码来绕过输入验证和过滤。

示例payload：

```java
String userInput = "EvilPayload"; // 带有特殊字符的输入
```

这些示例涵盖了一些常见的防御和绕过方法。要提高安全性，应综合使用多个防御层次，不断更新白名单、审计代码、监视日志，以及保持对最新安全威胁的了解。这将帮助您更好地保护系统免受远程代码执行漏洞的威胁。

-------------------------------

GroovyShell

需要本地38080端口开启http服务来验证,本地命令行 `python3 -m http.server 38080`

```java
//http://localhost:8080/rce/groovy?content=java.lang.Runtime.getRuntime().exec("curl 127.0.0.1:38080")
@GetMapping("groovy")
public void groovyshell(String content) {
    GroovyShell groovyShell = new GroovyShell();
    groovyShell.evaluate(content);
}
```

