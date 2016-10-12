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
## 请求折叠
## 请求上下文设置
## 常见模式
### 快速失败
### 失败静默
### 回退：静态
### 回退：stubbed
### 回退：网络缓存
### 主备回退
### 客户端不执行网络访问
### 使用无效请求缓存的Get-Set-Get
### 迁移