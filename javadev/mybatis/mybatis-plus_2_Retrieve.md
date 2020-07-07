# Mybatis-Plus Retrieve

ORM中的最核心的功能之一，查询。

## 基础查询方法

### 通过Id查询(selectById)

```java
    @Test
    void selectById() {
        Student student = studentMapper.selectById(10);
        if (student != null) {
            System.out.println(student);
        }
    }
```

```
SELECT id,name,email,age FROM student WHERE id=? 
```



### 通过多个Id查询(selectBatchIds)

```java
@Test
void selectByIds() {
    List<Long> idList = Arrays.asList(1L, 2L, 12L, 4L);
    //此处因为没有查到id=12的数据，students.size() == 3，而不是4.
    //不会出现空对象
    List<Student> students = studentMapper.selectBatchIds(idList);
    for (Student student :
            students) {
        System.out.println(student);
    }
}
```

```
SELECT id,name,email,age FROM student WHERE id IN ( ? , ? , ? , ? ) 
```



### 通过条件查询(selectByMap)

```java
    @Test
    void selectByMap() {
        Map<String, Object> map = new HashMap<>();
        map.put("name", "帅哥");
        map.put("age", 12);
        List<Student> students = studentMapper.selectByMap(map);
        for (Student student :
                students) {
            System.out.println(student);
        }
    }
```

```
SELECT id,name,email,age FROM student WHERE name = ? AND age = ? 
```

## 条件构造器的查询方法（selectList方法调用queryWrapper）

以下是一个使用条件构造器的实例

```java
    @Test
    void selectByWrapper(){
        QueryWrapper<Student> queryWrapper = new QueryWrapper<>();
        queryWrapper.like("name","帅哥").lt("age",40);
        List<Student> students = studentMapper.selectList(queryWrapper);
        System.out.println(students);
    }
```

```
SELECT id,name,email,age FROM student WHERE (name LIKE ? AND age < ?) 
```

### 其他的条件构造器如下：

#### 等于、不等于

```
eq("name", "老王")--->name = '老王'
ne("name", "老王")--->name <> '老王'
```

#### 大于、大于等于、小于、小于等于

```
gt("age", 18)--->age > 18
ge("age", 18)--->age >= 18
lt("age", 18)--->age < 18
le("age", 18)--->age <= 18
```

#### 在...之间、在...之外

```
between("age", 18, 30)--->age between 18 and 30
notBetween("age", 18, 30)--->age not between 18 and 30
```

#### 相似、不相似、左相似、右相似(与等于的区别在于可以进行模式匹配)

```
like("name", "王")--->name like '%王%'
notLike("name", "王")--->name not like '%王%'
likeLeft("name", "王")--->name like '%王'
likeRight("name", "王")--->name like '王%'
```

#### 字段为空、字段不为空

```
isNull("name")--->name is null
isNotNull("name")--->name is not null
```

#### 属于、不属于

```
in("age", 1, 2, 3)--->age in (1,2,3)
notIn("age",{1,2,3})--->age not in (1,2,3)
```

#### 组合(将以某列属性为准，进行合计)

| O_Id | OrderDate  | OrderPrice | Customer |
| :--- | :--------- | :--------- | :------- |
| 1    | 2008/12/29 | 1000       | Bush     |
| 2    | 2008/11/23 | 1600       | Carter   |
| 3    | 2008/10/05 | 700        | Bush     |
| 4    | 2008/09/28 | 300        | Bush     |
| 5    | 2008/08/06 | 2000       | Adams    |
| 6    | 2008/07/21 | 100        | Carter   |

```
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
```

| Customer | SUM(OrderPrice) |
| :------- | :-------------- |
| Bush     | 2000            |
| Carter   | 1700            |
| Adams    | 2000            |

```
groupBy("id", "name")`--->`group by id,name
```

#### 升序排列、降序排列

```
//升序
orderByAsc("id", "name")--->order by id ASC,name ASC
//降序
orderByDesc("id", "name")--->order by id DESC,name DESC
//true升序、false降序
orderBy(true, true, "id", "name")--->order by id ASC,name ASC
```

#### HAVING

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。

```
having("sum(age) > 10")--->having sum(age) > 10
having("sum(age) > {0}", 11)`--->`having sum(age) > 11
```

#### 或逻辑

```
//拼接方式 不使用or 默认为and
eq("id",1).or().eq("name","老王")--->id = 1 or name = '老王'
//嵌套方式 可以包括进多个条件（多个条件内部默认还是and）
queryWrapper.isNotNull("email").or(i -> i.like("name", "帅").lt("age", 12));
SELECT id,name,email,age FROM student WHERE (email IS NOT NULL OR (name LIKE ? AND age < ?))
```

#### 与逻辑

(默认是与逻辑，这里主要是与逻辑嵌套用法)

```
and(i -> i.eq("name", "李白").ne("status", "活着"))--->and (name = '李白' and status <> '活着')
```

```
关于与或逻辑，sql中，AND的默认结算优先级是高于OR的，所以AND和OR的这种逻辑嵌套，都是非常有必要的。
```

#### 纯嵌套

```
nested(i -> i.eq("name", "李白").ne("status", "活着"))`--->`(name = '李白' and status <> '活着')
```

#### APPLY(采用原生sql)

```
queryWrapper.apply("date_format(create_time,'%Y-%m-%d')={0}","2020-07-06");
SELECT id,name,email,age,create_time FROM student WHERE (date_format(create_time,'%Y-%m-%d')=?) 

queryWrapper.apply("name = {0}","帅哥");
SELECT id,name,email,age,create_time FROM student WHERE (name = ?) 
```

#### last

无视优化规则，拼接到sql最后

```
last("limit 1")
```

### 查询不返回全部列的方法

可以使用select方法

```java
queryWrapper.select("id","name").like("name","帅");
queryWrapper.select(Student.class,info->!info.getColumn().equals("create_time")).like("name","帅");
```

### condition的使用

几乎所有的条件构造器前面都有condition重载函数

```java
//使用该方法，可以简化业务逻辑代码
queryWrapper.like(name != null, "name", "帅");
```

### Entity构建条件构造器

实体进行查询，仅仅只是将查询条件和数据进行绑定，但是查询的这个对象并不确定是否是一个实体。

```java
Student student = new Student();
student.setName("帅哥");
QueryWrapper<Student> queryWrapper = new QueryWrapper<>(student);
List<Student> students = studentMapper.selectList(queryWrapper);
System.out.println(students);
```

```java
针对不同实体的构造条件构造器的方法
@TableField(condition = "%s&lt;#{%s}")
private Integer age;
@TableField(condition = SqlCondition.EQUAL)
private String createTime;
```

### allEq

```java
QueryWrapper<Student> wrapper = new QueryWrapper<>();
Map<String, Object> params = new HashMap<>();
params.put("name", "帅哥");
params.put("age", 12);
//wrapper.allEq(params);
wrapper.allEq((k, v) -> !k.equals("name"), params);

List<Student> students = studentMapper.selectList(wrapper);
System.out.println(students);
```

### 返回Maps类型

在没有定义实体类的时候，或者有些场景并不需要全部属性的时候

使用Maps类型替代实体类或许是一个不错的选择

```java
QueryWrapper<Student> wrapper = new QueryWrapper<>();
wrapper.like("name", "帅").lt("age", 13);
List<Map<String, Object>> maps = studentMapper.selectMaps(wrapper);
System.out.println(maps);
```

```java
QueryWrapper<Student> wrapper = new QueryWrapper<>();
wrapper.select("avg(age) avg_age", "min(age) min_age", "max(age) max_age").groupBy("id").having("sum(age) < {0}", 500);
List<Map<String, Object>> maps = studentMapper.selectMaps(wrapper);
System.out.println(maps);
```

### 查询个数

```java
QueryWrapper<Student> wrapper = new QueryWrapper<>();
wrapper.like("name", "帅");
Integer studentsLength = studentMapper.selectCount(wrapper);
System.out.println(studentsLength);
```

### lambda构造

```java
    @Test
    void selectLambda() {
//        LambdaQueryWrapper<Student> wrapper = new QueryWrapper<Student>().lambda();
//        LambdaQueryWrapper<Student> wrapper = Wrappers.<Student>lambdaQuery();
        LambdaQueryWrapper<Student> wrapper = new LambdaQueryWrapper<>();
        wrapper.like(Student::getName,"帅").lt(Student::getAge,12);
        List<Student> students = studentMapper.selectList(wrapper);
        System.out.println(students);
    }
```

## 自定义SQL

### 基础方式

Mapper中增加带有@Select注解的方法

```java
public interface StudentMapper extends BaseMapper<Student> {
    @Select("select * from student ")
    List<Student> selectAll();
}
```

使用该方法

```java
@Test
void customSql() {
    QueryWrapper<Student> queryWrapper = new QueryWrapper<>();
    List<Student> students = studentMapper.selectAll();
    System.out.println(students);
}
```

自定义sql中，引入条件构造器

```java
public interface StudentMapper extends BaseMapper<Student> {
    @Select("select * from student ${ew.customSqlSegment}")
    List<Student> selectAll(@Param(Constants.WRAPPER) Wrapper<Student> wrapper);
}
```

```java
@Test
void customSql() {
    QueryWrapper<Student> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name", "帅");
    List<Student> students = studentMapper.selectAll(queryWrapper);
    System.out.println(students);
}
```

## 分页插件

```java
@Test
void page() {
    QueryWrapper<Student> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 12);

    Page<Student> page = new Page<>(2, 2);
    //如果第三个参数传递false，那么就iPage.getPages() 与 ipage.getTotal() 将无法使用
    //Page<Student> page = new Page<>(2, 2, false);
	//多了一个page对象
    IPage<Student> iPage = studentMapper.selectPage(page, queryWrapper);


    System.out.println("总页数" + iPage.getPages());
    System.out.println("总记录数" + iPage.getTotal());

    List<Student> students = iPage.getRecords();
    System.out.println(students);
}
```