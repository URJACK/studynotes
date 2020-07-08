# Mybatis-plus Insert、Update And Delete

本章主要是mybatis-plus的更新和删除，掌握常用的更新、删除方法是十分有必要的。

## 增加insert

```java
    @Test
    void insertById() {
        Student student = new Student();
        student.setName("lubada" + Math.floor(Math.random() * 124));
        double a = Math.floor(Math.random() * 1255236) + 1000000;
        student.setEmail(a + "@qq.com");
        student.setAge((int) (Math.floor(Math.random() * 10) + 10));
        Date day = new Date();
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        student.setCreateTime(df.format(day));

        student.setId(3);
        studentMapper.insert(student);
    }
```

```
DEBUG==>  Preparing: INSERT INTO student ( name, email, age, create_time ) VALUES ( ?, ?, ?, ? ) 
DEBUG==> Parameters: lubada19.0(String), 2094848.0@qq.com(String), 16(Integer), 2020-07-08 15:41:40(String)
```

## 更新update

### 通过Id更新

```java
    @Test
    void updateById() {
        Student student = new Student();
        student.setId(2);
//        如果一个属性不存在，那么就不会更新这个属性
//        “主动”设置为null，或者是被动本身就是null，都符合这个结果
//        student.setName(null);
        student.setName("范帅");
        student.setEmail("4162123123@qq.com");
        student.setAge(12);
        int rows = studentMapper.updateById(student);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: UPDATE student SET name=?, email=?, age=? WHERE id=? 
DEBUG==> Parameters: 范帅(String), 4162123123@qq.com(String), 12(Integer), 2(Integer)
```

### 通过wrapper筛选条件

```java
    @Test
    void updateByWrapper() {
        UpdateWrapper<Student> wrapper = new UpdateWrapper<>();
        wrapper.eq("name", "范帅").or().like("name", "帅").eq("age", 12);
        Student student = new Student();
        student.setEmail("laofanshuai@qq.com");
        student.setAge(22);
        int rows = studentMapper.update(student, wrapper);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: UPDATE student SET email=?, age=? WHERE (name = ? OR name LIKE ? AND age = ?) 
DEBUG==> Parameters: laofanshuai@qq.com(String), 22(Integer), 范帅(String), %帅%(String), 12(Integer)
```

### 通过对象构建wrapper

update使用的是updateWrapper。而删除和查询使用的是都是queryWrapper

原因很简单，查看它们构造sql语句的方式就可以看出来了

```java
    @Test
    void updateByEntity() {
        Student whereStudent = new Student();
        whereStudent.setName("范帅");
        UpdateWrapper<Student> wrapper = new UpdateWrapper<>(whereStudent);
        Student student = new Student();
        student.setName("范少帅");
        student.setAge(12);
        int rows = studentMapper.update(student, wrapper);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: UPDATE student SET name=?, age=? WHERE name=? 
DEBUG==> Parameters: 范少帅(String), 12(Integer), 范帅(String)
```

### 直接在UpdateWrapper上进行更新操作

```java
    @Test
    void updateDirectlyByWrapper() {
        UpdateWrapper<Student> wrapper = new UpdateWrapper<>();
        wrapper.eq("name", "范少帅").set("name", "范帅");
        int rows = studentMapper.update(null, wrapper);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: UPDATE student SET name=? WHERE (name = ?) 
DEBUG==> Parameters: 范帅(String), 范少帅(String)
```

### lambda构造

```java
    @Test
    void updateByLambda() {
        LambdaUpdateWrapper<Student> wrapper = new LambdaUpdateWrapper<>();
        wrapper.eq(Student::getName, "范帅").set(Student::getAge, 24);
        int rows = studentMapper.update(null, wrapper);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: UPDATE student SET age=? WHERE (name = ?) 
DEBUG==> Parameters: 24(Integer), 范帅(String)
```

### 链式更新

```
    @Test
    void updateByChain() {
        LambdaUpdateChainWrapper<Student> wrapper = new LambdaUpdateChainWrapper<>(studentMapper);
        boolean success = wrapper.eq(Student::getName, "范帅").set(Student::getAge, 24).update();
        //如果一条也没有更新到，success就是false。至少更新一条，success就是true
        System.out.println(success ? "更新成功" : "更新失败");
    }
```

```
DEBUG==>  Preparing: UPDATE student SET age=? WHERE (name = ?) 
DEBUG==> Parameters: 24(Integer), 范帅(String)
```

## 删除delete

### 根据id删除

```java
    @Test
    void deleteById() {
        int rows = studentMapper.deleteById(3);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: DELETE FROM student WHERE id=? 
DEBUG==> Parameters: 3(Integer)
```

### 根据批量Id删除

```java
    @Test
    void deleteByBatchIds() {
        int rows = studentMapper.deleteBatchIds(Arrays.asList(3, 4));
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: DELETE FROM student WHERE id IN ( ? , ? ) 
DEBUG==> Parameters: 3(Integer), 4(Integer)
```

### 根据map删除

```java
    @Test
    void deleteByMap() {
        Map<String, Object> map = new HashMap<>();
        map.put("name", "范帅");
        map.put("age", 23);
        int rows = studentMapper.deleteByMap(map);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: DELETE FROM student WHERE name = ? AND age = ? 
DEBUG==> Parameters: 范帅(String), 23(Integer)
```

### 根据wrapper删除

使用的和查询的wrapper是一样的，都是QueryWrapper。因为SQL语句结构类似。

```
    @Test
    void deleteByWrapper() {
        LambdaQueryWrapper<Student> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(Student::getName, "daxiake");
        int rows = studentMapper.delete(wrapper);
        System.out.println("影响记录数 : " + rows);
    }
```

```
DEBUG==>  Preparing: DELETE FROM student WHERE (name = ?) 
DEBUG==> Parameters: daxiake(String)
```