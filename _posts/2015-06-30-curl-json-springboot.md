---
layout: post
title:  "利用Curl发送json数据到后台SpringBoot＋MongoDB测试CRUD"
tags: [Server]
---

web后端的开发涉及很多很多技术，要相互配合好需要不断的练习。本文要做的事情是实现一个服务器端程序，运用SpringBoot框架，结合MongoDB数据库。然后用刚学的curl命令发送json数据来测试数据库的CRUD功能。

#### 1. 新建一个工程并建立pom文件

```
<?xml version="1.0" encoding="UTF-8"?>`
<project xmlns="http://maven.apache.org/POM/4.0.0"``xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0`http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

   <groupId>com.hunter</groupId>
   <artifactId>SpringBootMongo</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <packaging>jar</packaging>

   <name>SpringBootMongo</name>
   <description>Demo project for Spring Boot</description>

   <!-- lookup parent from repository -->
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.1.6.RELEASE</version>
      <relativePath/>
   </parent>

   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-mongodb</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-ws</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
      </dependency>
   </dependencies>

   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <start-class>com.hunter.Application</start-class>
      <java.version>1.7</java.version>
   </properties>

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>

</project>
```

**maven会帮助开发者进行依赖管理。然后开发者就可以专注于功能模块的开发。**

#### 2. 在src/main/java/下新建目录com/hunter/，放入java文件
*Application.java*

```
package com.hunter;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
@EnableAutoConfiguration
public class Application {
    public static void main(String[] args) {

        SpringApplication.run(Application.class, args);
    }
}
```

在com/hunter/employee目录下面新建三个文件

*Employee.java*

```
package com.hunter.employee;

import org.springframework.data.annotation.Id;

public class Employee {

    @Id
    private String id;

    private String name;
    private String title;

    public Employee(){};
    public Employee(String id, String name, String title){
        super();
        this.id=id;
        this.name=name;
        this.title=title;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

*EmployeeController.java*

```
package com.hunter.employee;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/employee")
public class EmployeeController {
    @Autowired
    EmployeeRepository repository;

    //create
    @RequestMapping(value = "", method = RequestMethod.PUT)
    public Employee createEmployee(@RequestBody Employee employee){
        return repository.save(employee);
    }

    //read
    @RequestMapping(value = "",method = RequestMethod.GET)
    public List<Employee> readEmployee(@RequestParam(required = false) String name,
                                       @RequestParam(required = false) String title) {
        if (name != null) {
            return repository.findByName(name);
        } else if (title != null) {
            return repository.findByTitle(title);
        } else {
            return repository.findAll();
        }
    }

    //update
    @RequestMapping(value = "/{id}", method = RequestMethod.POST)
    public Employee updateEmployee(@PathVariable String id, @RequestBody Employee employee){

        Employee em = repository.findById(id);
        Employee newEm = new Employee(em.getId(),employee.getName(),employee.getTitle());
        return repository.save(newEm);
    }

    //delete
    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public void deleteEmployee(@PathVariable String id) {

        repository.delete(id);
    }
}
```

EmployeeRepository.java

```
package com.hunter.employee;

import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.List;


public interface EmployeeRepository extends MongoRepository<Employee, String> {
    public List<Employee> findByName(String name);
    public List<Employee> findByTitle(String title);

    public Employee findById(String id);
}
```

#### 3. 构建工程进行测试
一般我们都倾向于用ide去搭建工程以及测试，毕竟命令行做不到断点检查。
在终端cd到pom所在目录，可以直接用maven命令建立IDEA的工程

`$ mvn idea:idea`

然后就可以使用IntellijIDEA工作了。

------------------------
接下来测试这个程序：

1. 打开MongoDB的接口。 `$ mongod —dbpath ~/MongoDBPath`
2. 运行程序 
3. 终端输入 `$ curl http://localhost:8080/employee`
会返回空数据
[]
因为现在数据库什么都没有。

```
$ curl -v -i -H "Accept: application/json" -H "Content-Type: application/json" -X PUT -d '{"id":1,"name":"charlie","title":"here"}'  http://localhost:8080/employee/
```
发送一个json数据包到服务器，如果运行正常，数据库已经写入一条记录了

```
$ curl -v -i -X GET http://localhost:8080/employee
```
此时再次查询，就已经会返回一条关于charlie的数据了

```
$ curl -v -i -X POST -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"id":1,"name":"jane","title":"leader"}' http://localhost:8080/employee/1
```

```
$ curl -v -i -X DELETE http://localhost:8080/employee/1
```
删除id为1的数据，也就是charlie没了。

**强调，千万要注意引号的正确性。。。有时候，由于文本编辑工具的原因，引号会有误，也有可能造成curl命令错误。例如印象笔记就会把双引号变成另一种类型。。坑。。**




