### 内容
<!-- toc -->

## "Hello World!"
下面是 [***HystrixCommand***](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) 的一个基本的“Hello World”实现：
```java
public class CommandHelloWorld extends HystrixCommand<String> {

    private final String name;

    public CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        // a real example would do work like a network call here
        return "Hello " + name + "!";
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloWorld.java)


使用 [***HystrixObservableCommand***](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixObservableCommand.html) 的等效Hello World解决方案将涉及覆盖构造方法，如下所示：

```java
public class CommandHelloWorld extends HystrixObservableCommand<String> {

    private final String name;

    public CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected Observable<String> construct() {
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> observer) {
                try {
                    if (!observer.isUnsubscribed()) {
                        // a real example would do work like a network call here
                        observer.onNext("Hello");
                        observer.onNext(name + "!");
                        observer.onCompleted();
                    }
                } catch (Exception e) {
                    observer.onError(e);
                }
            }
         } );
    }
}
```

## 同步执行

你可以使用 [execute()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#execute()) 方法同步执行一个***HystrixCommand***，如以下示例所示：

```java
String s = new CommandHelloWorld("World").execute();
```

这种形式的执行通过以下测试：

```java
@Test
public void testSynchronous() {
    assertEquals("Hello World!", new CommandHelloWorld("World").execute());
    assertEquals("Hello Bob!", new CommandHelloWorld("Bob").execute());
}
```

> 对于 ***HystrixObservableCommand*** 没有简单的等价执行，但是如果你知道这样的命令产生的 ***Observable*** 总是只产生一个值，你可以通过应用 ***.toBlocking().toFuture().get()*** 来模拟execute的行为 通知到 ***Observable***。

## 异步执行

您可以使用 [queue()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#queue()) 方法异步执行HystrixCommand，如以下示例所示：

```java
Future<String> fs = new CommandHelloWorld("World").queue();
String s = fs.get()
```

下面的单元测试演示异步执行操作：
```java
@Test
public void testAsynchronous1() throws Exception {
    assertEquals("Hello World!", new CommandHelloWorld("World").queue().get());
    assertEquals("Hello Bob!", new CommandHelloWorld("Bob").queue().get());
}

@Test
public void testAsynchronous2() throws Exception {

    Future<String> fWorld = new CommandHelloWorld("World").queue();
    Future<String> fBob = new CommandHelloWorld("Bob").queue();

    assertEquals("Hello World!", fWorld.get());
    assertEquals("Hello Bob!", fBob.get());
}
```

以下代码是等效的：

```java
String s1 = new CommandHelloWorld("World").execute();
String s2 = new CommandHelloWorld("World").queue().get();
```

> 对于 ***HystrixObservableCommand*** 没有简单的等价执行，但是如果你知道这样的命令产生的 ***Observable*** 总是只产生一个值，你可以通过应用 ***.toBlocking().toFuture().get()*** 来模拟execute的行为 通知到 ***Observable***。

## 响应式执行

您还可以通过使用以下方法之一来观察HystrixCommand作为Observable的结果：

* [observe()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#observe()) - 返回一个“hot” ***Observable***立即执行命令，在订阅之前，因为 ***Observable*** 通过 ***ReplaySubject*** 过滤，你不会有丢失任何项目的危险。
* [toObservable()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#toObservable()) - 返回一个“cold”Observable，它不会执行命令并开始发出其结果，直到您订阅了 ***Observable***

> toObservable() 创建的对象，不能执行，只能订阅

```java
Observable<String> ho = new CommandHelloWorld("World").observe();
// or Observable<String> co = new CommandHelloWorld("World").toObservable();
```

然后，通过订阅Observable来检索命令的值：

```java
ho.subscribe(new Action1<String>() {

    @Override
    public void call(String s) {
         // value emitted here
    }

});
```

以下单元测试演示响应式执行：

```java
@Test
public void testObservable() throws Exception {

    Observable<String> fWorld = new CommandHelloWorld("World").observe();
    Observable<String> fBob = new CommandHelloWorld("Bob").observe();

    // blocking
    assertEquals("Hello World!", fWorld.toBlockingObservable().single());
    assertEquals("Hello Bob!", fBob.toBlockingObservable().single());

    // non-blocking 
    // - this is a verbose anonymous inner-class approach and doesn't do assertions
    fWorld.subscribe(new Observer<String>() {

        @Override
        public void onCompleted() {
            // nothing needed here
        }

        @Override
        public void onError(Throwable e) {
            e.printStackTrace();
        }

        @Override
        public void onNext(String v) {
            System.out.println("onNext: " + v);
        }

    });

    // non-blocking
    // - also verbose anonymous inner-class
    // - ignore errors and onCompleted signal
    fBob.subscribe(new Action1<String>() {

        @Override
        public void call(String v) {
            System.out.println("onNext: " + v);
        }

    });
}
```

使用Java 8 lambdas/closures重新上面的代码可以更紧凑; 它看起来像这样：
```java
fWorld.subscribe((v) -> {
        System.out.println("onNext: " + v);
    })

    // - or while also including error handling

    fWorld.subscribe((v) -> {
        System.out.println("onNext: " + v);
    }, (exception) -> {
        exception.printStackTrace();
    })
```

有关Observable的更多信息，请参见：[http://reactivex.io/documentation/observable.html](http://reactivex.io/documentation/observable.html)
## 响应式命令
您还可以创建一个 ***HystrixObservableCommand***，它是 ***HystrixCommand*** 的一个特殊版本，用于包装 ***Observables*** ,而不是使用上述方法将 ***HystrixCommand*** 转换为 ***Observable***。 ***HystrixObservableCommand*** 能够包装发出多个 ***Observable***，而普通的 ***HystrixCommands***，即使转换为 ***Observables***，也不会发出多个。

在这种情况下，不是使用命令逻辑覆盖run方法（就像使用普通的 ***HystrixCommand*** 时候一样），您将覆盖 ***construct*** 方法，以便它返回要包装的 ***Observable*** 。

## fallback

您可以通过添加一个fallback方法来支持Hystrix命令的正常降级，Hystrix将在主命令失败的时候调用该方法以获取一个或多个默认值。 这样做您可以为大多数可能会失败的Hystrix命令实施fallback，但有几个例外：
1. 执行写操作的命令

	* 如果你的Hystrix命令被设计为执行写操作而不是返回一个值（这样的命令通常在执行 ***HystrixCommand*** 的情况下返回一个 ***void*** 或者在  ***HystrixObservableCommand*** 的情况下返回一个空的 ***Observable*** ），没有切入点可以实施fallback。 如果写入失败，您可能希望失败通知到调用者。

2. 批处理系统或离线计算机

	* 如果你的Hystrix命令填满了一个缓存，或者生成一个报告，或者进行任何类型的离线计算，通常更合适的做法是将错误传播给调用者，调用者稍后可以重试该命令，而不是发送调用者静默降级反应。

无论您的命令是否具有fallback，所有 Hystrix状态和断路器状态/指标都会更新，以表明命令失败。

在普通的 ***HystrixCommand*** 中，通过 [getFallback()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#getFallback()) 实现来实现fallback。 Hystrix将对所有类型的故障（如run（）失败，超时，线程池或信号量拒绝和断路器短路）执行fallback。 以下示例包含此类回退：

```java
public class CommandHelloFailure extends HystrixCommand<String> {

    private final String name;

    public CommandHelloFailure(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        throw new RuntimeException("this command always fails");
    }

    @Override
    protected String getFallback() {
        return "Hello Failure " + name + "!";
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloFailure.java)

上面命令的 ***run()*** 方法在每次执行时都会失败。但是，调用者将总是接收由命令 ***getFallback()*** 方法返回的值，而不是接收异常：

```java
@Test
public void testSynchronous() {
    assertEquals("Hello Failure World!", new CommandHelloFailure("World").execute());
    assertEquals("Hello Failure Bob!", new CommandHelloFailure("Bob").execute());
}
```

> 对于 ***HystrixObservableCommand***，您可以覆盖 ***resumeWithFallback*** 方法，以便它返回第二个 ***Observable***，如果它失败，它将接管主 ***Observable***。 注意，因为***Observable***可能在已经发出一个或多个项后失败，您的回退不应该假设它发出观察者将看到的唯一值。

> 在内部，Hystrix使用RxJava [onErrorResumeNext](http://reactivex.io/documentation/operators/catch.html)运算符在发生错误的情况下在主要和后备Observable之间无缝转换。

## 错误传播
从 ***run()*** 方法抛出的除了 [HystrixBadRequestException](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/exception/HystrixBadRequestException.html) 以外的所有异常，都用于计数失败，触发器获取 ***Fallback()*** 和断路器处理逻辑。

你可以用 ***HystrixBadRequestException*** 包装你想抛出的异常，并通过 ***getCause()*** 检索它。 ***HystrixBadRequestException***  旨在用于报告非法参数或非系统故障的用例，这些故障不应计入故障指标，不应触发fallback逻辑。

>在使用 ***HystrixObservableCommand*** 的情况下，不可恢复的错误通过 ***onError*** 通知从最终的Observable返回，并且通过落回到Hystrix通过您实现的 ***resumeWithFallback*** 方法获得的第二个 ***Observable*** 来完成fallback。

## 命令名称

默认情况下，命令名称派生自类名：

```java
getClass().getSimpleName();
```
要显式定义名称，通过 ***HystrixCommand*** 或 ***HystrixObservableCommand*** 构造函数传递它：

```java
public CommandHelloWorld(String name) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
            .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld")));
    this.name = name;
}
```

要保存每个命令分配的Setter分配，您还可以缓存Setter，如下所示：

```java
private static final Setter cachedSetter =
    Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
        .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld"));

public CommandHelloWorld(String name) {
    super(cachedSetter);
    this.name = name;
}
```

[HystrixCommandKey](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html) 是一个接口，可以实现为枚举或常规类，但它也有帮助 [Factory](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.Factory.html) 类来构造和实例化实例，如：
```java
HystrixCommandKey.Factory.asKey("HelloWorld")
```
## 命令分组
Hystrix使用 ***command group Key*** 将诸如报告，警报，仪表板或团队/库所有权等命令分组在一起。

默认情况下Hystrix使用它来定义命令线程池，除非定义了一个单独的。

[HystrixCommandGroupKey](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.html)是一个接口，可以实现为枚举或常规类，但它也有帮助 [Factory](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.Factory.html) 类来构造和实例化实例，如：

```java
HystrixCommandGroupKey.Factory.asKey("ExampleGroup")
```
## 命令线程池

thread-poll key 标识了用于监视，指标发布，缓存和其他此类用途的 [HystrixThreadPool](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPool.html)。 ***HystrixCommand*** 与由 [HystrixThreadPoolKey](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.html) 注入的单个 ***HystrixThreadPool*** 相关联，或者它默认为使用创建的 ***HystrixCommandGroupKey*** 创建的 ***HystrixThreadPool***。

要显式定义名称，通过 ***HystrixCommand*** 或 ***HystrixObservableCommand*** 构造函数传递它：

```java
public CommandHelloWorld(String name) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
            .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld"))
            .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("HelloWorldPool")));
    this.name = name;
}
```

[HystrixThreadPoolKey](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.html)是一个接口，可以实现为枚举或常规类，但它也有帮助 [Factory](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.Factory.html) 类来构造和实例化实例，如：

```java
HystrixThreadPoolKey.Factory.asKey("HelloWorldPool")
```

您可能使用 ***HystrixThreadPoolKey*** 而不是一个不同的 ***HystrixCommandGroupKey*** 的理由是，多个命令可能属于同一“所有权组或逻辑功能”，但某些命令可能需要彼此隔离。

这里有一个简单例子：
* 两个用于访问视频元数据的命令
* 组名是“VideoMetadata”
* 命令A对应着#1号资源
* 命令B对应着#2号资源

如果命令A的线程池件变的渐渐饱和，则它不应该阻止命令B执行请求，因为它们每个命中不同的后端资源。因此，我们在逻辑上希望这些命令组合在一起，但希望它们隔离不同，并使用 ***HystrixThreadPoolKey*** 为每个命令分配不同的线程池。

## 请求缓存

通过在 ***HystrixCommand*** 或 ***HystrixObservableCommand*** 对象上实现 [getCacheKey](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#getCacheKey()) 方法来启用请求高速缓存，如下所示：

```java
public class CommandUsingRequestCache extends HystrixCommand<Boolean> {

    private final int value;

    protected CommandUsingRequestCache(int value) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.value = value;
    }

    @Override
    protected Boolean run() {
        return value == 0 || value % 2 == 0;
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(value);
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandUsingRequestCache.java)

因为这取决于请求上下文，我们必须初始化 ***HystrixRequestContext***。
在简单的单元测试中，你可以这样做：

```java
@Test
public void testWithoutCacheHits() {
    HystrixRequestContext context = HystrixRequestContext.initializeContext();
    try {
        assertTrue(new CommandUsingRequestCache(2).execute());
        assertFalse(new CommandUsingRequestCache(1).execute());
        assertTrue(new CommandUsingRequestCache(0).execute());
        assertTrue(new CommandUsingRequestCache(58672).execute());
    } finally {
        context.shutdown();
    }
}
```

通常，此上下文将通过包装用户请求或其他生命周期钩子的Servlet过滤器进行初始化和关闭。
以下是一个示例，显示命令如何在请求上下文中从缓存中检索其值（以及如何查询对象以了解其值是否来自缓存）：

```java
@Test
public void testWithCacheHits() {
    HystrixRequestContext context = HystrixRequestContext.initializeContext();
    try {
        CommandUsingRequestCache command2a = new CommandUsingRequestCache(2);
        CommandUsingRequestCache command2b = new CommandUsingRequestCache(2);

        assertTrue(command2a.execute());
        // this is the first time we've executed this command with
        // the value of "2" so it should not be from cache
        assertFalse(command2a.isResponseFromCache());

        assertTrue(command2b.execute());
        // this is the second time we've executed this command with
        // the same value so it should return from cache
        assertTrue(command2b.isResponseFromCache());
    } finally {
        context.shutdown();
    }

    // start a new request context
    context = HystrixRequestContext.initializeContext();
    try {
        CommandUsingRequestCache command3b = new CommandUsingRequestCache(2);
        assertTrue(command3b.execute());
        // this is a new request context so this 
        // should not come from cache
        assertFalse(command3b.isResponseFromCache());
    } finally {
        context.shutdown();
    }
}
```

## [Request Collapsing](https://github.com/Netflix/Hystrix/wiki/How-To-Use#request-collapsing)

## 请求上下文设置
要使用 reuest-scoped 的功能（请求缓存，Request Collapsing，请求日志），您必须管理 [HystrixRequestContext](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestContext.html) 生命周期（或实现替代[HystrixConcurrencyStrategy](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixConcurrencyStrategy.html)）。

这意味着您必须在请求之前执行以下操作：

```java
HystrixRequestContext context = HystrixRequestContext.initializeContext();
```

然后在这个请求结束时：

```java
context.shutdown();
```

在标准的Java Web应用程序中，可以使用Servlet Filter通过实现类似于以下的过滤器来初始化此生命周期：
```java
public class HystrixRequestContextServletFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
     throws IOException, ServletException {
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        try {
            chain.doFilter(request, response);
        } finally {
            context.shutdown();
        }
    }
}
```
您可以通过向web.xml中添加一个请求过滤器，如下所示：

```xml
<filter>
  <display-name>HystrixRequestContextServletFilter</display-name>
  <filter-name>HystrixRequestContextServletFilter</filter-name>
  <filter-class>com.netflix.hystrix.contrib.requestservlet.HystrixRequestContextServletFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>HystrixRequestContextServletFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 常见模式
下部分是 ***HystrixCommand*** 和 ***HystrixObservableCommand*** 的常见用法和模式。
### 快速失败(Fail Fast)
最基本的执行是一个单一的事情，没有fallback行为。 如果发生任何类型的故障，它将抛出异常。

```java
public class CommandThatFailsFast extends HystrixCommand<String> {

    private final boolean throwException;

    public CommandThatFailsFast(boolean throwException) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.throwException = throwException;
    }

    @Override
    protected String run() {
        if (throwException) {
            throw new RuntimeException("failure from CommandThatFailsFast");
        } else {
            return "success";
        }
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandThatFailsFast.java)

这些单元测试显示它的行为：

```java
@Test
public void testSuccess() {
    assertEquals("success", new CommandThatFailsFast(false).execute());
}

@Test
public void testFailure() {
    try {
        new CommandThatFailsFast(true).execute();
        fail("we should have thrown an exception");
    } catch (HystrixRuntimeException e) {
        assertEquals("failure from CommandThatFailsFast", e.getCause().getMessage());
        e.printStackTrace();
    }
}
```

***HystrixObservableCommand*** 的等效Fail-Fast解决方案将覆盖 ***resumeWithFallback（）*** 方法，如下所示：

```java
@Override
    protected Observable<String> resumeWithFallback() {
        if (throwException) {
            return Observable.error(new Throwable("failure from CommandThatFailsFast"));
        } else {
            return Observable.just("success");
        }
    }
```
### 失败静默(Fail Silent)
失败意味着相当于返回空响应或删除功能。 它可以通过返回null，空的Map，空列表或其他这样的响应来完成。

你可以通过在***HystrixCommand***实例上实现一个 ***getFallback()*** 方法来实现：
![](https://github.com/Netflix/Hystrix/wiki/images/fallback-640.png)

```java
public class CommandThatFailsSilently extends HystrixCommand<String> {

    private final boolean throwException;

    public CommandThatFailsSilently(boolean throwException) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.throwException = throwException;
    }

    @Override
    protected String run() {
        if (throwException) {
            throw new RuntimeException("failure from CommandThatFailsFast");
        } else {
            return "success";
        }
    }

    @Override
    protected String getFallback() {
        return null;
    }
}
```

[查看源码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandThatFailsSilently.java)

```java
@Test
public void testSuccess() {
    assertEquals("success", new CommandThatFailsSilently(false).execute());
}

@Test
public void testFailure() {
    try {
        assertEquals(null, new CommandThatFailsSilently(true).execute());
    } catch (HystrixRuntimeException e) {
        fail("we should not get an exception as we fail silently with a fallback");
    }
}
```

另一个返回空列表的实现将如下所示：

```java
@Override
protected List<String> getFallback() {
    return Collections.emptyList();
}
```

***HystrixObservableCommand*** 的等效的Fail-Silently 解决方案将涉及覆盖 ***resumeWithFallback()*** 方法，如下所示：

```java
@Override
protected Observable<String> resumeWithFallback() {
    return Observable.empty();
}
```

### fallback：static
Fallback可以返回静态嵌入在代码中的默认值。这不会导致功能或服务以“失败沉默”通常的方式删除，而会导致发生默认行为。

例如，如果命令基于用户凭据返回true / false，但命令执行失败，则可以默认为true：

```java
@Override
protected Boolean getFallback() {
    return true;
}
```

***HystrixObservableCommand***的等效静态解决方案将覆盖 ***resumeWithFallback*** 方法，如下所示：

```java
@Override
protected Observable<Boolean> resumeWithFallback() {
    return Observable.just( true );
}
```
### [fallback：stubbed](https://github.com/Netflix/Hystrix/wiki/How-To-Use#fallback-stubbed)

### fallback：网络缓存
有时，如果后端服务失败，可以从诸如memcached的缓存服务检索过时的数据版本。

由于fallbcak调用的是另一个网络上可能失败的点，所以它也需要一个 ***hystrixcommand*** 或 ***hystrixobservablecommand*** 包裹。
![fallback:cache via network](https://github.com/Netflix/Hystrix/wiki/images/fallback-via-command-640.png)

下面的代码演示了 ***CommandWithFallbackViaNetowrk *** 如何在 ***getFallback()*** 方法中执行 ***FallbackViaNetWork***

注意如果fallback失败，它也有一个fallback，它做“fail silent”的方法返回null。

要配置 ***FallbackViaNetwork*** 命令在与从 ***HystrixCommandGroupKey*** 派生的默认 ***RemoteServiceX***不同的线程池上运行，它会将HystrixThreadPoolKey.Factory.asKey（“RemoteServiceXFallback”）注入构造函数。

这意味着 ***CommandWithFallbackViaNetwork*** 将在名为 ***RemoteServiceX*** 的线程池上运行，并且 ***FallbackViaNetwork*** 将在线程池名称上运行RemoteServiceXFallback。

```java
public class CommandWithFallbackViaNetwork extends HystrixCommand<String> {
    private final int id;

    protected CommandWithFallbackViaNetwork(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueCommand")));
        this.id = id;
    }

    @Override
    protected String run() {
        //        RemoteServiceXClient.getValue(id);
        throw new RuntimeException("force failure for example");
    }

    @Override
    protected String getFallback() {
        return new FallbackViaNetwork(id).execute();
    }

    private static class FallbackViaNetwork extends HystrixCommand<String> {
        private final int id;

        public FallbackViaNetwork(int id) {
            super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueFallbackCommand"))
                    // use a different threadpool for the fallback command
                    // so saturating the RemoteServiceX pool won't prevent
                    // fallbacks from executing
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("RemoteServiceXFallback")));
            this.id = id;
        }

        @Override
        protected String run() {
            MemCacheClient.getValue(id);
        }

        @Override
        protected String getFallback() {
            // the fallback also failed
            // so this fallback-of-a-fallback will 
            // fail silently and return null
            return null;
        }
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandWithFallbackViaNetwork.java)

### 主备Fallback

某些系统具有双模式行为 - 主要和次要，或主要和故障转移。

有时次要或故障转移被认为是故障状态，它仅用于fallback; 在这些情况下，它将适合与上述“通过网络缓存”相同的模式。

然而，如果跳转到辅助系统是常见的，例如推出新代码的正常部分（有时这是有状态系统处理代码推送的一部分），则每次使用辅助系统时，主节点将处于故障状态 ，脱扣断路器和点火警报。

这不是所期望的行为，如果没有其他原因，当一个真正的问题发生，将导致警报被忽略，为了避免“狼来了”的情况发生。

因此，在这种情况下，战略反而是将初级和次级之间的转换视为正常，健康的模式，并在其前面放置一个外观模式

![Primary+secondary](https://github.com/Netflix/Hystrix/wiki/images/primary-secondary-example-640.png)

主要和次要 ***HystrixCommand*** 实现是线程隔离的，因为它们正在做网络流量和业务逻辑。 它们可以具有非常不同的性能特性（通常次级系统是静态高速缓存），因此每个的单独命令的另一个好处是它们可以被单独调谐。

您不公开这两个命令，而是将它们隐藏在信号量隔离的另一个 ***HystrixCommand*** 之后，并实现关于是调用主命令还是辅助命令的条件逻辑。 如果主失败和辅助失败，则控制切换到外观命令本身的回退。

外观模式 ***HystrixCommand*** 可以使用信号量隔离，因为它正在进行的所有工作都通过已经线程隔离的另外两个 ***HystrixCommand*** 。 只要外观的 [run()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#run()) 方法不执行任何其他网络调用，重试逻辑或其他“容易出错”的事情，就不必再有一层线程

```java
public class CommandFacadeWithPrimarySecondary extends HystrixCommand<String> {

    private final static DynamicBooleanProperty usePrimary = DynamicPropertyFactory.getInstance().getBooleanProperty("primarySecondary.usePrimary", true);

    private final int id;

    public CommandFacadeWithPrimarySecondary(int id) {
        super(Setter
                .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("PrimarySecondaryCommand"))
                .andCommandPropertiesDefaults(
                        // we want to default to semaphore-isolation since this wraps
                        // 2 others commands that are already thread isolated
                        HystrixCommandProperties.Setter()
                                .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }

    @Override
    protected String run() {
        if (usePrimary.get()) {
            return new PrimaryCommand(id).execute();
        } else {
            return new SecondaryCommand(id).execute();
        }
    }

    @Override
    protected String getFallback() {
        return "static-fallback-" + id;
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(id);
    }

    private static class PrimaryCommand extends HystrixCommand<String> {

        private final int id;

        private PrimaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("PrimaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("PrimaryCommand"))
                    .andCommandPropertiesDefaults(
                            // we default to a 600ms timeout for primary
                            HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(600)));
            this.id = id;
        }

        @Override
        protected String run() {
            // perform expensive 'primary' service call
            return "responseFromPrimary-" + id;
        }

    }

    private static class SecondaryCommand extends HystrixCommand<String> {

        private final int id;

        private SecondaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("SecondaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("SecondaryCommand"))
                    .andCommandPropertiesDefaults(
                            // we default to a 100ms timeout for secondary
                            HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(100)));
            this.id = id;
        }

        @Override
        protected String run() {
            // perform fast 'secondary' service call
            return "responseFromSecondary-" + id;
        }

    }

    public static class UnitTest {

        @Test
        public void testPrimary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", true);
                assertEquals("responseFromPrimary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }

        @Test
        public void testSecondary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", false);
                assertEquals("responseFromSecondary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandFacadeWithPrimarySecondary.java)

### 客户端不执行网络访问

当您执行不执行网络访问的行为，但关注延迟或线程开销是不可接受的，您可以将 ***executionIsolationStrategy*** 属性设置为***ExecutionIsolationStrategy.SEMAPHORE*** ，Hystrix将使用信号量隔离。

下面显示了如何通过代码将此属性设置为命令的默认值（您也可以在运行时通过动态属性覆盖它）。

```java
public class CommandUsingSemaphoreIsolation extends HystrixCommand<String> {

    private final int id;

    public CommandUsingSemaphoreIsolation(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                // since we're doing an in-memory cache lookup we choose SEMAPHORE isolation
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }

    @Override
    protected String run() {
        // a real implementation would retrieve data from in memory data structure
        return "ValueFromHashMap_" + id;
    }

}
```
[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandUsingSemaphoreIsolation.java)

### 使请求缓存无效的Get-Set-Get

如果您正在实施一个Get-Set-Get用例，其中Get接收到足够的流量，请求缓存是需要的，但有时一个Set发生在另一个应该使同一请求中的缓存失效的命令，您可以通过调用 [HystrixRequestCache.clear()](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixRequestCache.html#clear(java.lang.String))。

这里是一个示例实现：

```java
public class CommandUsingRequestCacheInvalidation {

    /* represents a remote data store */
    private static volatile String prefixStoredOnRemoteDataStore = "ValueBeforeSet_";

    public static class GetterCommand extends HystrixCommand<String> {

        private static final HystrixCommandKey GETTER_KEY = HystrixCommandKey.Factory.asKey("GetterCommand");
        private final int id;

        public GetterCommand(int id) {
            super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("GetSetGet"))
                    .andCommandKey(GETTER_KEY));
            this.id = id;
        }

        @Override
        protected String run() {
            return prefixStoredOnRemoteDataStore + id;
        }

        @Override
        protected String getCacheKey() {
            return String.valueOf(id);
        }

        /**
         * Allow the cache to be flushed for this object.
         * 
         * @param id
         *            argument that would normally be passed to the command
         */
        public static void flushCache(int id) {
            HystrixRequestCache.getInstance(GETTER_KEY,
                    HystrixConcurrencyStrategyDefault.getInstance()).clear(String.valueOf(id));
        }

    }

    public static class SetterCommand extends HystrixCommand<Void> {

        private final int id;
        private final String prefix;

        public SetterCommand(int id, String prefix) {
            super(HystrixCommandGroupKey.Factory.asKey("GetSetGet"));
            this.id = id;
            this.prefix = prefix;
        }

        @Override
        protected Void run() {
            // persist the value against the datastore
            prefixStoredOnRemoteDataStore = prefix;
            // flush the cache
            GetterCommand.flushCache(id);
            // no return value
            return null;
        }
    }
}
```

[查看代码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandUsingRequestCacheInvalidation.java)

确认行为的单元测试是：

```java
@Test
public void getGetSetGet() {
    HystrixRequestContext context = HystrixRequestContext.initializeContext();
    try {
        assertEquals("ValueBeforeSet_1", new GetterCommand(1).execute());
        GetterCommand commandAgainstCache = new GetterCommand(1);
        assertEquals("ValueBeforeSet_1", commandAgainstCache.execute());
        // confirm it executed against cache the second time
        assertTrue(commandAgainstCache.isResponseFromCache());
        // set the new value
        new SetterCommand(1, "ValueAfterSet_").execute();
        // fetch it again
        GetterCommand commandAfterSet = new GetterCommand(1);
        // the getter should return with the new prefix, not the value from cache
        assertFalse(commandAfterSet.isResponseFromCache());
        assertEquals("ValueAfterSet_1", commandAfterSet.execute());
    } finally {
        context.shutdown();
    }
}
```
### 迁移

当您将现有客户端库迁移到使用Hystrix时，应该使用 ***HystrixCommadn*** 替换每个“服务方法”。

然后，服务方法应该将调用转发到 ***HystrixCommand*** ，而不在其中包含任何附加的业务逻辑。

因此，在迁移之前，服务库可能如下所示：
![Without Hystrix](https://github.com/Netflix/Hystrix/wiki/images/library-migration-to-hystrix-without-640.png)

迁移后，库的用户将能够通过委派给 ***HystrixCommand*** 的服务外观直接或间接访问 ***HystrixCommand***。
![Migrated](https://github.com/Netflix/Hystrix/wiki/images/library-migration-to-hystrix-with-640.png)