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

## 命令的属性
[查看原文表格](https://github.com/Netflix/Hystrix/wiki/Configuration#command-properties)

### Execution

以下属性控制 *HystrixCommand.run()* 如何执行

#### execution.isolation.strategy

此属性指示HystrixCommand.run（）执行的隔离策略，具有以下两个选择之一：

* THREAD --它在单独的线程上执行，并发请求受线程池中的线程数量的限制
* SEMAPHORE --它在调用线程上执行，并发请求受到信号量计数的限制

**Thread or Semaphore**

默认值和推荐设置是使用线程隔离（THREAD）运行命令.

在线程中执行的命令具有超出网络超时可提供的延迟的额外保护层。

一般来说，你应该使用信号量隔离（SEMAPHORE）的唯一场景是当调用是如此高的量（每秒百万次，每个实例），单独的线程的开销太高; 这通常仅适用于非网络呼叫。
>Netflix API有40多个命令在40多个线程池中运行，并且只有少数命令不在线程中运行 - 从内存中缓存获取元数据或者是面向线程隔离命令的外观（参见 [“Primary + 辅助与后退“模式](https://xieyanze.gitbooks.io/hystrix-document/content/use.html#主备fallback) 有关的更多信息）。

![isolation-options-1280.png](images/isolation-options-640.png)

有关此决策的更多信息，请参阅隔离的[工作原理](https://github.com/Netflix/Hystrix/wiki/How-it-Works#isolation)。
默认值 	 | THREAD
---	  	   |:--
可选值 	 | THREAD, SEMAPHORE
默认属性	| hystrix.command.default.execution.isolation.strategy
实例属性	| hystrix.command.HystrixCommandKey.execution.isolation.strategy
用法		 | HystrixCommandProperties.Setter().withExecutionIsolationStrategy(ExecutionIsolationStrategy.THREAD)
		  | HystrixCommandProperties.Setter().withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)

#### execution.isolation.thread.timeoutInMilliseconds

此属性设置以毫秒为单位的时间，在此之后，调用者将观察到超时并离开命令执行。 Hystrix将 *HystrixCommand* 标记为TIMEOUT，并执行回退逻辑。 注意，如果需要，可以关闭每个命令的超时（参见command.timeout.enabled）。

**注意：**即使调用者从未在结果Future上调用get()，超时将在HystrixCommand.queue()上触发。 在Hystrix 1.4.0之前，只有调用get()触发超时机制在这种情况下生效。

默认值 	 | 1000
---	  	   |:--
默认属性	| hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds
实例属性	| hystrix.command.HystrixCommandKey.execution.isolation.thread.timeoutInMilliseconds
用法		 | HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(int value)

#### execution.timeout.enabled

此属性指示 *HystrixCommand.run()* 执行是否应该有超时。

默认值 	 | true
---	  	   |:--
默认属性	| hystrix.command.default.execution.timeout.enabled
实例属性	| hystrix.command.HystrixCommandKey.execution.timeout.enabled
用法		 | HystrixCommandProperties.Setter().withExecutionTimeoutEnabled(boolean value)

#### execution.isolation.thread.interruptOnTimeout

此属性指示在发生超时时是否应中断 *HystrixCommand.run()* 执行。

默认值 	 | true
---	  	   |:--
默认属性	| hystrix.command.default.execution.isolation.thread.interruptOnTimeout
实例属性	| hystrix.command.HystrixCommandKey.execution.isolation.thread.interruptOnTimeout
用法		 | HystrixCommandProperties.Setter().withExecutionIsolationThreadInterruptOnTimeout(boolean value)

#### execution.isolation.thread.interruptOnCancel

此属性指示当发生取消时，*HystrixCommand.run()* 执行是否应该中断。

默认值 	 | false
---	  	   |:--
默认属性	| hystrix.command.default.execution.isolation.thread.interruptOnCancel
实例属性	| hystrix.command.HystrixCommandKey.execution.isolation.thread.interruptOnCancel
用法		 | HystrixCommandProperties.Setter().withExecutionIsolationThreadInterruptOnCancel(boolean value)

#### execution.isolation.semaphore.maxConcurrentRequests

此属性设置在使用执行策略为ExecutionIsolationStrategy.SEMAPHORE时，允许执行HystrixCommand.run（）方法的最大请求数。

如果此最大并发限制被命中，则后续请求将被拒绝。

你在使用信号量时使用的逻辑基本上与你选择添加到线程池中的线程数量相同，但是信号量的开销要小得多，通常执行速度要快得多（亚毫秒级） ，否则你会使用线程。

>例如，在单个实例上的5000 rps的内存中查找与度量收集已被视为与仅2的信号量一起工作。

隔离原理仍然是相同的，所以信号量应当仍然是整个容器（即Tomcat）线程池的小百分比，而不是它的全部或大部分，否则它不提供保护。

默认值 	 | 10
---	  	   |:--
默认属性	| hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests
实例属性	| hystrix.command.HystrixCommandKey.execution.isolation.semaphore.maxConcurrentRequests
用法		 | HystrixCommandProperties.Setter().withExecutionIsolationSemaphoreMaxConcurrentRequests(int value)

### Fallback

以下属性控制 *HystrixCommand.getFallback()* 如何执行。 这些属性适用于 *ExecutionIsolationStrategy.THREAD* 和 *Execution Isolation Strategy.SEMAPHORE* 。

#### fallback.isolation.semaphore.maxConcurrentRequests

此属性设置 *HystrixCommand.Fallback()* 方法允许从调用线程发出的请求的最大数量。

如果命中最大并发限制，则后续请求将被拒绝，并且由于没有回退，可以检索到异常。
>注意：这意味着如果请求线程数如果大于配置值（默认10），将直接fallback，**生产环境中注意调整此值的大小**

默认值 	 | 10
---	  	   |:--
默认属性	| hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests
实例属性	| hystrix.command.HystrixCommandKey.fallback.isolation.semaphore.maxConcurrentRequests
用法		 | HystrixCommandProperties.Setter().withFallbackIsolationSemaphoreMaxConcurrentRequests(int value)

#### fallback.enabled

Since: 1.2

此属性确定在发生失败或拒绝时是否尝试调用 *HystrixCommand.Fallback()*。

默认值 	 | true
---	  	   |:--
默认属性	| hystrix.command.default.fallback.enabled
实例属性	| hystrix.command.HystrixCommandKey.fallback.enabled
用法		 | HystrixCommandProperties.Setter().withFallbackEnabled(boolean value)

### Circuit Breaker

断路器特性控制 [HystrixCircuitBreaker](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCircuitBreaker.html) 的行为。

#### circuitBreaker.enabled

此属性确定断路器是否将用于跟踪运行状况和短路请求（如果熔断）。

默认值 	 | true
---	  	   |:--
默认属性	| hystrix.command.default.circuitBreaker.enabled
实例属性	| hystrix.command.HystrixCommandKey.circuitBreaker.enabled
用法		 | HystrixCommandProperties.Setter().withCircuitBreakerEnabled(boolean value)

#### circuitBreaker.requestVolumeThreshold

此属性在滚动窗口中设置将跳过请求链路的最小请求数。

例如，如果值为20，则如果在滚动窗口（例如10秒的窗口）中仅接收到19个请求，则即使所有19个请求失败，电路也不会熔断。

>备注：断路器熔断的最小值阀值，如果单位时间内请求小于这个值，断路器永远不会熔断。

#### circuitBreaker.sleepWindowInMilliseconds

此属性设置在调用链路熔断之后，在允许再次尝试确定调用链路是否应闭合之前拒绝请求的时间量。

>备注：可以理解为，调用链路熔断后，一定时间后(默认5秒)，将再次重试调用run()方法，检测是否需要闭合调用链路。实际多线程测试的过程中，只有一个线程会尝试闭合链路，如果仍旧符合熔断条件（比如默认值，响应时间大于1秒），则继续熔断。其他线程一直处于熔断状态，这种设计比较节省调用者的资源，没有必要所有线程都去尝试闭合。

默认值 	 | 5000
---	  	   |:--
默认属性	| hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds
实例属性	| hystrix.command.HystrixCommandKey.circuitBreaker.sleepWindowInMilliseconds
用法		 | HystrixCommandProperties.Setter().withCircuitBreakerSleepWindowInMilliseconds(int value)

#### circuitBreaker.errorThresholdPercentage

此属性设置错误百分比，在该值或以上时调用链路应熔断，并开始短路请求到fallback逻辑。

>错误数达到或超过配置值时，执行熔断操作，调用fallback.注意此百分比是值在一定时间范围内（默认10秒，见metrics.rollingStats.timeInMilliseconds）

默认值 	 | 50
---	  	   |:--
默认属性	| hystrix.command.default.circuitBreaker.errorThresholdPercentage
实例属性	| hystrix.command.HystrixCommandKey.circuitBreaker.errorThresholdPercentage
用法		 | HystrixCommandProperties.Setter().withCircuitBreakerErrorThresholdPercentage(int value)

#### circuitBreaker.forceOpen

该属性，如果为true，强制断路器进入打开（熔断）状态，其中它将拒绝所有请求。此属性优先 *circuitBreaker.forceClosed*。

默认值 	 | false
---	  	   |:--
默认属性	| hystrix.command.default.circuitBreaker.forceOpen
实例属性	| hystrix.command.HystrixCommandKey.circuitBreaker.forceOpen
用法		 | HystrixCommandProperties.Setter().withCircuitBreakerForceOpen(boolean value)

circuitBreaker.forceClosed

该属性如果为true，则迫使断路器进入闭合状态，其中它将允许所有请求通过，而不考虑误差百分比。

*circuitBreaker.force* 打开属性优先，如果设置为true，此属性不执行任何操作。

默认值 	 | false
---	  	   |:--
默认属性	| hystrix.command.default.circuitBreaker.forceClosed
实例属性	| hystrix.command.HystrixCommandKey.circuitBreaker.forceClosed
用法		 | HystrixCommandProperties.Setter().withCircuitBreakerForceClosed(boolean value)
