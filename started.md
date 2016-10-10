## 添加依赖包

maven项目pom文件添加一下内容

```xml
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>1.5.6</version>
</dependency>
```

> 2016年10月10日为止最新版本为1.5.6

lvy添加：

```xml
<dependency org="com.netflix.hystrix" name="hystrix-core" rev="1.5.6" />
```

如果你需要下载jar而不是使用构建系统，创建一个所需版本的maven pom文件，例如：

```xml
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.netflix.hystrix.download</groupId>
    <artifactId>hystrix-download</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Simple POM to download hystrix-core and dependencies</name>
    <url>http://github.com/Netflix/Hystrix</url>
    <dependencies>
        <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-core</artifactId>
            <version>1.5.6</version>
            <scope/>
        </dependency>
    </dependencies>
</project>
```

执行：

```shell
mvn -f download-hystrix-pom.xml dependency:copy-dependencies
```

上述命令会将hystrix-core-1.5.6.jar 和它的依赖包一起下载到  ./target/dependency/. 目录，执行上述命令需要java 6以上。

## HelloWorld

Hystrix的最简单的用法如下:

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

[查看源码](https://github.com/Netflix/Hystrix/blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloWorld.java)
