# mabatis学习
## 简言
### 软件的三层架构
界面层、业务逻辑层、数据访问层(mabatis所处的位置)
```
用户->界面层->业务逻辑层->数据访问层->数据库
```
### 框架(Framework)是一个模版、一个"半成品"的软件
```
框架自己预设了一些模块，自己补充新的功能的时候，可以利用框架提供的模块
```
### 与jdbc的比较
```
1·jdbc的重复的代码太多
2·业务逻辑代码与数据操作的代码混杂在一起，没有解耦
3·有多个对象需要重复的创建和销毁
```
### Mabatis的历史
```
apache 开源项目 ibatis  --2010
github 的 Mybatis 	--2013
mabatis 提供了两个核心功能：SQLMappings 和 数据库访问
```
SQLMappings
```
将数据库的一行数据，映射为一个java对象
```
数据库访问
```
对数据库执行增删改查
```
### 使用mabatis比jdbc的好处
```
mabatis自动创建jdbc所需要的对象
mabatis自动关闭jdbc对象所创建的链接
mabatis提供了将sql对象转为java单个对象、和java集合对象的能力
开发人员只需要自己提供sql语句即可
```
## springboot-Mabatis
### 全局的配置
pom.xml中，我们引入两个依赖，一个是springboot的mybatis，另一个是mysql的connector
```
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```
其次全局的配置文件中application.properties中，我们加入以下内容
```
mybatis.type-aliases-package=com.example.demo.entity
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.url = jdbc:mysql://localhost:3306/mblearn?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
spring.datasource.username = root
spring.datasource.password = admin
```

url字段与driverClassName字段需要特别注意

driverClassName不要使用"com.mysql.jdbc.Driver"，需要使用带cj的

url"jdbc:mysql://localhost:3306/mblearn?"中的"mblearn"就是具体使用的数据库名字，并且因为驱动的版本问题，一定要加上"&serverTimezone=GMT"

### 使用

/mapper/UserMapper中
```java
package com.example.demo.mapper;

import com.example.demo.entity.StudentEntity;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface UserMapper {
    @Select("SELECT * from student")
    @Results({
            @Result(property = "id", column = "id"),
            @Result(property = "name", column = "name"),
            @Result(property = "email", column = "email"),
            @Result(property = "age", column = "age")
    })
    List<StudentEntity> getAll();

    @Select("SELECT * from student WHERE id = #{id}")
    @Results({
            @Result(property = "id", column = "id"),
            @Result(property = "name", column = "name"),
            @Result(property = "email", column = "email"),
            @Result(property = "age", column = "age")
    })
    StudentEntity getOne(long id);

//    @Insert("INSERT INTO student(name,email,age) VALUES(#{name},#{email},#{age})")
//    void insert(StudentEntity studentEntity);

    @Insert("INSERT INTO student(name,email,age) VALUES(#{name},#{email},#{age})")
    void insert(String name, String email, int age);

    @Update("UPDATE student SET name=#{name},email=#{email},age=#{age} WHERE id =#{id}")
    void update(StudentEntity studentEntity);

    @Delete("DELETE FROM student WHERE id =#{id}")
    void delete(long id);

    @Delete("truncate student;")
    void clear();
}
```
需要注意的是insert以及后续的各种方法中，传入的参数只能是**两种方式**，要不然**都传基本类型**，要不然**只传对象类型**。如果传入的参数中例如为这样，既包括对象、又包括基本类型。
```
    @Insert("INSERT INTO student(name,email,age) VALUES(#{name},#{email},#{newage})")
    void insert(StudentEntity studentEntity,int newage);
```
这样一定会报错

单元测试代码
```java
package com.example.demo;

import com.example.demo.entity.StudentEntity;
import com.example.demo.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class MabatisLearningApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Test
    void daoSelect() {
        System.out.println(userMapper.getAll());
        System.out.println(userMapper.getOne(1));
    }

    @Test
    void daoInsert() {
        StudentEntity studentEntity = new StudentEntity();
        studentEntity.setAge(15);
        studentEntity.setEmail("52125@af.com");
        studentEntity.setName("fanzcbe");
//		userMapper.insert(studentEntity,"trueName");
        userMapper.insert(studentEntity.getName(), studentEntity.getEmail(), studentEntity.getAge());
    }

    @Test
    void daoUpdate() {
        StudentEntity studentEntity = new StudentEntity();
        studentEntity.setId(2);
        studentEntity.setAge(18);
        studentEntity.setEmail("51252125@af.com");
        studentEntity.setName("fazcnzcbe");
        userMapper.update(studentEntity);
    }

    @Test
    void daoDelete() {
//		userMapper.delete(3);
        userMapper.clear();
    }
}
```