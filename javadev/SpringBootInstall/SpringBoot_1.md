# Spring Boot

## 基础

### maven相关设置

手动安装maven后，找到maven/conf/settings.xml

#### 设置maven

```xml
<!-- <mirror>标签需要放在<mirrors>标签组下 profile同理 -->
<!-- mirror是配置阿里云镜像 properties是指maven使用的java版本 -->

<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>

<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

```
还有对仓库存储位置的修改
```

#### idea对maven的配置

```
这是必须修改的一步，因为idea中的maven，默认使用的配置文件是另外的，你必须执行下面的步骤，更改idea中maven的默认配置文件
```


找到settings->Build .....->Maven
修改其中的
"Maven Home Directory":maven的安装路径
"User settings file":上文改动的settings.xml
"Local repository":本地的Maven仓库

### 在Maven项目的基础上生成SpringBoot项目

#### step 1 创建

在idea中，创建一个Maven项目

#### step 2 修改pom文件

在创建的maven项目中，我们需要修改maven项目的pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SpringDemo1</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <!--将应用打包-->
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

#### step 3 创建应用

创建应用程序：
"com.basic.HelloWorldMainApplication":

```java
package com.basic;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @SpringBootApplication 标注这个是一个 SpringBoot 应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

"com.basic.controller.HelloController":

```java
package com.basic.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello , This is Spring Boot";
    }
}
```

### 部署(SpringBoot打包成一个jar包)

pom.xml中，引用了build插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

这让我们可以使用打包功能:
在idea的左下角->选中maven->选中Lifecycle->双击package
从而整个Spring应用成为一个jar包

### maven项目的手动创建过程

使用SpringInitializer 生成的项目，在idea打开时，并不算一个默认的maven项目，对pom.xml文件的改动并不能让maven进行工作。
必须使用右键，点击pom.xml，add as a maven project。

### gradle 的使用

针对单个项目的配置，找到build.gradle

```gradle
repositories {
 maven { url 'https://maven.aliyun.com/repository/public/' }
 mavenLocal()
 mavenCentral()
}
```

针对所有项目的配置，在gradle的安装目录下
在init.d文件夹下，创建后缀名是.gradle的文件即可
如init.gradle

```gradle
allprojects {
    repositories {
         maven {
             name "aliyunmaven"
             url "http://maven.aliyun.com/nexus/content/groups/public/"
         }
    }
}
```

## 深入

### pom文件

我们对pom文件添加了几个部分的内容：
"parent":它的artifactid是spring-boot-starter-parent，可见这个应该是任何一个spring-boot-starter都应该填写的父类那种感觉？应该是能够继承这个父类的较多的功能
，大概吧。
"dependencies":spring-boot-starter-web，我们给这个应用加上了这个依赖，从而才能够正常的运行web的相关功能
"build->plugins":spring-boot-maven-plugin，这个插件仅仅用来打包

### starter是什么

在pom文件中，....starter...的出现概率非常高，它的作用简要来说就是，每一个starter自身都会去依赖一大堆的包，但是对于用户来说，仅仅只需要引用这一个starter就够了。

### 注解

#### @SpringBootApplication

包含"@SpringBootConfiguration":这个表明是一个配置类
包含"@EnableAutoConfiguration":这个注解可以让SpringBoot的web应用去自动导入其他的文件（前提是必须处在自己所属的包内）
