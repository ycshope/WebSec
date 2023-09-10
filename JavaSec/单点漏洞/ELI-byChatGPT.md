# 1. 概述

## 1.1. EL表达式和SPEL表达式简介

EL（Expression Language）是一种用于访问和操作数据的表达式语言，通常用于JavaEE应用程序中的JSP页面和JSF组件。它允许开发人员在模板和页面中嵌入表达式，以动态地获取和显示数据。

SPEL（Spring Expression Language）是一种类似于EL的表达式语言，但通常与Spring框架一起使用，用于配置和运行时操作。SPEL更强大，支持更丰富的功能，包括方法调用和Bean属性访问。

然而，不正确使用EL和SPEL表达式可能导致安全漏洞，允许攻击者注入恶意代码并执行远程操作。

## 1.2. 漏洞定义和原理

EL和SPEL注入漏洞是指攻击者能够在应用程序中的EL或SPEL表达式中注入恶意代码。这通常发生在以下情况下：

- **用户输入**：当应用程序允许用户提供表达式或变量作为输入时，攻击者可以注入恶意代码。
- **配置文件**：不正确配置的配置文件可能包含可执行的EL或SPEL表达式，攻击者可以修改这些文件以执行恶意操作。
- **动态查询**：在动态查询中使用EL或SPEL表达式时，如果没有正确验证和过滤用户提供的查询参数，可能会导致注入攻击。

这些漏洞的原理在于攻击者能够欺骗应用程序执行他们提供的恶意表达式，而不经过适当的验证和授权。

## 1.3. 危害和风险

EL和SPEL表达式注入漏洞的危害包括：

- **远程代码执行**：攻击者可以注入恶意EL或SPEL表达式，远程执行任意代码，危及应用程序的安全性。
- **敏感信息泄露**：攻击者可以访问应用程序的敏感数据，如数据库凭证或用户信息。
- **拒绝服务攻击**：通过构建恶意表达式，攻击者可以导致应用程序崩溃或不可用。

由于这些潜在威胁，EL和SPEL表达式注入漏洞需要认真对待，进行漏洞挖掘、修复和防范是至关重要的。

# 2. 漏洞挖掘和利用

## 2.1. 恶意输入

攻击者可以通过在用户输入或查询参数中注入恶意的EL或SPEL表达式来触发注入漏洞。以下是一个示例，其中用户可以输入一个表达式，并应用程序会将其评估：

```java
String userInput = getUserInput(); // 从用户获取输入
String elExpression = "${" + userInput + "}";
String result = evaluateEL(elExpression); // 恶意输入触发漏洞
```

在这个示例中，如果不适当验证和过滤`userInput`，攻击者可以输入恶意表达式，如`${T(java.lang.Runtime).getRuntime().exec('calc.exe')}`，从而执行远程代码。

### Sink点示例：

在上述示例中，`evaluateEL`方法是一个潜在的sink点，它接受用户输入和EL表达式并执行EL表达式。

```java
public String evaluateEL(String expression) {
    // 使用EL解析器执行表达式
    ELProcessor elp = new ELProcessor();
    Object result = elp.eval(expression);
    return result.toString();
}
```

## 2.2. 配置文件漏洞

EL或SPEL表达式也可以包含在应用程序的配置文件中，攻击者可以修改这些文件以执行恶意操作。以下是一个示例，其中配置文件包含一个恶意的EL表达式：

```xml
<property name="query" value="${T(java.lang.Runtime).getRuntime().exec('calc.exe')}"/>
```

攻击者可以通过修改配置文件来执行不受信任的代码。

### Sink点示例：

在这种情况下，sink点是配置文件中包含EL或SPEL表达式的位置。攻击者可以修改这些表达式来执行恶意代码。

## 2.3. 动态查询

在动态查询中使用EL或SPEL表达式时，要特别小心不要接受不受信任的输入。例如，以下是一个使用EL的动态查询示例：

```java
@Query("SELECT e FROM Employee e WHERE e.name = :userInput")
List<Employee> findEmployeesByName(@Param("userInput") String userInput);
```

如果`userInput`未经适当验证和过滤，攻击者可以构造恶意的输入来执行危险查询。

### Sink点示例：

在这个示例中，sink点是使用EL或SPEL表达式构建的动态查询，其中包含`userInput`。攻击者可以注入危险的`userInput`来执行攻击。

# 3. 防御

## 3.1. 输入验证和过滤

在接受用户输入时，进行充分的验证和过滤，确保用户输入不包含恶意的EL或SPEL

表达式。

### 防御示例：

```java
String userInput = getUserInput(); // 从用户获取输入
if (!isValidInput(userInput)) {
    // 非法输入，进行适当的处理，例如拒绝或清理输入
} else {
    // 正常处理输入
}
```

## 3.2. 安全配置文件

保护应用程序的配置文件，确保其中不包含可执行的EL或SPEL表达式。限制配置文件的访问权限，定期审查和检查配置文件以查找潜在的漏洞。

### 防御示例：

- 限制配置文件的访问权限，只允许授权用户或进程访问。
- 定期审查配置文件以查找包含EL或SPEL表达式的不安全部分。

## 3.3. 安全上下文

在评估EL或SPEL表达式时，使用安全的上下文来限制可执行的函数和类。禁用危险的函数，只允许受信任的类和方法。

### 防御示例：

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();

// 注册允许的函数和类
context.registerFunction("safeFunction", SafeFunction.class.getMethod("safeMethod"));
context.registerFunction("safeClass", SafeClass.class);

String userInput = getUserInput(); // 从用户获取输入
String expression = "#{" + userInput + "}";
Object result = parser.parseExpression(expression).getValue(context);
```

# 4. 绕过

## 4.1. 改变表达式结构

攻击者可能尝试改变EL或SPEL表达式的结构，以绕过输入验证和过滤。例如，尝试使用不同的表达式运算符或函数。

### 绕过示例：

```java
String userInput = "1+1"; // 输入中包含不同的表达式结构
```

## 4.2. 使用编码

攻击者可以尝试使用编码来混淆EL或SPEL表达式，使其不易检测。例如，使用十六进制或Base64编码。

### 绕过示例：

```java
String userInput = "#{'T(java.lang.Runtime).getRuntime()'}"; // 输入经过编码
```

## 4.3. 长度限制绕过

攻击者可能尝试构造超长的输入，以绕过长度限制和过滤。

### 绕过示例：

```java
String userInput = generateLongInput(); // 构造超长输入
```

总之，EL和SPEL表达式注入漏洞可能导致严重的安全问题，因此必须小心处理用户输入，保护配置文件，并使用安全的上下文来限制可执行的函数和类。此外，定期审查和检查应用程序以确保漏洞没有被引入。

这篇文章旨在提供关于Java EL表达式和SPEL表达式注入漏洞的基本了解，并介绍了漏洞的挖掘、危害、防御和绕过方法。在实际应用程序中，安全性是至关重要的，因此开发人员和安全专家应该密切关注并采取必要的措施来保护应用程序免受这些漏洞的威胁。