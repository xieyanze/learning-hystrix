[Configuration](https://github.com/Netflix/Hystrix/wiki/Configuration)
===
## 内容
<!-- toc -->
## 介绍
Hystrix使用 [Archaius](https://github.com/Netflix/archaius) 作为配置的默认实现属性。

下面的文档描述了使用Hystrix默认属性的策略实现，除非你通过使用插件来覆盖它。

每个属性有四个优先级别：

1. 从代码配置全局默认
如果未设置以下3中的任何一个，则这是默认值。全局默认值在下表中显示为“默认值”。

2. 动态全局默认属性
您可以使用属性更改全局默认值。全局默认属性名称在下表中显示为“默认属性”。

3. 从代码配置实例的默认属性
您可以定义特定于实例的默认值。 例：

```java
HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(int value)
```

您将以类似于以下方式将此类命令插入HystrixCommand构造函数：

```java
public HystrixCommandInstance(int id) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
        .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
               .withExecutionTimeoutInMilliseconds(500)));
    this.id = id;
}
```

有常用的初始值的方便构造函数。 这里有一个例子：

```java
public HystrixCommandInstance(int id) {
    super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"), 500);
    this.id = id;
}
```

4. 动态实例属性

您可以动态设置实例特定值，覆盖前面的三个默认值。动态实例属性名称在下表中显示为“实例属性”。
例如：

  instance property |  hystrix.command.HystrixCommandKey.execution.isolation.thread.timeoutInMilliseconds
 -----|------
 |
将属性的Hystrix命令键部分替换为您要定位的HystrixCommand的Hystrix命令Key.name（）值。

例如，如果键被命名为“SubscriberGetAccount”，则属性名称将为
>hystrix.command.SubscriberGetAccount.execution.isolation.thread.timeoutInMilliseconds

命令的属性
[查看原文表格](https://github.com/Netflix/Hystrix/wiki/Configuration#command-properties)