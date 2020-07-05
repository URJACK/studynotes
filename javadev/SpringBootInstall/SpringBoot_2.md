# Spring应用_2

## Java

### 定义controller

@RestController：@Controller与@ResponseBody 两个一起的等价注解

@Controller：指定这个是一个Controller对象

@ResponseBody: 返回数据而不是一个页面，如果是一个对象还会以json的格式返回

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "Hello , This is Spring Boot Application~";
    }
}
```

### 数据校验 @validated

@Email注解配合使用

## Resources

### static

保存静态资源

### templates

使用的模版引擎

### application.properties(application.yml)

只要文件名是application，并且放在resources文件夹下，就可以起作用
yml的配置更为精准

```yml
server:
  port: 3000
```

如果是.properties

```properties
server.port=80
```

.properties文件的编码默认是GBK，需要在Settings->Editor->File Encoding->Properties Files：
1·修改为UTF-8
2·勾选Transparent native-to-ascii conversion

#### @ConfigurationProperties

```java
//Person.java 是一个Bean文件，在该文件中的Person类加上这个注解、以及对应配置文件中的变量名
//使用了Component，才能通过Spring容器进行访问(使用@Autowired)
@Component
@ConfigurationProperties(prefix = "person")
```

```yml
#配置文件application.yml
person:
  firstName: fu
  lastName: fangzhou
  age: 12
  boss: true
  birth: 1996/09/30
  map: {gongjie: shuai,jike: wei}
  list:
    - fangshuai
    - gongjie
    - heling
  dog:
    name: xiaohua
    age: 2
```

#### @Value

#### 两种获取数值的对比

Value对单个属性进行注解也能够从配置文件中获取值

|  |@ConfigurationProperties  |@Value  |
|---------|---------|---------|
|松散语法绑定|支持|不支持|
|SpEL|不支持|支持|
|数据校验|支持|不支持|
|复杂类型封装|支持|不支持|

综上，@Value仅仅是只需要给单个属性获取一个简单的数据，针对复杂的数据仍建议使用@ConfigurationProperties

## Test

DemoApplicationTests.java

```java
@SpringBootTest
class DemoApplicationTests {
 @Autowired  //从Spring容器中获取对象
 Person person;

 @Test
 void contextLoads() {
  System.out.println(person);
 }

}
```
