## 添加依赖包

maven项目pom文件添加一下内容：

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

上述命令会将hystrix-core-1.5.6.jar 和它的依赖包一起下载到./target/dependency/. 目录，执行上述命令需要java 6以上。

## Hello World!

Hystrix的最简单的用法如下：

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

这个命令可以这样使用：

```java
String s = new CommandHelloWorld("Bob").execute();
Future<String> s = new CommandHelloWorld("Bob").queue();
Observable<String> s = new CommandHelloWorld("Bob").observe();
```

更多的例子和信息可以在[如何使用](use.md)章节找到。

示例源码可以在[hystrix-examples](https://github.com/Netflix/Hystrix/tree/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples)模块找到。

### 构建

检出源码并且构建

```shell
$ git clone git@github.com:Netflix/Hystrix.git
$ cd Hystrix/
$ ./gradlew build
```

做一个干净的构建：

```shell
$ ./gradlew clean build
```

在构建过程中你可以看到类似这样的信息：
```console
$ ./gradlew build
:hystrix-core:compileJava
:hystrix-core:processResources UP-TO-DATE
:hystrix-core:classes
:hystrix-core:jar
:hystrix-core:sourcesJar
:hystrix-core:signArchives SKIPPED
:hystrix-core:assemble
:hystrix-core:licenseMain UP-TO-DATE
:hystrix-core:licenseTest UP-TO-DATE
:hystrix-core:compileTestJava
:hystrix-core:processTestResources UP-TO-DATE
:hystrix-core:testClasses
:hystrix-core:test
:hystrix-core:check
:hystrix-core:build
:hystrix-examples:compileJava
:hystrix-examples:processResources UP-TO-DATE
:hystrix-examples:classes
:hystrix-examples:jar
:hystrix-examples:sourcesJar
:hystrix-examples:signArchives SKIPPED
:hystrix-examples:assemble
:hystrix-examples:licenseMain UP-TO-DATE
:hystrix-examples:licenseTest UP-TO-DATE
:hystrix-examples:compileTestJava
:hystrix-examples:processTestResources UP-TO-DATE
:hystrix-examples:testClasses
:hystrix-examples:test
:hystrix-examples:check
:hystrix-examples:build

BUILD SUCCESSFUL

Total time: 30.758 secs
```

在一个干净的构建过程中你将看到单元测试运行，看到类似的信息：
```shell
> Building > :hystrix-core:test > 147 tests completed
```