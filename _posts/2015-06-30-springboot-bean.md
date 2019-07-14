---
layout: post
title:  "Spring注入Bean的几种方式"
tags: [Server]
---

首先，Bean的概念需要清晰：
Bean是一种符合一定要求的Java类

* 1. 有默认的构造函数
* 2. 对于私有属性，有setter和getter

满足了上述的条件，在工程中就可以在XML文件中定义类的实例。一开始我不是很理解，为什么要在XML中去定义实例，而不是正正常常的代码中创建。后来看了一些资料，敲了一些sample代码，看了一下《Spring实战》，或多或少有一点理解。在实际变成中，我们需要实现多态，主要用到继承和接口。一般开发中提倡“面向接口编程”，从而降低代码的藕合度。所以Spring的思想中，把类的实例化抽取出来，而代码中只需要做核心的逻辑实现。类的实例化在Spring里面称为wiring（装配）或者inject（注入）。咋一看，装配这个词十分唬人，实际上就是把类实例化，然后把一些参数“塞”到这个对象里面，同时把这个对象和别的对象的依赖关系一并搞清楚，如谁是谁的私有属性之类的。

通过学习，我个人归纳了一下Spring里面所谓的“依赖注入”的几种方式。

* 1. 完全通过XML文件装配
* 2. 用Annotation自动装配
* 3. 用Java装配
* 4. 还有一些不伦不类的，介于使用XML和使用Annotation之间的做法，不提倡

###第一步，配置XML文件
首先，无论用何种方式装配，都至少需要一个XML文件，做一些基础的调起启动
新建一个XML文件，样式如下

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd
    ">

     ...
     add bean here...
     using <bean></bean>

</beans>
```

#### <完全通过XML文件装配>
新建一个XML文件，样式如下

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd
    ">

    <bean id="charlie" class="com.hunter.PoeticJuggler">
        <constructor-arg value = "25"/>
        <constructor-arg ref = "jingyesi"/>
    </bean>

    <bean id="jane" class = "com.hunter.Instrumentalist">
        <property name = "song" value="Jingle Bells"/>
    </bean>

</beans>
```
其中用<bean></bean>来标写JavaBean的内容，id最好是工程内的唯一，或者至少是package里面的唯一。id就会去寻找class参数所示的类定义，然后实例化这个类。要使用这个对象，就需要在Java代码里面用getBean。

#### <用Annotation装配>
把XML文件里面的bean删掉。咱们开始用Annotation来实现自动装配。
Annotation指的是Java代码里面，用@开头的一些关键词。
一般Spring是默认ban掉这个功能的。所以需要在XML文件里面添加一些东西。如下。
`<context:annotation-config>`

然后在Java代码里面，就可以用一些关键词去自动装配了。Spring里面使用@Autowired，例如：

```
public class Instrumentalist implements Performer{
    private Instrument instrument;

    @Autowired
    public void setInstrument(Instrument instrument){
        this.instrument = instrument;
    }
}
```
这里，Autowired就自动把Instrument这个类实例化了。实际上有了这个Annotation都可以不需要下面那个setter函数。
假如Autowired写在属性上面，也是可行的做法，而且那样还可以省去setter，如
```
public class Instrumentalist implements Performer{
    @Autowired
    private Instrument instrument;
    }
}
```
实际上这里的Autowired是有比较大的局限性的。实际使用的时候估计要对自动装配做一下限定。

除了这种方法， 还有一种更狠的自动装配，也叫自动监测。首先XML文件中写入这么一句
`<context:component-scan>`
然后Spring会自动检测代码中含有关键字@Component的class，然后注册成为bean，id为类名，如

```
@Component
public class Instrumentalist implements Performer{

    public  Instrumentalist(){

    }
    public void perform(){
        System.out.println("Playing "+song+" : ");
        instrument.play();
    }
｝
```
注意一点，这个类里面必须要用默认构造函数，否则会报错。JavaBean的要求正是有默认构造函数。
如果希望这个bean的名字是自己命名的，则可以写成
`@Component("Charlie")`
则bean注册id为charlie

#### <基于Java的装配>
因为某些原因，有人很讨厌尖括号的XML配置。所以Spring也提供了基于Java的配置方法。
不过，还是要在XML文件里面，加入这么一句话。
`<context:component-scan base-package=“com.hunter">`
后面的base-package是所在的包名，
然后新建一个Java类，如下

```
package com.hunter;
import org.springframework.context.annotation.Configuration;
@Configuration
public class SpringConfig{
    @Bean
    public Performer duke(){
        return new Juggler();
    }
}
```
其中@Bean意味着接下来是一个bean的定义
Performer是返回的类
duke相当于id

小结：
作为初学者，不好客观评价各种注入的方式，只能提一提个人的直觉。
第一种显式定义bean显然比较粗暴，实际开发中估计不建议这么做，因为显式定义将产生冗长的XML文件，根本不便于维护；第二种方式代码上看比较不错，代码简单，不过同时可读性相对弱一点，只有理解了bean注入的开发者才能体会代码，而且一旦不慎容易装配错误；第三种做法比较有意思，不过有点失去Spring精髓的味道，感觉Spring的开发还是有点尖括号比较有feel。。嗯。。走题了～然而现在有一个叫做SpringBoot的项目，基本上就不需要任何XML文件，变为需要记住不少Annotation。不过Annotation的确很有好处，起码编译器可以帮助我们报错，如果是XML文件，经常出错了还找不到出错的点，就比较麻烦。
