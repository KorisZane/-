## SSTI模板注入

1、Thymeleaf
2、Velocity
3、Freemarker

[[SSTI服务端&模版注入&利用分类&语言引擎&数据渲染&项目工具&挖掘思路]]
[[JavaEE应用&SpringBoot栈&模版注入&Thymeleaf&Freemarker&Velocity]]

## SPEL表达式注入

 SpEL（Spring Expression Language）是Spring Framework中的一种表达式语言，它允许在运行时对对象图进行查询和操作。在应用程序中，如果使用不当，攻击者可以通过构造恶意输入来注入SpEL表达式，从而在表达式被解析时执行任意的命令，导致安全漏洞

安全代码

```
/**
 * 产生原因：默认的StandardEvaluationContext权限过大，用户输入的表达式被直接解析和执行
 * PoC: T(java.lang.Runtime).getRuntime().exec(%22open%20-a%20Calculator%22)
 */
public String vul(String ex) {
    ExpressionParser parser = new SpelExpressionParser();

    EvaluationContext evaluationContext = new StandardEvaluationContext();

    Expression exp = parser.parseExpression(ex);
    String result = exp.getValue(evaluationContext).toString();
    return result;
}
```

缺陷代码

```
/**
 * SimpleEvaluationContext 旨在仅支持 SpEL 语言语法的一个子集。它不包括 Java 类型引用，构造函数和 bean 引用
 */
public String spelSafe(String ex) {
    ExpressionParser parser = new SpelExpressionParser();

    EvaluationContext simpleContext = SimpleEvaluationContext.forReadOnlyDataBinding().build();

    Expression exp = parser.parseExpression(ex);
    String result = exp.getValue(simpleContext).toString();
    return result;
}
```

## Swagger

[[JavaEE应用&SpringBoot栈&Actuator&Swagger&HeapDump&提取自动化]]

发现swagger：扫描器

导入APIFox

1. 找到json格式的URL
![](sw1.png)

2. ![](sw2.png)

3. ![](sw3.png)

4. ![](sw4.png)

5. ![](sw5.png)

6. ![](sw6.png)

或者把保存为json格式文件然后文件导入

## Actuator泄露

要知道有哪些泄露，如Heapdump泄露，druid jolokia等

### Spring综合漏扫工具

![](yy.png)

### heapdump泄露工具

启动工具

```
java -jar JDumpSpiderGUI-1.0-SNAPSHOT-full.jar --gui
```

![](jd.png)

