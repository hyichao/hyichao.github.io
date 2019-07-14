---
layout: post
title:  "配置spring boot工程"
tags: [Server]
---

快速配置spring boot的文档。

<http://docs.spring.io/spring-boot/docs/1.2.3.RELEASE/reference/htmlsingle/#getting-started>

为了下次不用翻看英文文档，记录如下（平台环境：Mac+Intellij）

* 1. 安装java jdk
* 2. 安装maven
* 3. 利用homebrew下载springboot

```
$ brew tap pivotal/tap
$ brew install springboot
```

* 4. 安装完毕后直接在窗口输入

`$ spring  `

有一些反应就意味着安装成功了。

* 5. 创建一个pom.xml文件，这是maven的核心，也将是工程的管理核心文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.2.3.RELEASE</version>
    </parent>

    <!-- Additional lines to be added here... -->

</project>
```
然后直接用`$ mvn package`可以根据pom文件的内容创建一个target文件夹

* 6. 在pom文件中添加依赖包
可以先利用下面这个命令显示一下目前拥有的依赖包
`$ mvn dependency:tree`

然后在pom文件中 parents部分的下方添加dependency

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
   
然后再次使用dependency：tree的命令，就会重新scan整个工程。
特别地，如果电脑本身没有dependency里面的库，就会用repository的信息里面找到下载点，然后下载到本地仓库。所谓本地仓库，就是保存在M2_HOME路径下。

* 7. 新建java代码
maven的默认编译路径是src/main/java/ 所以建好目录后在该目录下新建java文件，也可以再建立一个包路径

```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController@EnableAutoConfigurationpublic class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }
}
```
文档解释了一下这个代码里面一些重要元素的意义。

* 8. 切回到工程根目录下，运行以下命令
`$ mvn spring-boot:run`

此时如果还缺少依赖包，maven依然会义无反顾地下载依赖包。
然后就会运行程序

* 9. 如果说，第八步就想debug模式下打开程序的话，第九步就是release一个可执行文件。
在pom文件添加一段关于build的信息

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
 </build>
```
然后用下面的命令重新打包工程
`$ mvn package`

完成后，target路径下面会出现jar包。用
`$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar`

就可以执行这个jar包

---
至此，spring boot的quick start已经结束。我们甚至可以小修小改源码，看看会发生什么。
但是如果要投入开发，这远远不够。由于spring的封装特性，有很多内容我们还需要慢慢探索。


