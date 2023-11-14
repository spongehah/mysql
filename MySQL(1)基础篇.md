[TOC]



 基础篇



# MySQL基础篇



**通用语法：**

在学习具体的SQL语句之前，先来了解一下SQL语言的通用语法。

- 1). SQL语句可以单行或多行书写，以分号结尾。

- 2). SQL语句可以使用空格/缩进来增强语句的可读性。

- 3). MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。

- 4). 注释：

  - 单行注释：-- 注释内容 或 # 注释内容

  - 多行注释：/* 注释内容 */



**分类：**

- DDL（Data DefinitionLanguage）: 数据定义语言，用来定义数据库对象（数据库、表、字段）
- DML（Data ManipulationLanguage）: 数据操作语言，用来对数据库表中的数据进行增删改
- DQL（Data Query Language ）: 数据查询语言，用来查询数据库中表的记录
- DCL（Data Control Language）: 数据控制语言，用来创建数据库用户、控制数据库的控制权限



# DDL（数据定义语言）



## 数据库操作

查询所有数据库：

```mysql
SHOW DATABASES;
```

查询当前数据库：

```mysql
SELECT DATABASE();
```

创建数据库：

```mysql
CREATE DATABASE [ IF NOT EXISTS ] 数据库名 [ DEFAULT CHARSET 字符集] [COLLATE 排序规则 ];
```

删除数据库：

```mysql
DROP DATABASE [ IF EXISTS ] 数据库名;
```

使用数据库：

```mysql
USE 数据库名;
```

**注意事项**

- UTF8字符集长度为3字节，有些符号占4字节，所以推荐用utf8mb4字符集



## 表操作



查询当前数据库所有表：

```
SHOW TABLES;
```

查询表结构：

```
`DESC 表名;`
```

查询指定表的建表语句：

```
SHOW CREATE TABLE 表名;
```

创建表：

```mysql
CREATE TABLE 表名(
	字段1 字段1类型 [COMMENT 字段1注释],
	字段2 字段2类型 [COMMENT 字段2注释],
	字段3 字段3类型 [COMMENT 字段3注释],
	...
	字段n 字段n类型 [COMMENT 字段n注释]
)[ COMMENT 表注释 ];
```

**最后一个字段后面没有逗号**



**修改表结构：**

添加字段：

```
`ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束];`
```

```
例：`ALTER TABLE emp ADD nickname varchar(20) COMMENT '昵称';`
```

修改数据类型：

```
`ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);`
```

修改字段名和字段类型：

```
`ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度) [COMMENT 注释] [约束];`
```

例：将emp表的nickname字段修改为username，类型为varchar(30)

```
ALTER TABLE emp CHANGE nickname username varchar(30) COMMENT '昵称';
```

删除字段：

```
ALTER TABLE 表名 DROP 字段名;
```

修改表名：

```
ALTER TABLE 表名 RENAME TO 新表名
```

删除表：

```
DROP TABLE [IF EXISTS] 表名;
```

删除表，并重新创建该表：

```
TRUNCATE TABLE 表名;
```



# DQL（数据查询语言）

- 简单查询
- 自连接
- 内连接
- 外连接
- 子查询





# DML（数据操作语言）

## 添加数据

指定字段：

```
INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...);
```

全部字段：

```
INSERT INTO 表名 VALUES (值1, 值2, ...);
```

批量添加数据：

```
`INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);`
`INSERT INTO 表名 VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);`
```

**注意事项**

- 字符串和日期类型数据应该包含在引号中
- 插入的数据大小应该在字段的规定范围内



## 更新和删除数据

修改数据：

```
UPDATE 表名 SET 字段名1 = 值1, 字段名2 = 值2, ... [ WHERE 条件 ];
```

例：

```
UPDATE emp SET name = 'Jack' WHERE id = 1;
```

删除数据：

```
DELETE FROM 表名 [ WHERE 条件 ];
```



> 删除表，并重新创建该表：（类似于删除数据）
> `TRUNCATE TABLE 表名;`





# DCL（数据控制语言）

**登录MySQL服务器**

启动MySQL服务后，可以通过mysql命令来登录MySQL服务器，命令如下：

```mysql
mysql –h hostname|hostIP –P port –u username –p DatabaseName –e "SQL语句"
```

- `DatabaseName参数`指明登录到哪一个数据库中。如果没有该参数，就会直接登录到MySQL数据库中，然后可以使用USE命令来选择数据库。
- `-e参数`后面可以直接加SQL语句。登录MySQL服务器以后即可执行这个SQL语句，然后退出MySQL服务器。



## 增删改用户

**创建用户**

```mysql
CREATE USER 用户名 [IDENTIFIED BY '密码'][,用户名 [IDENTIFIED BY '密码']];
```

举例：

```mysql
CREATE USER zhang3 IDENTIFIED BY '123123'; # 默认host是 %
CREATE USER 'kangshifu'@'localhost' IDENTIFIED BY '123456';
```

**修改用户**

```mysql
UPDATE mysql.user SET USER='li4' WHERE USER='wang5'; 
FLUSH PRIVILEGES;
```

**删除用户**

**方式1：使用DROP方式删除（推荐）**

```mysql
DROP USER user[,user]…;
```

举例：

```mysql
DROP USER li4 ; # 默认删除host为%的用户
DROP USER 'kangshifu'@'localhost';
```

**方式2：使用DELETE方式删除（不推荐，有残留信息）**

```mysql
DELETE FROM mysql.user WHERE Host=’hostname’ AND User=’username’;
FLUSH PRIVILEGES;
```

>注意：不推荐通过 DELETE FROM USER u WHERE USER='li4' 进行删除，系统会有残留信息保留。而**drop user命令会删除用户以及对应的权限，执行命令后你会发现mysql.user表和mysql.db表的相应记录都消失了。**

 

**设置当前用户密码**

**1.** **使用ALTER USER命令来修改当前用户密码**

```mysql
ALTER USER USER() IDENTIFIED BY 'new_password';
```

**2.** **使用SET语句来修改当前用户密码**

```mysql
SET PASSWORD='new_password';
```

**1.6** **修改其它用户密码** 

**1.** **使用ALTER语句来修改普通用户的密码**

```mysql
ALTER USER user [IDENTIFIED BY '新密码'] 
[,user[IDENTIFIED BY '新密码']]…;
```

**2.** **使用SET命令来修改普通用户的密码**

```mysql
SET PASSWORD FOR 'username'@'hostname'='new_password';
```







## 权限管理

**2.1权限列表**

```mysql
show privileges;
```

- `CREATE和DROP权限`，可以创建新的数据库和表，或删除（移掉）已有的数据库和表。如果将MySQL数据库中的DROP权限授予某用户，用户就可以删除MySQL访问权限保存的数据库。
- `SELECT、INSERT、UPDATE和DELETE权限`允许在一个数据库现有的表上实施操作。
- `SELECT权限`只有在它们真正从一个表中检索行时才被用到。
- `INDEX权限`允许创建或删除索引，INDEX适用于已有的表。如果具有某个表的CREATE权限，就可以在CREATE TABLE语句中包括索引定义。
- `ALTER权限`可以使用ALTER TABLE来更改表的结构和重新命名表。
- `CREATE ROUTINE权限`用来创建保存的程序（函数和程序），`ALTER ROUTINE权限`用来更改和删除保存的程序，`EXECUTE权限`用来执行保存的程序。
- `GRANT权限`允许授权给其他用户，可用于数据库、表和保存的程序。
- `FILE权限`使用户可以使用LOAD DATA INFILE和SELECT ... INTO OUTFILE语句读或写服务器上的文件，任何被授予FILE权限的用户都能读或写MySQL服务器上的任何文件（说明用户可以读任何数据库目录下的文件，因为服务器可以访问这些文件）。

**2.2** **授予权限的原则**

权限控制主要是出于安全因素，因此需要遵循以下几个`经验原则`：

1、只授予能`满足需要的最小权限`，防止用户干坏事。比如用户只是需要查询，那就只给select权限就可以了，不要给用户赋予update、insert或者delete权限。

2、创建用户的时候`限制用户的登录主机`，一般是限制成指定IP或者内网IP段。

3、为每个用户`设置满足密码复杂度的密码`。 

4、`定期清理不需要的用户`，回收权限或者删除用户。

**2.3** **授予权限**

```mysql
GRANT 权限1,权限2,…权限n ON 数据库名称.表名称 TO 用户名@用户地址 [IDENTIFIED BY ‘密码口令’];
```

- 该权限如果发现没有该用户，则会直接新建一个用户。
- 给li4用户用本地命令行方式，授予atguigudb这个库下的所有表的插删改查的权限。

```mysql
GRANT SELECT,INSERT,DELETE,UPDATE ON atguigudb.* TO li4@localhost;
```

- 授予通过网络方式登录的joe用户 ，对所有库所有表的全部权限，密码设为123。注意这里唯独不包括grant的权限

```mysql
GRANT ALL PRIVILEGES ON *.* TO joe@'%' IDENTIFIED BY '123';
```

**2.4** **查看权限**

- 查看当前用户权限

```mysql
SHOW GRANTS; 
 或 
SHOW GRANTS FOR CURRENT_USER; 
 或 
SHOW GRANTS FOR CURRENT_USER();
```

- 查看某用户的全局权限

```mysql
SHOW GRANTS FOR 'user'@'主机地址';
```

**2.5** **收回权限**

**注意：在将用户账户从user表删除之前，应该收回相应用户的所有权限。**

- 收回权限命令

```mysql
REVOKE 权限1,权限2,…权限n ON 数据库名称.表名称 FROM 用户名@用户地址;
```

- 举例

```mysql
收回全库全表的所有权限 
REVOKE ALL PRIVILEGES ON *.* FROM joe@'%'; 
收回mysql库下的所有表的插删改查权限 
REVOKE SELECT,INSERT,UPDATE,DELETE ON mysql.* FROM joe@localhost;
```

- 注意：`须用户重新登录后才能生效` 



## 角色管理

**3.1** **创建角色**

```mysql
CREATE ROLE 'role_name'[@'host_name'] [,'role_name'[@'host_name']]...
```

角色名称的命名规则和用户名类似。如果`host_name省略，默认为%`，`role_name不可省略`，不可为空。

**3.2** **给角色赋予权限**

```mysql
GRANT privileges ON table_name TO 'role_name'[@'host_name'];
```

上述语句中privileges代表权限的名称，多个权限以逗号隔开。可使用SHOW语句查询权限名称

```mysql
SHOW PRIVILEGES\G
```

**3.3** **查看角色的权限**

```mysql
SHOW GRANTS FOR 'role_name';
```

只要你创建了一个角色，系统就会自动给你一个“`USAGE`”权限，意思是`连接登录数据库的权限`。

**3.4** **回收角色的权限**

```mysql
REVOKE privileges ON tablename FROM 'rolename';
```

**3.5** **删除角色**

```mysql
DROP ROLE role [,role2]...
```

注意，`如果你删除了角色，那么用户也就失去了通过这个角色所获得的所有权限`。

**3.6** **给用户赋予角色**

角色创建并授权后，要赋给用户并处于`激活状态`才能发挥作用。

```mysql
GRANT role [,role2,...] TO user [,user2,...];
```

查询当前已激活的角色

```mysql
SELECT CURRENT_ROLE();
```

**3.7** **激活角色**

**方式1：使用set default role 命令激活角色**

```mysql
SET DEFAULT ROLE ALL TO 'kangshifu'@'localhost';
```

**方式2：将activate_all_roles_on_login设置为ON**

```mysql
SET GLOBAL activate_all_roles_on_login=ON;
```

这条 SQL 语句的意思是，对`所有角色永久激活`。

**3.8** **撤销用户的角色**

```mysql
REVOKE role FROM user;
```

**3.9** **设置强制角色(mandatory role)**

方式1：服务启动前设置

```ini
[mysqld] 
mandatory_roles='role1,role2@localhost,r3@%.atguigu.com'
```

方式2：运行时设置

```mysql
SET PERSIST mandatory_roles = 'role1,role2@localhost,r3@%.example.com'; #系统重启后仍然有效
SET GLOBAL mandatory_roles = 'role1,role2@localhost,r3@%.example.com'; #系统重启后失效
```





# 其它非表数据库对象



## 视图VIEW



### 视图的理解

视图的理解

- 视图，可以看做是一个虚拟似表，本身是不存储数据的。
  - 视图的本质，就可以看做是存储起来的SE工ECT语句
- 视图中SELECT语句中涉及到的表，称为基表
- 针对视图做DML操作，会影响到对应的基表中的数据。反之亦然。
- 视图本身的删除，不会导致基表中数据的删除。
- 视图的应用场景：针对于小型项目，不推荐使用视图。针对于大型项目，可以考虑使用视图。
- 视图的优点：简化查询；控制数据的访问权限。



### 视图的优缺点

**优点**

**1.** **操作简单**

将经常使用的查询操作定义为视图，可以使开发人员不需要关心视图对应的数据表的结构、表与表之间的关联关系，也不需要关心数据表之间的业务逻辑和查询条件，而只需要简单地操作视图即可，极大简化了开发人员对数据库的操作。

**2.** **减少数据冗余**

视图跟实际数据表不一样，它存储的是查询语句。所以，在使用的时候，我们要通过定义视图的查询语句来获取结果集。而视图本身不存储数据，不占用数据存储的资源，减少了数据冗余。

**3.** **数据安全**

MySQL将用户对数据的 访问限制 在某些数据的结果集上，而这些数据的结果集可以使用视图来实现。用户不必直接查询或操作数据表。这也可以理解为视图具有 隔离性 。视图相当于在用户和实际的数据表之间加了一层虚拟表。同时，MySQL可以根据权限将用户对数据的访问限制在某些视图上，**用户不需要查询数据表，可以直接通过视图获取数据表中的信息**。这在一定程度上保障了数据表中数据的安全性。

**4.** **适应灵活多变的需求** 

当业务系统的需求发生变化后，如果需要改动数据表的结构，则工作量相对较大，可以使用视图来减少改动的工作量。这种方式在实际工作中使用得比较多。

**5.** **能够分解复杂的查询逻辑** 

数据库中如果存在复杂的查询逻辑，则可以将问题进行分解，创建多个视图获取数据，再将创建的多个视图结合起来，完成复杂的查询逻辑。



**缺点**

如果我们在实际数据表的基础上创建了视图，那么，**如果实际数据表的结构变更了，我们就需要及时对相关的视图进行相应的维护**。特别是嵌套的视图（就是在视图的基础上创建视图），维护会变得比较复杂， 可读性不好 ，容易变成系统的潜在隐患。因为创建视图的 SQL 查询可能会对字段重命名，也可能包含复杂的逻辑，这些都会增加维护的成本。

实际项目中，如果视图过多，会导致数据库维护成本的问题。

所以，在创建视图的时候，你要结合实际项目需求，综合考虑视图的优点和不足，这样才能正确使用视图，使系统整体达到最优。



### 创建视图



- **在** **CREATE VIEW** **语句中嵌入子查询**

```mysql
CREATE [OR REPLACE] 
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] 
VIEW 视图名称 [(字段列表)]
AS 查询语句
[WITH [CASCADED|LOCAL] CHECK OPTION]
```

- 精简版

```mysql
CREATE VIEW 视图名称 
AS 查询语句
```



- **创建单表视图**

举例1：

```mysql
CREATE VIEW empvu80
AS 
SELECT  employee_id, last_name, salary
FROM    employees
WHERE   department_id = 80;
```

查询视图：

```
SELECT *
FROM	salvu80;
```

举例2：

```mysql
CREATE VIEW emp_year_salary (ename,year_salary)
AS 
SELECT ename,salary*12*(1+IFNULL(commission_pct,0))
FROM t_employee;
```

> 说明1：实际上就是我们在 SQL 查询语句的基础上封装了视图 VIEW，这样就会基于 SQL 语句的结果集形成一张虚拟表。
>
> 说明2：在创建视图时，没有在视图名后面指定字段列表，则视图中字段列表默认和SELECT语句中的字段列表一致。如果SELECT语句中给字段取了别名，那么视图中的字段名和别名相同。



- **创建多表联合视图**

举例：

```mysql
CREATE VIEW empview 
AS 
SELECT employee_id emp_id,last_name NAME,department_name
FROM employees e,departments d
WHERE e.department_id = d.department_id;
```

```mysql
CREATE VIEW emp_dept
AS 
SELECT ename,dname
FROM t_employee LEFT JOIN t_department
ON t_employee.did = t_department.did;
```

```mysql
CREATE VIEW	dept_sum_vu
(name, minsal, maxsal, avgsal)
AS 
SELECT d.department_name, MIN(e.salary), MAX(e.salary),AVG(e.salary)
FROM employees e, departments d
WHERE e.department_id = d.department_id 
GROUP BY  d.department_name;
```

- **利用视图对数据进行格式化**

我们经常需要输出某个格式的内容，比如我们想输出员工姓名和对应的部门名，对应格式为 emp_name(department_name)，就可以使用视图来完成数据格式化的操作：

```mysql
CREATE VIEW emp_depart
AS
SELECT CONCAT(last_name,'(',department_name,')') AS emp_dept
FROM employees e JOIN departments d
WHERE e.department_id = d.department_id
```



- **基于视图创建视图**

当我们创建好一张视图之后，还可以在它的基础上继续创建视图。

举例：联合“emp_dept”视图和“emp_year_salary”视图查询员工姓名、部门名称、年薪信息创建 “emp_dept_ysalary”视图。

```mysql
CREATE VIEW emp_dept_ysalary
AS 
SELECT emp_dept.ename,dname,year_salary
FROM emp_dept INNER JOIN emp_year_salary
ON emp_dept.ename = emp_year_salary.ename;
```



### 查看视图

语法1：查看数据库的表对象、视图对象

```mysql
SHOW TABLES;
```

语法2：查看视图的结构

```mysql
DESC/ DESCRIBE 视图名称;
```

语法3：查看视图的属性信息

```mysql
 查看视图信息（显示数据表的存储引擎、版本、数据行数和数据大小等）
SHOW TABLE STATUS LIKE '视图名称'\G
```

执行结果显示，注释Comment为VIEW，说明该表为视图，其他的信息为NULL，说明这是一个虚表。

语法4：查看视图的详细定义信息

```mysql
SHOW CREATE VIEW 视图名称;
```



### 更新视图的数据



- **一般情况**

MySQL支持使用INSERT、UPDATE和DELETE语句对视图中的数据进行插入、更新和删除操作。当视图中的数据发生变化时，数据表中的数据也会发生变化，反之亦然。

举例：UPDATE操作

```mysql
mysql> SELECT ename,tel FROM emp_tel WHERE ename = '孙洪亮';
+---------+-------------+
| ename   | tel         |
+---------+-------------+
| 孙洪亮 	| 13789098765 |
+---------+-------------+
1 row in set (0.01 sec)

mysql> UPDATE emp_tel SET tel = '13789091234' WHERE ename = '孙洪亮';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT ename,tel FROM emp_tel WHERE ename = '孙洪亮';
+---------+-------------+
| ename	  | tel         |
+---------+-------------+
| 	孙洪亮 | 13789091234 |
+---------+-------------+
1 row in set (0.00 sec)

mysql> SELECT ename,tel FROM t_employee WHERE ename = '孙洪亮';
+---------+-------------+
| ename   | tel         |
+---------+-------------+
| 孙洪亮 	| 13789091234 |
+---------+-------------+
1 row in set (0.00 sec)

```

举例：DELETE操作

```mysql
mysql> SELECT ename,tel FROM emp_tel WHERE ename = '孙洪亮';
+---------+-------------+
| ename  	| tel           |
+---------+-------------+
| 孙洪亮 	| 13789091234 |
+---------+-------------+
1 row in set (0.00 sec)

mysql> DELETE FROM emp_tel  WHERE ename = '孙洪亮';
Query OK, 1 row affected (0.01 sec)

mysql> SELECT ename,tel FROM emp_tel WHERE ename = '孙洪亮';
Empty set (0.00 sec)

mysql> SELECT ename,tel FROM t_employee WHERE ename = '孙洪亮';
Empty set (0.00 sec)

```



- **不可更新的视图**

- 要使视图可更新，视图中的行和底层基本表中的行之间必须存在`一对一`的关系。另外当视图定义出现如下情况时，视图不支持更新操作：

  - 在定义视图的时候指定了“ALGORITHM = TEMPTABLE”，视图将不支持INSERT和DELETE操作；

  - 视图中不包含基表中所有被定义为非空又未指定默认值的列，视图将不支持INSERT操作；

  - 在定义视图的SELECT语句中使用了`JOIN联合查询`，视图将不支持INSERT和DELETE操作；

  - 在定义视图的SELECT语句后的字段列表中使用了`数学表达式`或`子查询`，视图将不支持INSERT，也不支持UPDATE使用了数学表达式、子查询的字段值；

  - 在定义视图的SELECT语句后的字段列表中使用`DISTINCT`、`聚合函数`、`GROUP BY`、`HAVING`、`UNION`等，视图将不支持INSERT、UPDATE、DELETE；

  - 在定义视图的SELECT语句中包含了子查询，而子查询中引用了FROM后面的表，视图将不支持INSERT、UPDATE、DELETE；

  - 视图定义基于一个`不可更新视图`；

  - 常量视图。

- <font color='red'>即：基表和视图的字段不存在一一对应的关系时，都不可以执行添加、更新和删除等操作</font>

举例：

```mysql
mysql> CREATE OR REPLACE VIEW emp_dept
    -> (ename,salary,birthday,tel,email,hiredate,dname)
    -> AS SELECT ename,salary,birthday,tel,email,hiredate,dname
    -> FROM t_employee INNER JOIN t_department
    -> ON t_employee.did = t_department.did ;
Query OK, 0 rows affected (0.01 sec)

```

```mysql
mysql> INSERT INTO emp_dept(ename,salary,birthday,tel,email,hiredate,dname)
    -> VALUES('张三',15000,'1995-01-08','18201587896',
    -> 'zs@atguigu.com','2022-02-14','新部门');
    
ERROR 1393 (HY000): Can not modify more than one base table through a join view 'atguigu_chapter9.emp_dept'

```

从上面的SQL执行结果可以看出，在定义视图的SELECT语句中使用了**JOIN联合查询**，视图将不支持更新操作。

> 虽然可以更新视图数据，但总的来说，视图作为`虚拟表`，主要用于`方便查询`，不建议更新视图的数据。**对视图数据的更改，都是通过对实际数据表里数据的操作来完成的。**



### 修改、删除视图



**修改视图**

方式1：使用CREATE **OR REPLACE** VIEW 子句**修改视图**

```mysql
CREATE OR REPLACE VIEW empvu80
(id_number, name, sal, department_id)
AS 
SELECT  employee_id, first_name || ' ' || last_name, salary, department_id
FROM employees
WHERE department_id = 80;
```

> 说明：CREATE VIEW 子句中各列的别名应和子查询中各列相对应。

方式2：ALTER VIEW

修改视图的语法是：

```mysql
ALTER VIEW 视图名称 
AS
新的查询语句
```



**删除视图**

- 删除视图只是删除视图的定义，并不会删除基表的数据。

- 删除视图的语法是：

  ```mysql
  DROP VIEW IF EXISTS 视图名称;
  ```

  ```mysql
  DROP VIEW IF EXISTS 视图名称1,视图名称2,视图名称3,...;
  ```

- 举例：

  ```mysql
  DROP VIEW empvu80;
  ```

- 说明：基于视图a、b创建了新的视图c，如果将视图a或者视图b删除，会导致视图c的查询失败。这样的视图c需要手动删除或修改，否则影响使用。





## 变量



### 系统变量

系统变量 是MySQL服务器提供，不是用户定义的，属于服务器层面。

- 分为全局变量（GLOBAL）、
- 会话变量（SESSION）。



1). 查看系统变量

```mysql
SHOW [ SESSION | GLOBAL ] VARIABLES ; -- 查看所有系统变量
SHOW [ SESSION | GLOBAL ] VARIABLES LIKE '......'; -- 可以通过LIKE模糊匹配方式查找变量
SELECT @@[SESSION | GLOBAL].系统变量名; -- 查看指定变量的值
```

2). 设置系统变量

```mysql
SET [ SESSION | GLOBAL ] 系统变量名 = 值 ;
SET @@[SESSION | GLOBAL].系统变量名 = 值  ;
```

>注意:
>
>如果没有指定SESSION/GLOBAL，默认是SESSION，会话变量。
>
>```
>mysql服务重新启动之后，所设置的全局参数会失效，要想不失效，可以在 /etc/my.cnf 中配置。
>```
>
>A. 全局变量(GLOBAL): 全局变量针对于所有的会话。
>
>B. 会话变量(SESSION): 会话变量针对于单个会话，在另外一个会话窗口就不生效了。



**演示示例:**

```mysql
-- 查看系统变量
show session variables ;

show session variables like 'auto%';
show global variables like 'auto%';

select @@global.autocommit;
select @@session.autocommit;

-- 设置系统变量
set session autocommit = 1;

set global autocommit = 0;

select @@global.autocommit;
```



### 用户定义变量

用户定义变量 是用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用 "@变量名" 使用就可以。其作用域为当前连接。



1). 赋值

方式一:

```mysql
SET @var_name = expr [, @var_name = expr] ... ;
SET @var_name := expr [, @var_name := expr] ... ;
```

赋值时，可以使用 = ，也可以使用 := （推荐）。

方式二:

```mysql
SELECT @var_name := expr [, @var_name := expr] ... ;
SELECT 字段名 INTO @var_name FROM 表名;
```



2). 使用

```mysql
SELECT @var_name;
```

>注意: 用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NULL。



**演示示例:**

```mysql
-- 赋值
set @myname = 'itcast';
set @myage := 10;
set @mygender := '男',@myhobby := 'java';

select @mycolor := 'red';
select count(*) into @mycount from tb_user;

-- 使用
select @myname,@myage,@mygender,@myhobby;
select @mycolor , @mycount;
select @abc;
```



### 局部变量

局部变量 是根据需要定义的在局部生效的变量，访问之前，需要DECLARE声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的BEGIN ... END块。



1). 声明

```mysql
DECLARE 变量名 变量类型 [DEFAULT ... ] ;
```

变量类型就是数据库字段类型：INT、BIGINT、CHAR、VARCHAR、DATE、TIME等。



2). 赋值

```mysql
SET 变量名 = 值 ;
SET 变量名 := 值 ;
SELECT 字段名 INTO 变量名 FROM 表名 ... ;
```



**演示示例:**

```mysql
-- 声明局部变量 - declare
-- 赋值
create procedure p2()
begin
    declare stu_count int default 0;
    
    select count(*) into stu_count from student;
    select stu_count;
end;

call p2();
```



### 游标

上面的局部变量等只能接收标量，

而游标可以接收多个数据的值



游标（CURSOR）是用来存储**查询结果集**的数据类型 , 在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用包括游标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。



- 声明游标

  - ```mysql
    DECLARE 游标名称 CURSOR FOR 查询语句 ;
    ```

- 打开游标

  - ```mysql
    OPEN 游标名称 ;
    ```

- 获取游标记录

  - ```mysql
    FETCH 游标名称 INTO 变量 [, 变量 ] ;
    ```

- 关闭游标

  - ```mysql
    CLOSE 游标名称 ;
    ```



**案例**

根据传入的参数uage，来查询用户表tb_user中，所有的用户年龄小于等于uage的用户姓名（name）和专业（profession），并将用户的姓名和专业插入到所创建的一张新表(id,name,profession)中

```mysql
-- 逻辑:
-- A. 声明游标, 存储查询结果集
-- B. 准备: 创建表结构
-- C. 开启游标
-- D. 获取游标中的记录
-- E. 插入数据到新表中
-- F. 关闭游标

create procedure p11(in uage int)
begin
    declare uname varchar(100);
    declare upro varchar(100);
    declare u_cursor cursor for select name,profession from tb_user where age <= uage;
    
    drop table if exists tb_user_pro;
    create table if not exists tb_user_pro(
        id int primary key auto_increment,
        name varchar(100),
        profession varchar(100)
    );
    
    open u_cursor;
        while true do
            fetch u_cursor into uname,upro;
            insert into tb_user_pro values (null, uname, upro);
        end while;
    close u_cursor;
end;

 调用：
call p11(30);
```

```
上述的存储过程，最终我们在调用的过程中，会报错，之所以报错是因为上面的while循环中，并没有退出条件。当游标的数据集获取完毕之后，再次获取数据，就会报错，从而终止了程序的执行
但是此时，tb_user_pro表结构及其数据都已经插入成功了，我们可以直接刷新表结构，检查表结构
中的数据。
```

上述的功能，虽然我们实现了，但是逻辑并不完善，而且程序执行完毕，获取不到数据，数据库还报错。 接下来，我们就需要来完成这个存储过程，并且解决这个问题。

要想解决这个问题，就需要通过MySQL中提供的 条件处理程序 Handler 来解决



### 条件处理程序

条件处理程序（Handler）可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。具体语法为：

```mysql
DECLARE handler_action HANDLER FOR condition_value [, condition_value]... statement ;
```

>- handler_action 的取值：
>  - CONTINUE: 继续执行当前程序
>  - EXIT: 终止执行当前程序
>- condition_value 的取值：
>  - SQLSTATE sqlstate_value: 状态码，如 02000
>  - SQLWARNING: 所有以01开头的SQLSTATE代码的简写
>  - NOT FOUND: 所有以02开头的SQLSTATE代码的简写
>  - SQLEXCEPTION: 所有没有被SQLWARNING 或 NOT FOUND捕获的SQLSTATE代码的简写
>- statement的取值：
>  - 为具体操作的时候执行这个条件处理程序
>  - eg：close u_cursor（关闭某个游标时）



 **案例**

我们继续来完成在上一小节提出的这个需求，并解决其中的问题。

根据传入的参数uage，来查询用户表tb_user中，所有的用户年龄小于等于uage的用户姓名（name）和专业（profession），并将用户的姓名和专业插入到所创建的一张新表(id,name,profession)中。

A. 通过 **SQLSTATE** 指定**具体的**状态码

```mysql
-- 逻辑:
-- A. 声明游标, 存储查询结果集
-- B. 准备: 创建表结构
-- C. 开启游标
-- D. 获取游标中的记录
-- E. 插入数据到新表中
-- F. 关闭游标
create procedure p11(in uage int)
begin
    declare uname varchar(100);
    declare upro varchar(100);
    declare u_cursor cursor for select name,profession from tb_user where age <= uage;
    
    -- 声明条件处理程序 ： 当SQL语句执行抛出的状态码为02000时，将关闭游标u_cursor，并退出
    declare exit handler for SQLSTATE '02000' close u_cursor;
    
    drop table if exists tb_user_pro;
    create table if not exists tb_user_pro(
        id int primary key auto_increment,
        name varchar(100),
        profession varchar(100)
    );

    open u_cursor;
        while true do
            fetch u_cursor into uname,upro;
            insert into tb_user_pro values (null, uname, upro);
        end while;
    close u_cursor;	#自动执行条件处理程序
end;

 调用：
call p11(30);
```



B. 通过SQLSTATE的代码简写方式 NOT FOUND

02 开头的状态码，代码简写为 NOT FOUND

```mysql
create procedure p12(in uage int)
begin
    declare uname varchar(100);
    declare upro varchar(100);
    declare u_cursor cursor for select name,profession from tb_user where age <= uage;
    
    -- 声明条件处理程序 ： 当SQL语句执行抛出的状态码为02开头时，将关闭游标u_cursor，并退出
    declare exit handler for not found close u_cursor;		#  NOT FOUND
    
    drop table if exists tb_user_pro;
    create table if not exists tb_user_pro(
        id int primary key auto_increment,
        name varchar(100),
        profession varchar(100)
    );
    
    open u_cursor;
        while true do
            fetch u_cursor into uname,upro;
            insert into tb_user_pro values (null, uname, upro);
        end while;
    close u_cursor;
end;

 调用：
call p12(30);
```





## 存储过程PROCEDURE



### 理解和优缺点

**含义**：存储过程的英文是 Stored Procedure 。它的思想很简单，就是一组经过 预先编译 的 SQL 语句的封装。

执行过程：存储过程预先存储在 MySQL 服务器上，需要执行的时候，客户端只需要向服务器端发出调用存储过程的命令，服务器端就可以把预先存储好的这一系列 SQL 语句全部执行。

**好处**：

1、简化操作，提高了sql语句的重用性，减少了开发程序员的压力

2、减少操作过程中的失误，提高效率

3、减少网络传输量（客户端不需要把所有的 SQL 语句通过网络发给服务器） 

4、减少了 SQL 语句暴露在网上的风险，也提高了数据查询的安全性

**和视图、函数的对比**：

它和视图有着同样的优点，清晰、安全，还可以减少网络传输量。不过它和视图不同，视图是 虚拟表 ，通常不对底层数据表直接操作，而存储过程是程序化的 SQL，可以 直接操作底层数据表 ，相比于面向集合的操作方式，能够实现一些更复杂的数据处理。一旦存储过程被创建出来，使用它就像使用函数一样简单，我们直接通过调用存储过程名即可。相较于函数，存储过程是 没有返回值 的。



- **优点**
  - **存储过程可以一次编译多次使用**
  - **可以减少开发工作量**
  - **存储过程的安全性强。**
  - **可以减少网络传输量**
  - **良好的封装性**
- **缺点**
  - **可移植性差。**
  - **调试困难。**
  - **存储过程的版本管理很困难。**
  - **它不适合高并发的场景**



### 基本语法

- <font color='cornflowerblue'>创建</font>

  - ```mysql
    CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名 参数类型,...)
    [characteristics ...]
    BEGIN
    	存储过程体
    END
    ```

  - 参数的类型，主要分为以下三种：IN、OUT、INOUT。 具体的含义如下：

    - IN ：当前参数为输入参数，也就是表示入参；
      - 存储过程只是读取这个参数的值。如果没有定义参数种类， <font color='cornflowerblue'>默认就是 IN </font>，表示输入参数。
    - OUT ：当前参数为输出参数，也就是表示出参；
      - 执行完成之后，调用这个存储过程的客户端或者应用程序就可以读取这个参数返回的值了。
    - INOUT ：当前参数既可以为输入参数，也可以为输出参数。

  - **注意：在命令行中，执行创建存储过程的SQL时，需要通过关键字 delimiter 指定SQL语句的结束符。**

  - > characteristics 表示创建存储过程时指定的对存储过程的约束条件，其取值信息如下：
    >
    >```mysql
    >LANGUAGE SQL
    >| [NOT] DETERMINISTIC
    >| { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
    >| SQL SECURITY { DEFINER | INVOKER }
    >| COMMENT 'string
    >```
    >
    >- `LANGUAGE SQL` ：说明存储过程执行体是由SQL语句组成的，当前系统支持的语言为SQL。
    >- `[NOT] DETERMINISTIC `：指明存储过程执行的结果是否确定。
    >  - DETERMINISTIC表示结果是确定的。每次执行存储过程时，相同的输入会得到相同的输出。
    >  - NOT DETERMINISTIC表示结果是不确定的，相同的输入可能得到不同的输出。
    >  - 如果没有指定任意一个值，默认为NOT DETERMINISTIC。
    >- `{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA } `：指明子程序使用SQL语句的限制。
    >  - CONTAINS SQL：表示子程序包含SQL语句，但不包含读或写数据的语句。
    >  - NO SQL：表示子程序中不包含SQL语句
    >  - READS SQL DATA：表示子程序中包含读数据的语句。
    >  - MODIFIES SQL DATA：表示子程序中包含写数据的语句。
    >  - 默认情况下，系统会指定为CONTAINS SQL
    >- `SQL SECURITY { DEFINER | INVOKER } `：指明谁有权限来执行。
    >  - `DEFINER` ：表示只有定义者自己才能够执行。
    >  - `INVOKER` 表示权限的调用者可以执行
    >  - 如果没有设置相关的值，则MySQL默认指定值为DEFINER。
    >- `COMMENT 'string'` ：表示注释信息。

  - 举例

    - ```mysql
      create procedure p1()
      begin
      	declare student_count int
      	select count(*) into student_count from student;
      	
      	select student_count;
      end;
      ```

    - ```mysql
      # 命令行中：
      DELIMITER $
      
      CREATE PROCEDURE select_all_data()
      BEGIN
      	SELECT * FROM emps;
      END $
      
      DELIMITER ;
      ```

      

- <font color='cornflowerblue'>调用</font>

  - ```mysql
    CALL 存储过程名(实参列表)
    ```

  - 1、调用in模式的参数：

    - ```mysql
      CALL sp1('值');
      ```

  - 2、调用out模式的参数：

    - ```mysql
      SET @name;
      CALL sp1(@name);
      SELECT @name;
      ```

  - 3、调用inout模式的参数：

    - ```mysql
      SET @name=值;
      CALL sp1(@name);
      SELECT @name;
      ```

  - ```mysql
    # 举例：
    DELIMITER //
    
    CREATE PROCEDURE CountProc(IN sid INT,OUT num INT)
    BEGIN
    	SELECT COUNT(*) INTO num FROM fruits
    WHERE s_id = sid;
    END //
    
    DELIMITER ;
    
    # 调用存储过程：
    mysql> CALL CountProc (101, @num);
    Query OK, 1 row affected (0.00 sec)
    
    # 查看返回结果：
    mysql> SELECT @num;
    ```

    

- <font color='cornflowerblue'>查看</font>

  - **使用SHOW CREATE语句查看存储过程和函数的创建信息**

    - ```mysql
      SHOW CREATE {PROCEDURE | FUNCTION} 存储过程名或函数名（未指定数据库默认为本数据库）
      
      # 举例：
      SHOW CREATE PROCEDURE test_db.CountProc \G
      ```

  - **2.** **使用SHOW STATUS语句查看存储过程和函数的状态信息**

    - ```mysql
      SHOW {PROCEDURE | FUNCTION} STATUS [LIKE 'pattern'] \G
      ```

  - **3.** **从information_schema.Routines表中查看存储过程和函数的信息**

    - ```mysql
      SELECT * FROM information_schema.Routines
      WHERE ROUTINE_NAME='存储过程或函数的名' [AND ROUTINE_TYPE = {'PROCEDURE|FUNCTION'}];
      
      # 举例：
      SELECT * FROM information_schema.Routines
      WHERE ROUTINE_NAME='count_by_id' AND ROUTINE_TYPE = 'FUNCTION' \G
      ```

      

- <font color='cornflowerblue'>修改</font>

  - 修改存储过程或函数，**不能修改存储过程体**或函数功能，**只是修改相关特性**。使用ALTER语句实现。

  - ```mysql
    ALTER {PROCEDURE | FUNCTION} 存储过程或函数的名 [characteristic ...]
    
    # 举例：
    ALTER PROCEDURE CountProc
    MODIFIES SQL DATA
    SQL SECURITY INVOKER ;
    ```



- <font color='cornflowerblue'>删除</font>

  - ```mysql
    DROP {PROCEDURE | FUNCTION} [IF EXISTS] 存储过程或函数的名
    ```





## 存储函数FUNCTION

存储函数是有返回值的存储过程，存储函数的参数**只能是IN类型的**。具体语法如下：

学过的函数：LENGTH、SUBSTR、CONCAT等

语法格式：

```mysql
CREATE FUNCTION 函数名(参数名 参数类型,...)
RETURNS 返回值类型
[characteristics ...]
BEGIN
	函数体 #函数体中肯定有 RETURN 语句
END
```



- FUNCTION中参数只能是IN类型的。
- RETURNS type 语句表示函数返回数据的类型；
  - RETURNS子句只能对FUNCTION做指定，对函数而言这是 强制 的。它用来指定函数的返回类型，而且函数体必须包含一个 RETURN value 语句。
- characteristic 创建函数时指定的对函数的约束。取值与创建存储过程时相同，这里不再赘述。
- 函数体也可以用BEGIN…END来表示SQL代码的开始和结束。如果函数体只有一条语句，也可以省略BEGIN…END。



**调用存储函数**

```mysql
SELECT 函数名(实参列表)
```



举例：

```mysql
create function fun1([IN] n int)
returns int 
	deterministic
begin
    declare total int default 0;
    while n>0 do
        set total := total + n;
        set n := n - 1;
    end while;
    return total;
end;

 调用：
select fun1(50);
```





## 控制过程



### IF

if 用于做条件判断，具体的语法结构为：

```mysql
IF 条件1 THEN
	.....
ELSEIF 条件2 THEN 	-- 可选
	.....
ELSE 				 -- 可选
	.....
END IF;
```

在if条件判断的结构中，ELSE IF 结构可以有多个，也可以没有。 ELSE结构可以有，也可以没有。



**案例**

根据定义的分数score变量，判定当前分数对应的分数等级。

- score >= 85分，等级为优秀。
- score >= 60分 且 score < 85分，等级为及格。
- score < 60分，等级为不及格。

```mysql
create procedure p3(in score int, out result varchar(10))
begin
    
    if score >= 85 then
    	set result := '优秀';
    elseif score >= 60 then
    	set result := '及格';
    else
    	set result := '不及格';
    end if;
    
end;

 调用：
call p3(18, @result);
select @result;
```



### CASE

case结构及作用，和我们在基础篇中所讲解的流程控制函数很类似。有两种语法格式：

语法1：

```mysql
-- 含义： 当case_value的值为 when_value1时，执行statement_list1，当值为 when_value2时，执行statement_list2， 否则就执行 statement_list
CASE case_value
    WHEN when_value1 THEN statement_list1
    [ WHEN when_value2 THEN statement_list2] ...
    [ ELSE statement_list ]
END CASE;
```

语法2：

```mysql
-- 含义： 当条件search_condition1成立时，执行statement_list1，当条件search_condition2成立时，执行statement_list2， 否则就执行 statement_list
CASE
    WHEN search_condition1 THEN statement_list1
    [WHEN search_condition2 THEN statement_list2] ...
    [ELSE statement_list]
END CASE;
```



**案例**

根据传入的月份，判定月份所属的季节（要求采用case结构）。

- 1-3月份，为第一季度
- 4-6月份，为第二季度
- 7-9月份，为第三季度
- 10-12月份，为第四季度

```mysql
create procedure p6(in month int)
begin
    declare result varchar(10);
    case
        when month >= 1 and month <= 3 then
        	set result := '第一季度';
        when month >= 4 and month <= 6 then
        	set result := '第二季度';
        when month >= 7 and month <= 9 then
        	set result := '第三季度';
        when month >= 10 and month <= 12 then
        	set result := '第四季度';
        else
        	set result := '非法参数';
    end case ;
    
    select concat('您输入的月份为: ',month, ', 所属的季度为: ',result);
end;

 调用：
call p6(16);
```





### WHILE

while 循环是有条件的循环控制语句。满足条件后，再执行循环体中的SQL语句。具体语法为：

```mysql
-- 先判定条件，如果条件为true，则执行逻辑，否则，不执行逻辑
WHILE 条件 DO
    SQL逻辑...
END WHILE;
```



 **案例**

计算从1累加到n的值，n为传入的参数值。

```mysql
-- A. 定义局部变量, 记录累加之后的值;
-- B. 每循环一次, 就会对n进行减1 , 如果n减到0, 则退出循环
create procedure p7(in n int)
begin
    declare total int default 0;
    
    while n>0 do
        set total := total + n;
        set n := n - 1;
    end while;
    
    select total;
end;

 调用：
call p7(100);
```



### REPEAT

repeat是有条件的循环控制语句, 当**满足until声明的条件的时候，则退出循环** 。具体语法为：

```mysql
-- 先执行一次逻辑，然后判定UNTIL条件是否满足，如果满足，则退出。如果不满足，则继续下一次循环
REPEAT
    SQL逻辑...
    UNTIL 条件
END REPEAT;
```

 **案例**

计算从1累加到n的值，n为传入的参数值。(使用repeat实现)

```mysql
-- A. 定义局部变量, 记录累加之后的值;
-- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环
create procedure p8(in n int)
begin
    declare total int default 0;
    
    repeat
        set total := total + n;
        set n := n - 1;
        until n <= 0
    end repeat;
    
    select total;
end;

 调用：
call p8(10);
call p8(100);
```



### LOOP

LOOP 实现简单的循环，如果不在SQL逻辑中增加退出循环的条件，可以用其来实现简单的死循环。

LOOP可以配合一下两个语句使用：

- LEAVE ：配合循环使用，退出循环。
- ITERATE：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

```mysql
[begin_label:] LOOP
SQL逻辑...
END LOOP [end_label];


LEAVE label; -- 退出指定标记的循环体
ITERATE label; -- 直接进入下一次循环
```

上述语法中出现的 begin_label，end_label，label 指的都是我们所自定义的标记。



**案例一**

计算从1累加到n的值，n为传入的参数值。

```mysql
-- A. 定义局部变量, 记录累加之后的值;
-- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环 ----> leave xx
create procedure p9(in n int)
begin
    declare total int default 0;
    
    sum:loop
    
        if n<=0 then
        	leave sum;
        end if;
        
        set total := total + n;
        set n := n - 1;
    end loop sum;
    
    select total;
end;

 调用：
call p9(100);
```

 **案例二**

计算从1到n之间的偶数累加的值，n为传入的参数值。

```mysql
-- A. 定义局部变量, 记录累加之后的值;
-- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环 ----> leave xx
-- C. 如果当次累加的数据是奇数, 则直接进入下一次循环. --------> iterate xx
create procedure p10(in n int)
begin
    declare total int default 0;
    
    sum:loop
        if n<=0 then
        	leave sum;
        end if;
        
        if n%2 = 1 then
            set n := n - 1;
            iterate sum;
        end if;
        	set total := total + n;
        	set n := n - 1;
    end loop sum;
    
    select total;
end;

 调用：
call p10(100);
```





## 触发器TRIGGER



触发器是与表有关的数据库对象，指在insert/update/delete之前(BEFORE)或之后(AFTER)，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性, 日志记录 , 数据校验等操作 。

使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。



触发器类型：

- INSERT 型触发器 
  - NEW 表示将要或者已经新增的数据
- UPDATE 型触发器 
  - OLD 表示修改之前的数据 , NEW 表示将要或已经修改后的数据
- DELETE 型触发器 
  - OLD 表示将要或者已经删除的数据



**语法**

1). 创建

```mysql
CREATE TRIGGER trigger_name
	BEFORE/AFTER INSERT/UPDATE/DELETE
	ON tbl_name FOR EACH ROW -- 行级触发器
BEGIN
	trigger_stmt ;
END;
```

2). 查看

```mysql
SHOW TRIGGERS ;
```

3). 删除

```mysql
DROP TRIGGER [schema_name.]trigger_name ; -- 如果没有指定 schema_name，默认为当前数据库 。
```



**案例**

通过触发器记录 tb_user 表的数据变更日志，将变更日志插入到日志表user_logs中, 包含增加,

修改 , 删除 ;

表结构准备：

```mysql
-- 准备工作 : 日志表 user_logs
create table user_logs(
    id int(11) not null auto_increment,
    operation varchar(20) not null comment '操作类型, insert/update/delete',
    operate_time datetime not null comment '操作时间',
    operate_id int(11) not null comment '操作的ID',
    operate_params varchar(500) comment '操作参数',
    primary key(`id`)
)engine=innodb default charset=utf8;
```

A. 插入数据触发器

```mysql
create trigger tb_user_insert_trigger
	after insert on tb_user for each row
begin
    insert into user_logs(id, operation, operate_time, operate_id, operate_params)
    VALUES
    (null, 'insert', now(), new.id, concat('插入的数据内容为:
    id=',new.id,',name=',new.name, ', phone=', NEW.phone, ', email=', NEW.email, ',
    profession=', NEW.profession));
end;
```

测试:

```mysql
-- 查看
show triggers ;
-- 插入数据到tb_user
insert into tb_user(id, name, phone, email, profession, age, gender, status,createtime) VALUES (26,'三皇子','18809091212','erhuangzi@163.com','软件工程',23,'1','1',now());
```

 测试完毕之后，检查日志表中的数据是否可以正常插入，以及插入数据的正确性



B. 修改数据触发器

```mysql
create trigger tb_user_update_trigger
	after update on tb_user for each row
begin
    insert into user_logs(id, operation, operate_time, operate_id, operate_params)
    VALUES
    (null, 'update', now(), new.id,
    concat('更新之前的数据: id=',old.id,',name=',old.name, ', phone=',
    old.phone, ', email=', old.email, ', profession=', old.profession,
    ' | 更新之后的数据: id=',new.id,',name=',new.name, ', phone=',
    NEW.phone, ', email=', NEW.email, ', profession=', NEW.profession));
end;
```

测试:

```mysql
-- 查看
show triggers ;

-- 更新
update tb_user set profession = '会计' where id = 23;
update tb_user set profession = '会计' where id <= 5;
```

测试完毕之后，检查日志表中的数据是否可以正常插入，以及插入数据的正确性。



C. 删除数据触发器

```mysql
create trigger tb_user_delete_trigger
	after delete on tb_user for each row
begin
    insert into user_logs(id, operation, operate_time, operate_id, operate_params)
    VALUES
    (null, 'delete', now(), old.id,
    concat('删除之前的数据: id=',old.id,',name=',old.name, ', phone=',
    old.phone, ', email=', old.email, ', profession=', old.profession));
end;
```

测试:

```mysql
-- 查看
show triggers ;

-- 删除数据
delete from tb_user where id = 26;
```

测试完毕之后，检查日志表中的数据是否可以正常插入，以及插入数据的正确性。





# MySQL8.0新特性



## 新特性1：窗口函数

MySQL从8.0版本开始支持窗口函数。窗口函数的作用类似于在查询中对数据进行分组，

不同的是，**分组操作会把分组的结果聚合成一条记录，而窗口函数是将结果置于每一条数据记录中。**

<font color='red'>即：窗口函数是在每行新增一个字段，用于显示窗口函数的计算值，并将同一个组的数据放在一起，但是并不合并</font>

<font color='cornflowerblue'>强化、代替	GROUP BY</font>



窗口函数的分类：

![image-20211012162944536](image/MySQL(1)基础篇.assets/image-20211012162944536.png)



具体使用案例看尚硅谷基础篇笔记 《MySQL8.0新特性》篇



## 新特性2：公用表达式

公用表表达式（或通用表表达式）简称为CTE（Common Table Expressions）。CTE是一个命名的临时结果集，作用范围是当前语句。CTE可以理解成一个可以复用的子查询，当然跟子查询还是有点区别的，CTE可以引用其他CTE，但子查询不能引用其他子查询。所以，可以考虑代替子查询。

依据语法结构和执行方式的不同，公用表表达式分为`普通公用表表达式`和`递归公用表表达式` 2 种。

<font color='cornflowerblue'>强化、代替	子查询</font>



具体使用案例看尚硅谷基础篇笔记 《MySQL8.0新特性》篇



















































































