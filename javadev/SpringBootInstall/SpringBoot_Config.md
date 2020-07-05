# SpringBoot的配置

## SpringBoot中配置的使用流程

在SpringBoot中，使用配置文件进行配置的工作流程如下：

1·写好配置文件

2·给**数据类A**(专门给配置文件写的数据类)加上配置文件的注解，来获取值

3·给**数据类A**加上@*Component*注解，从而能让**它的一个对象**可以被Spring框架以其他方式(如@*Autowired*)获取到

## 配置文件的相关注解

### 注意事项

数据类中的所有对象，必须加上对应的Getter和Setter方法，否则最后配置文件的取值会出现一些问题(比如值为空)

### @ConfigurationProperties(prefix = "person")

这个注解添加于数据类，是从全局配置文件中，取得前缀为"person"的配置信息

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
```

### @Value

这个注解应用于数据类的一个成员变量中，也是从全局配置文件中进行读取。

### @PropertySource("classpath:person.properties")

这个注解可以指定具体的配置文件，好像只能针对".properties"后缀的配置文件

### Properties文件

可以使用站位符

```properties
person.firstName=付
person.lastName=枋洲
person.age=${random.int}
person.boss=true
person.birth=1996/09/30
person.map.k1=v1
person.map.k2=v2
person.list=a,b,c

person.dog.name=${person.firstName}${person.lastName}的狗
person.dog.age=2
```

## Spring的配置导入(Traditional)

### @ImportResource

```java
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class DemoApplication {
```

### Spring beans文件的写法

尤其注意xsi:schemaLocation有两段url，少了一段都不行

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans  xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="helloService" class="com.example.demo.service.HelloService"/>
</beans>
```

### Spring 获取到 Context

```java
    @Autowired
    ApplicationContext ioc;

    @Test
    void testHelloService(){
        System.out.println(ioc.containsBean("helloService"));
    }

```

## SpringBoot的配置导入(Recommended)

### 配置类与Bean注解

```java
/**
 * configuration 指明是一个配置类
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值 添加到容器：容器中的组件默认id为 该方法的方法名
    @Bean
    public HelloService helloService(){
        return new HelloService();
    }
}

```

## SpringBoot多环境的配置文件

### properties的配置方法

1·编写多个前缀名相同的配置文件

2·在主配置文件中，去启用不同后缀的配置文件

项目结构如下

```properties
application.properties
application-dev.properties
application-prod.properties
```

3·properties的文件的内容

application.properties

```properties
spring.profiles.active=dev
#spring.profiles.active=prod
```

### yml的配置方法

1·在单个yml文件就可以描述这种多环境关系，使用分隔号分隔开这些不同的环境

```yml
spring:
  profiles:
    active: prod
#    active: dev
---
server:
  port: 8080
spring:
  profiles: dev
---
server:
  port: 80
spring:
  profiles: prod
```

### 命令行启用具体的环境

显而易见，**命令行**对环境选择的**优先级更高**

命令行中加入参数 --spring.profiles.active=dev

在Idea配置命令行参数的过程：

1·右上角运行按钮旁，Edit configuration

2·program arguments 添加这段参数

即便是最后打包成为了jar包(默认是prod环境)，也可以使用额外参数启用其他环境

```cmd
java -jar ...jar --spring.profiles.active=dev
```

## 关于配置其他知识

### 内部配置文件的默认存放位置

优先级，从高到低

> 需要注意的是，只有/src路径下的资源才会被打包...意味着前两种方式虽然具有较高的优先级，但是是不会被打包进去的

```txt
file:/config
file:
file:/src/main/resources/config
file:/src/main/resources
```

### 配置文件的常用属性

```properties
# 服务器的端口号
server.port=8081
# 这个属性会让你的url访问路径，变为:"/ffz01/hello"
server.context-path=/ffz01
# 设置一个环境的名称(properties与yml 的配置方法见上文)
# 选择一个具体的环境名，并启动
spring.profiles.active=dev
# 选择一个外部的配置文件(但是在开发阶段是不起作用的，是在项目打包之后，通过命令行的参数进行使用的)
# 指定的这个外部配置文件具有较高的优先级
# --spring.config.location=G:/application.properties
```

### 外部配置的加载方式

按优先级从高到低

1·命令行参数:

java -jar ...... --server.port

2·jar包外部带profile的文件

application-(profile).properties

3·jar包内部带profile的文件

4·jar包外部不带profile的文件

application.properties

5·jar包内部不带profile的文件

> 什么叫jar包外部和jar包内部呢？
> 这里的jar包外部是指将代码打包成jar包之后，jar包存储的路径记为jarpath/a.jar。那么jarpath/application.properties就是一个外部的配置

## 自动配置原理

@EnableAutoConfiguration 这个注解可以自动配置
