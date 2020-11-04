# Mysql查询

## 使用

### 链接、关闭链接

**启动链接到数据库**

mysql -u root -p
输入密码即可

```
D:\Program Files (x86)\mariaDb\bin>mysql -u root -p
Enter password: *****
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.5.3-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

**关闭与数据库的链接**

quit

```
MariaDB [(none)]> quit
Bye

D:\Program Files (x86)\mariaDb\bin>
```

### 查看数据库版本

```
MariaDB [(none)]> SELECT VERSION(), CURRENT_DATE;
+----------------+--------------+
| VERSION()      | CURRENT_DATE |
+----------------+--------------+
| 10.5.3-MariaDB | 2020-10-26   |
+----------------+--------------+
1 row in set (0.009 sec)
```

## 特点

### 大小写不敏感

```
mysql> SELECT VERSION(), CURRENT_DATE;
mysql> select version(), current_date;
mysql> SeLeCt vErSiOn(), current_DATE;
```

### 具有函数、表达式运算

```
MariaDB [(none)]> SELECT SIN(PI()/4), (4+1)*5;
+--------------------+---------+
| SIN(PI()/4)        | (4+1)*5 |
+--------------------+---------+
| 0.7071067811865476 |      25 |
+--------------------+---------+
1 row in set (0.001 sec)
```

### 分号是语句的结束符

**一行可以包括多个语句**

使用分号隔开多个语句，相当于一次性执行多条语句。

```
MariaDB [(none)]> SELECT VERSION(); SELECT NOW();
+----------------+
| VERSION()      |
+----------------+
| 10.5.3-MariaDB |
+----------------+
1 row in set (0.000 sec)

+---------------------+
| NOW()               |
+---------------------+
| 2020-10-26 19:05:27 |
+---------------------+
1 row in set (0.000 sec)
```

**多行执行一条语句**

如果不键入分号，那么就会一直等待分号的出现。

```
MariaDB [(none)]> SELECT
    -> USER()
    -> ;
+----------------+
| USER()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.000 sec)
```

**不同的promt**

| Prompt   | Meaning                                                      |
| :------- | :----------------------------------------------------------- |
| `mysql>` | Ready for new query                                          |
| `->`     | Waiting for next line of multiple-line query                 |
| `'>`     | Waiting for next line, waiting for completion of a string that began with a single quote (`'`) |
| `">`     | Waiting for next line, waiting for completion of a string that began with a double quote (`"`) |
| ``>`     | Waiting for next line, waiting for completion of an identifier that began with a backtick (```) |
| `/*>`    | Waiting for next line, waiting for completion of a comment that began with `/*` |

我们会发现当键入的括号不匹配的时候，mysql输入框的提示符会发生变化

```
MariaDB [mblearn]> SELECT * FROM student WHERE name = "帅哥";
+----+------+--------------------+------+---------------------+
| id | name | email              | age  | create_time         |
+----+------+--------------------+------+---------------------+
|  6 | 帅哥 | laofanshuai@qq.com |   22 | 2020-07-06 09:21:59 |
|  7 | 帅哥 | shauige@qq.com     |   11 | 2020-07-06 09:22:01 |
+----+------+--------------------+------+---------------------+
2 rows in set (0.000 sec)

MariaDB [mblearn]> SELECT * FROM student WHERE name = "帅
    "> 哥";
Empty set (0.000 sec)

MariaDB [mblearn]> SELECT * FROM student WHERE name = "帅
    "> "
    -> ;
Empty set (0.000 sec)

MariaDB [mblearn]> SELECT * FROM student WHERE name = "帅
    "> ;
    "> "
    -> ;
Empty set (0.001 sec)
```

似乎这个提示符并不算太靠谱，毕竟在帅哥分开查询的时候，感觉没有达到我想要的效果。

## 数据库操作

### 查看数据库

SHOW DATABASES;

```
MariaDB [mblearn]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| counterattack      |
| information_schema |
| mblearn            |
| mysql              |
| performance_schema |
| test               |
| vueblog            |
+--------------------+
7 rows in set (0.001 sec)
```

### 使用数据库

USE mblearn;

```
MariaDB [mblearn]> USE mblearn;
Database changed
```

## 表操作

### 查看所有表

SHOW TABLES;

```
MariaDB [(none)]> SHOW TABLES;
ERROR 1046 (3D000): No database selected
MariaDB [(none)]> USE mbLEARN;
Database changed
MariaDB [mbLEARN]> SHOW TABLES;
+-------------------+
| Tables_in_mblearn |
+-------------------+
| student           |
| student_tag       |
| tag               |
+-------------------+
3 rows in set (0.001 sec)
```

### 查看单个表的字段

DESCRIBE student;

```
MariaDB [mbLEARN]> DESCRIBE student;
+-------------+--------------+------+-----+---------+----------------+
| Field       | Type         | Null | Key | Default | Extra          |
+-------------+--------------+------+-----+---------+----------------+
| id          | int(11)      | NO   | PRI | NULL    | auto_increment |
| name        | varchar(255) | YES  |     |         |                |
| email       | varchar(255) | YES  |     |         |                |
| age         | int(11)      | YES  |     | NULL    |                |
| create_time | datetime     | NO   |     | NULL    |                |
+-------------+--------------+------+-----+---------+----------------+
5 rows in set (0.011 sec)
```

### 通过文件添加表的数据

文件的内容

```
23	光华	123112@qq.com	99	1996-04-29
```

通过文件插入表的语句

普通的使用这种方式就可以了

 LOAD DATA LOCAL INFILE "C:\\Users\\scffz\\Desktop\\a.txt" INTO TABLE student;

如果在 Windows 上创建了使用用作行终止符的编辑器的文件，则应改为使用此语句：

 LOAD DATA LOCAL INFILE "C:\\Users\\\scffz\\Desktop\\a.txt" INTO TABLE student LINES TERMINATED BY '\r\n';

```
MariaDB [mbLEARN]> LOAD DATA LOCAL INFILE "C:\Users\scffz\Desktop\a.txt" INTO TABLE student
    -> LINES TERMINATED BY '\r\n';
ERROR 2 (HY000): File 'C:UsersscffzDesktopa.txt' not found (Errcode: 2)
MariaDB [mbLEARN]> LOAD DATA LOCAL INFILE "C:\\Users\\scffz\\Desktop\\a.txt" INTO TABLE student
    -> LINES TERMINATED BY '\r\n';
Query OK, 1 row affected (0.002 sec)
Records: 1  Deleted: 0  Skipped: 0  Warnings: 0
```

需要注意的是，在数据库中的string文本也是需要**识别转义字符**的

### INSERT

这里ID是自增的，所以在添加的时候，这里给的数值可以直接给null就行了。

```
MariaDB [mblearn]> INSERT INTO student VALUES(null,"来福","123122@qq.com",12,"2020-10-26 20:13:40");
Query OK, 1 row affected (0.002 sec)
```

### SELECT

#### SELECT的结构

SELECT *what_to_select* 

FROM *which_table* 

WHERE *conditions_to_satisfy*;

```
MariaDB [mblearn]> SELECT name,email FROM student WHERE age=11;
+------+----------------+
| name | email          |
+------+----------------+
| 帅哥 | shauige@qq.com |
| 帅   | shauidi@qq.com |
+------+----------------+
2 rows in set (0.000 sec)
```

#### 运算的优先级

在conditions这里，需要注意的是，AND的运算优先级更高。

```
MariaDB [mblearn]> SELECT * FROM student;
+----+------------+--------------------+------+---------------------+
| id | name       | email              | age  | create_time         |
+----+------------+--------------------+------+---------------------+
|  1 | fanzcbe    | 52125@af.com       |   12 | 2020-07-06 09:21:45 |
|  2 | 范帅       | 4162123123@qq.com  |   24 | 2020-07-06 09:21:48 |
|  6 | 帅哥       | laofanshuai@qq.com |   22 | 2020-07-06 09:21:59 |
|  7 | 帅哥       | shauige@qq.com     |   11 | 2020-07-06 09:22:01 |
|  8 | 帅         | shauidi@qq.com     |   11 | 2020-07-06 09:22:04 |
|  9 | lubada19.0 | 2094848.0@qq.com   |   16 | 2020-07-08 15:41:40 |
| 10 | lubada64.0 | 2061879.0@qq.com   |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com   |   19 | 2020-09-26 14:50:56 |
| 23 | ?…‰???   | 123112@qq.com      |   99 | 1996-04-29 00:00:00 |
| 24 | 来福       | 123122@qq.com      |   12 | 2020-10-26 20:13:40 |
+----+------------+--------------------+------+---------------------+
10 rows in set (0.000 sec)

MariaDB [mblearn]> SELECT * FROM student WHERE (id < 20 OR age > 10) AND name = "帅哥";
+----+------+--------------------+------+---------------------+
| id | name | email              | age  | create_time         |
+----+------+--------------------+------+---------------------+
|  6 | 帅哥 | laofanshuai@qq.com |   22 | 2020-07-06 09:21:59 |
|  7 | 帅哥 | shauige@qq.com     |   11 | 2020-07-06 09:22:01 |
+----+------+--------------------+------+---------------------+
2 rows in set (0.000 sec)
```

#### DISTINCT-查值范围

查出表中的独立属性值

```
MariaDB [mblearn]> SELECT name FROM student;
+------------+
| name       |
+------------+
| fanzcbe    |
| 范帅       |
| 帅哥       |
| 帅哥       |
| 帅         |
| lubada19.0 |
| lubada64.0 |
| lubada18.0 |
| ?…‰???   |
| 来福       |
+------------+
10 rows in set (0.000 sec)

MariaDB [mblearn]> SELECT DISTINCT name FROM student;
+------------+
| name       |
+------------+
| fanzcbe    |
| 范帅       |
| 帅哥       |
| 帅         |
| lubada19.0 |
| lubada64.0 |
| lubada18.0 |
| ?…‰???   |
| 来福       |
+------------+
9 rows in set (0.008 sec)
```

如果DISTINCT有两个值的时候，它是将两个值看在一起做独立。

```
MariaDB [mblearn]> SELECT DISTINCT email, name FROM student;
+--------------------+------------+
| email              | name       |
+--------------------+------------+
| 52125@af.com       | fanzcbe    |
| 4162123123@qq.com  | 范帅       |
| laofanshuai@qq.com | 帅哥       |
| shauige@qq.com     | 帅哥       |
| shauidi@qq.com     | 帅         |
| 2094848.0@qq.com   | lubada19.0 |
| 2061879.0@qq.com   | lubada64.0 |
| 1339782.0@qq.com   | lubada18.0 |
| 123112@qq.com      | ?…‰???   |
| 123122@qq.com      | 来福       |
+--------------------+------------+
10 rows in set (0.000 sec)
```

#### ORDER-查询顺序

默认都是升序

SELECT * FROM student ORDER BY age;

SELECT * FROM student ORDER BY age ASC;

```
MariaDB [mblearn]> SELECT * FROM student ORDER BY age;
+----+------------+--------------------+------+---------------------+
| id | name       | email              | age  | create_time         |
+----+------------+--------------------+------+---------------------+
|  7 | 帅哥       | shauige@qq.com     |   11 | 2020-07-06 09:22:01 |
|  8 | 帅         | shauidi@qq.com     |   11 | 2020-07-06 09:22:04 |
|  1 | fanzcbe    | 52125@af.com       |   12 | 2020-07-06 09:21:45 |
| 24 | 来福       | 123122@qq.com      |   12 | 2020-10-26 20:13:40 |
|  9 | lubada19.0 | 2094848.0@qq.com   |   16 | 2020-07-08 15:41:40 |
| 10 | lubada64.0 | 2061879.0@qq.com   |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com   |   19 | 2020-09-26 14:50:56 |
|  6 | 帅哥       | laofanshuai@qq.com |   22 | 2020-07-06 09:21:59 |
|  2 | 范帅       | 4162123123@qq.com  |   24 | 2020-07-06 09:21:48 |
| 23 | ?…‰???   | 123112@qq.com      |   99 | 1996-04-29 00:00:00 |
+----+------------+--------------------+------+---------------------+
10 rows in set (0.000 sec)
```

降序查询

SELECT * FROM student ORDER BY age DESC;

```
MariaDB [mblearn]> SELECT * FROM student ORDER BY age DESC;
+----+------------+--------------------+------+---------------------+
| id | name       | email              | age  | create_time         |
+----+------------+--------------------+------+---------------------+
| 23 | ?…‰???   | 123112@qq.com      |   99 | 1996-04-29 00:00:00 |
|  2 | 范帅       | 4162123123@qq.com  |   24 | 2020-07-06 09:21:48 |
|  6 | 帅哥       | laofanshuai@qq.com |   22 | 2020-07-06 09:21:59 |
| 11 | lubada18.0 | 1339782.0@qq.com   |   19 | 2020-09-26 14:50:56 |
| 10 | lubada64.0 | 2061879.0@qq.com   |   18 | 2020-09-23 14:28:43 |
|  9 | lubada19.0 | 2094848.0@qq.com   |   16 | 2020-07-08 15:41:40 |
|  1 | fanzcbe    | 52125@af.com       |   12 | 2020-07-06 09:21:45 |
| 24 | 来福       | 123122@qq.com      |   12 | 2020-10-26 20:13:40 |
|  7 | 帅哥       | shauige@qq.com     |   11 | 2020-07-06 09:22:01 |
|  8 | 帅         | shauidi@qq.com     |   11 | 2020-07-06 09:22:04 |
+----+------------+--------------------+------+---------------------+
```

#### LIKE-格式匹配

**"%"符号的使用**：指代若干字符

```
MariaDB [mblearn]> SELECT * FROM student WHERE name LIKE 'l%';
+----+------------+------------------+------+---------------------+
| id | name       | email            | age  | create_time         |
+----+------------+------------------+------+---------------------+
|  9 | lubada19.0 | 2094848.0@qq.com |   16 | 2020-07-08 15:41:40 |
| 10 | lubada64.0 | 2061879.0@qq.com |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com |   19 | 2020-09-26 14:50:56 |
+----+------------+------------------+------+---------------------+
3 rows in set (0.001 sec)

MariaDB [mblearn]> SELECT * FROM student WHERE name LIKE '%l';
Empty set (0.009 sec)

MariaDB [mblearn]> SELECT * FROM student WHERE name LIKE '%l%';
+----+------------+------------------+------+---------------------+
| id | name       | email            | age  | create_time         |
+----+------------+------------------+------+---------------------+
|  9 | lubada19.0 | 2094848.0@qq.com |   16 | 2020-07-08 15:41:40 |
| 10 | lubada64.0 | 2061879.0@qq.com |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com |   19 | 2020-09-26 14:50:56 |
+----+------------+------------------+------+---------------------+
3 rows in set (0.000 sec)
```

**"_"的使用**：指代单个字符

```
MariaDB [mblearn]> SELECT * FROM student WHERE name LIKE '%l_________';
+----+------------+------------------+------+---------------------+
| id | name       | email            | age  | create_time         |
+----+------------+------------------+------+---------------------+
|  9 | lubada19.0 | 2094848.0@qq.com |   16 | 2020-07-08 15:41:40 |
| 10 | lubada64.0 | 2061879.0@qq.com |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com |   19 | 2020-09-26 14:50:56 |
+----+------------+------------------+------+---------------------+
3 rows in set (0.000 sec)
```

使用正则表达式(MariaDB)

```
MariaDB [mblearn]> SELECT * from student WHERE name REGEXP '^l';
+----+------------+------------------+------+---------------------+
| id | name       | email            | age  | create_time         |
+----+------------+------------------+------+---------------------+
|  9 | lubada19.0 | 2094848.0@qq.com |   16 | 2020-07-08 15:41:40 |
| 10 | lubada64.0 | 2061879.0@qq.com |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com |   19 | 2020-09-26 14:50:56 |
+----+------------+------------------+------+---------------------+
3 rows in set (0.000 sec)
```

### 日期计算

#### 计算时间差

**TIMESTAMPDIFF(YEAR,a,b)**

```
MariaDB [mblearn]> SELECT name,email,TIMESTAMPDIFF(MONTH,create_time,CURDATE()) AS age_month from student;
+------------+--------------------+-----------+
| name       | email              | age_month |
+------------+--------------------+-----------+
| fanzcbe    | 52125@af.com       |         3 |
| 范帅       | 4162123123@qq.com  |         3 |
| 帅哥       | laofanshuai@qq.com |         3 |
| 帅哥       | shauige@qq.com     |         3 |
| 帅         | shauidi@qq.com     |         3 |
| lubada19.0 | 2094848.0@qq.com   |         3 |
| lubada64.0 | 2061879.0@qq.com   |         1 |
| lubada18.0 | 1339782.0@qq.com   |         1 |
| ?…‰???   | 123112@qq.com      |       294 |
| 来福       | 123122@qq.com      |         0 |
+------------+--------------------+-----------+
10 rows in set (0.000 sec)
```

#### 取日期的某一位

```
MariaDB [mblearn]> SELECT create_time FROM student;
+---------------------+
| create_time         |
+---------------------+
| 2020-07-06 09:21:45 |
| 2020-07-06 09:21:48 |
| 2020-07-06 09:21:59 |
| 2020-07-06 09:22:01 |
| 2020-07-06 09:22:04 |
| 2020-07-08 15:41:40 |
| 2020-09-23 14:28:43 |
| 2020-09-26 14:50:56 |
| 1996-04-29 00:00:00 |
| 2020-10-26 20:13:40 |
+---------------------+
10 rows in set (0.000 sec)

MariaDB [mblearn]> SELECT MONTH(create_time) FROM student;
+--------------------+
| MONTH(create_time) |
+--------------------+
|                  7 |
|                  7 |
|                  7 |
|                  7 |
|                  7 |
|                  7 |
|                  9 |
|                  9 |
|                  4 |
|                 10 |
+--------------------+
10 rows in set (0.000 sec)
```

---

```
YEAR,MONTH,DAY,HOUR,MINUTE,SECOND 都可以使用
```

日期计算的表达式也可以放进查询子句中

```
MariaDB [mblearn]> SELECT * FROM student WHERE MONTH(create_time) = 9;
+----+------------+------------------+------+---------------------+
| id | name       | email            | age  | create_time         |
+----+------------+------------------+------+---------------------+
| 10 | lubada64.0 | 2061879.0@qq.com |   18 | 2020-09-23 14:28:43 |
| 11 | lubada18.0 | 1339782.0@qq.com |   19 | 2020-09-26 14:50:56 |
+----+------------+------------------+------+---------------------+
2 rows in set (0.000 sec)
```

#### 日期的运算

查询三个月之前的创建的字段

```
MariaDB [mblearn]> SELECT name,email FROM student WHERE MONTH(create_time)=MONTH(DATE_SUB(CURDATE(),INTERVAL 3 MONTH));
+------------+--------------------+
| name       | email              |
+------------+--------------------+
| fanzcbe    | 52125@af.com       |
| 范帅       | 4162123123@qq.com  |
| 帅哥       | laofanshuai@qq.com |
| 帅哥       | shauige@qq.com     |
| 帅         | shauidi@qq.com     |
| lubada19.0 | 2094848.0@qq.com   |
+------------+--------------------+
6 rows in set (0.000 sec)
```

非法日期会返回null

```
MariaDB [mblearn]> SELECT '2018-10-31' + INTERVAL 1 DAY;
+-------------------------------+
| '2018-10-31' + INTERVAL 1 DAY |
+-------------------------------+
| 2018-11-01                    |
+-------------------------------+
1 row in set (0.001 sec)

MariaDB [mblearn]> SELECT '2018-10-32' + INTERVAL 1 DAY;
+-------------------------------+
| '2018-10-32' + INTERVAL 1 DAY |
+-------------------------------+
| NULL                          |
+-------------------------------+
1 row in set, 1 warning (0.000 sec)
```

### 空值对象

NULL是缺失的未知值，可以使用IS 和 IS NOT 来判定一个值是否为NULL。

```
MariaDB [mblearn]> SELECT 1 IS NULL,1 IS NOT NULL;
+-----------+---------------+
| 1 IS NULL | 1 IS NOT NULL |
+-----------+---------------+
|         0 |             1 |
+-----------+---------------+
1 row in set (0.000 sec)
```

```
MariaDB [mblearn]> SELECT NULL = NULL;
+-------------+
| NULL = NULL |
+-------------+
|        NULL |
+-------------+
1 row in set (0.000 sec)

MariaDB [mblearn]> SELECT NULL IS NULL;
+--------------+
| NULL IS NULL |
+--------------+
|            1 |
+--------------+
1 row in set (0.000 sec)
```

0与''空串，都有近似于NULL的意思。但在MYSQL中，他们都是不同的。

```
MariaDB [mblearn]> SELECT 0 IS NULL, 0 IS NOT NULL, '' IS NULL, '' IS NOT NULL;
+-----------+---------------+------------+----------------+
| 0 IS NULL | 0 IS NOT NULL | '' IS NULL | '' IS NOT NULL |
+-----------+---------------+------------+----------------+
|         0 |             1 |          0 |              1 |
+-----------+---------------+------------+----------------+
1 row in set (0.000 sec)
```

### 统计数量

查询学生的数量

```
MariaDB [mblearn]> SELECT COUNT(*) FROM student;
+----------+
| COUNT(*) |
+----------+
|       10 |
+----------+
1 row in set (0.000 sec)

MariaDB [mblearn]> SELECT name,COUNT(*) FROM student;
+---------+----------+
| name    | COUNT(*) |
+---------+----------+
| fanzcbe |       10 |
+---------+----------+
1 row in set (0.000 sec)

MariaDB [mblearn]> SELECT name,COUNT(*) FROM student GROUP BY name;
+------------+----------+
| name       | COUNT(*) |
+------------+----------+
| fanzcbe    |        1 |
| lubada18.0 |        1 |
| lubada19.0 |        1 |
| lubada64.0 |        1 |
| 哇哦       |        1 |
| 帅         |        1 |
| 帅哥       |        2 |
| 来福       |        1 |
| 范帅       |        1 |
+------------+----------+
9 rows in set (0.000 sec)

MariaDB [mblearn]> SELECT name,email,COUNT(*) FROM student GROUP BY name,email;
+------------+--------------------+----------+
| name       | email              | COUNT(*) |
+------------+--------------------+----------+
| fanzcbe    | 52125@af.com       |        1 |
| lubada18.0 | 1339782.0@qq.com   |        1 |
| lubada19.0 | 2094848.0@qq.com   |        1 |
| lubada64.0 | 2061879.0@qq.com   |        1 |
| 哇哦       | 123112@qq.com      |        1 |
| 帅         | shauidi@qq.com     |        1 |
| 帅哥       | laofanshuai@qq.com |        1 |
| 帅哥       | shauige@qq.com     |        1 |
| 来福       | 123122@qq.com      |        1 |
| 范帅       | 4162123123@qq.com  |        1 |
+------------+--------------------+----------+
10 rows in set (0.000 sec)

```



