# yaml

## 基本语法

1· k:"空格"v 表示键值对
2· 空格的缩进来控制层级关系
3· 属性与值都是大小写敏感

## 值的写法

### 字面量

#### 数字，布尔

#### 字符串

实际上字符串也可以不加引号
单引号字符串，不会转义，特殊字符的书写会成为两个普通的字符
双引号字符串，会转义，特殊字符就是所代表的字符

```yml
name: "zhangsan \n lisi"
```

```yml
name: 'zhangsan \n lisi'
```

### 对象

```yml
friend:
    lastName: zhangsan
    age: 20
```

```yml
friend: {lastName: zhangsan,age: 18}
```

### 数组

```yml
pets:
 - cat
 - dog
 - pig
```

```yml
pets: [cat,dog,pig]
```
