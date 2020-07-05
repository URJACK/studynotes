# Mybatis-plus
## 成功运行时的相关配置
### maven配置

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1.tmp</version>
</dependency>
```

### application配置

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mblearn?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8
    username: root
    password: admin
```

### 文档结构的配置

```
com/ffz/demo/entity/Student
com/ffz/demo/dao/StudentMapper
这里的Student必须和数据库的表名student一致！"类名"和"表名"具有映射关系
```

### 启动类的配置

```
package com.ffz.demo;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.ffz.demo.dao")
public class MybatisPlusLearnApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusLearnApplication.class, args);
    }
}
```

## 使用语法

### 实体类定义

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
//@TableName(value = "student")
public class Student {
    @TableId(type = IdType.AUTO)
    Integer id;
//    @TableField("name")
    String name;
    String email;
    Integer age;
}
```

```txt
@TableName 指定表名
@TableId 如果不指定，会找到变量名默认为id的对象
@TableField 指定属性名
```

## 其他使用场景

### 排除非表字段

#### 方法1：transient修饰变量，中止该变量的序列化

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
//@TableName(value = "student")
public class Student {
    @TableId(type = IdType.AUTO)
    Integer id;
//    @TableField("name")
    String name;
    String email;
    Integer age;
    transient String remark;
}
```

```
在这个场景下，remark将不会被进行序列化
```

#### 方法2：static修饰变量，存储在类中，序列化的时候不会采取该变量

```java
static String remark;
```

#### 方法3：使用mybatis-plus 的@TableField注解的exist属性

```java
@TableField(exist = false)
private String remark;
```

