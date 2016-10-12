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

你可以使用***execute()***方法同步执行一个***HystrixCommand***，如以下示例所示：

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

您可以使用queue（）方法异步执行HystrixCommand，如以下示例所示：

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

* ***observe()*** - 返回一个“hot” ***Observable***立即执行命令，在订阅之前，因为 ***Observable*** 通过 ***ReplaySubject*** 过滤，你不会有丢失任何项目的危险。
* ***toObservable()*** - 返回一个“cold”Observable，它不会执行命令并开始发出其结果，直到您订阅了 ***Observable***
> ***toObservable()*** 创建的对象，不能只想，只能订阅

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

## 回退
## 错误传播
## 命令名称
## 命令分组
## 命令线程池
## 请求缓存
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