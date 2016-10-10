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

#### HystrixObservableCommand Equivalent

使用 [***HystrixObservableCommand***](http://netflix.github.io/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixObservableCommand.html) 而不是 ***HystrixCommand*** 的等效Hello World解决方案将涉及覆盖构造方法，如下所示：

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

### HystrixObservableCommand
对于 ***HystrixObservableCommand*** 没有简单的等价执行，但是如果你知道这样的命令产生的 ***Observable*** 总是只产生一个值，你可以通过应用 ***.toBlocking().toFuture().get()*** 来模拟execute的行为 通知到 ***Observable***。

## 异步执行
## 响应式执行
## 响应式命令
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