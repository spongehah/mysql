[TOC]



 架构篇

>学习课程：B站搜尚硅谷MySQL康师傅
>
>author：hah

# MySQL架构篇

# Linux下MySQL的安装与使用

> 具体看尚硅谷笔记高级篇《第01章 Linux下MySQL的安装与使用》

## Linux版MySQL的安装和卸载

> 具体看尚硅谷笔记高级篇《第01章 Linux下MySQL的安装与使用》



有需要还可以设置密码的安全设置



## 字符集的相关操作



- 修改MySQL5.7字符集

  - 在MySQL 8.0版本之前，默认字符集为 latin1 ，utf8字符集指向的是 utf8mb3 。网站开发人员在数据库设计的时候往往会将编码修改为utf8字符集。如果遗忘修改默认的编码，就会出现乱码的问题。从MySQL8.0开始，数据库的默认编码将改为 utf8mb4 ，从而避免上述乱码的问题。

  - ```mysql
    # 操作1：查看默认使用的字符集
    show variables like 'character%';
    # 或者
    show variables like '%char%';
    ```

  - ```mysql
    # 修改：
    vim /etc/my.cnf
    # 加上
    character_set_server=utf8
    # 重启
    systemctl restart mysqld
    ```

  - MySQL5.7版本中，以前创建的库，创建的表字符集还是latin1。

  - 并且，在以前的库新创建的表也是latin1。

  - <font color='red'>因为创建表的字符集默认是和数据库一致的</font>



- 各级别的字符集
  - 服务器级别
  - 数据库级别
  - 表级别
  - 列级别
  - <font color='red'>若未设置字符集，则会默认采用它上一层的字符集级别</font>



**字符集与比较规则**

**utf8** **与** **utf8mb4**

utf8 字符集表示一个字符需要使用1～4个字节，但是我们常用的一些字符使用1～3个字节就可以表示了。而字符集表示一个字符所用的最大字节长度，在某些方面会影响系统的存储和性能，所以设计MySQL的设计者偷偷的定义了两个概念：

- utf8mb3 ：阉割过的 utf8 字符集，只使用1～3个字节表示字符。
- utf8mb4 ：正宗的 utf8 字符集，使用1～4个字节表示字符。

一般情况下使用utf8mb3 即可，若要存储emoji表情等，则需要使用utf8mb4 

**比较规则**

上表中，MySQL版本一共支持41种字符集，其中的 Default collation 列表示这种字符集中一种默认的比较规则，里面包含着该比较规则主要作用于哪种语言，比如 utf8_polish_ci 表示以波兰语的规则比较， utf8_spanish_ci 是以西班牙语的规则比较， utf8_general_ci 是一种通用的比较规则。

**常用操作示例：**

```mysql
查看GBK字符集的比较规则
SHOW COLLATION LIKE 'gbk%';
查看UTF-8字符集的比较规则
SHOW COLLATION LIKE 'utf8%';

查看服务器的字符集和比较规则
SHOW VARIABLES LIKE '%_server';
查看数据库的字符集和比较规则
SHOW VARIABLES LIKE '%_database';
查看具体数据库的字符集
SHOW CREATE DATABASE dbtest1;
修改具体数据库的字符集
ALTER DATABASE dbtest1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';

查看表的字符集
show create table employees;
查看表的比较规则
show table status from atguigudb like 'employees';
修改表的字符集和比较规则
ALTER TABLE emp1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
```

 

**请求到响应过程中字符集的变化**

。。。



## SQL大小写规范

 

Windows和Linux平台区别

在 SQL 中，关键字和函数名是不用区分字母大小写的，比如 SELECT、WHERE、ORDER、GROUP BY 等关键字，以及 ABS、MOD、ROUND、MAX 等函数名。不过在 SQL 中，你还是要确定大小写的规范，因为在 Linux 和 Windows 环境下，你可能会遇到不同的大小写问题。 **windows系统默认大小写不敏感 ，但是 linux系统是大小写敏感的** 。

通过如下命令查看：

```mysql
SHOW VARIABLES LIKE '%lower_case_table_names%';
```

lower_case_table_names参数值的设置：

- 默认为0，大小写敏感 。
- 设置1，大小写不敏感。创建的表，数据库都是以小写形式存放在磁盘上，对于sql语句都是转换为小写对表和数据库进行查找。
- 设置2，创建的表和数据库依据语句上格式存放，凡是查找都是转换为小写进行。

两个平台上SQL大小写的区别具体来说:

>MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：
>
>1、<font color='red'>数据库名、表名、表的别名、变量名是严格区分大小写的；</font>
>
>2、关键字、函数名称在 SQL 中不区分大小写；
>
>3、列名（或字段名）与列的别名（或字段别名）在所有的情况下均是忽略大小写的；
>
>
>
>**MySQL在Windows的环境下全部不区分大小写**



**Linux**下大小写规则设置

当想设置为大小写不敏感时，要在 my.cnf 这个配置文件 [mysqld] 中加入lower_case_table_names=1 ，然后重启服务器。

- 但是要在重启数据库实例之前就需要将原来的数据库和表转换为小写，否则将找不到数据库名。
- SHOW VARIABLES LIKE '%lower_case_table_names%'此参数适用于MySQL5.7。在MySQL 8下禁止在重新启动 MySQL 服务时将

lower_case_table_names 设置成不同于初始化 MySQL 服务时设置的lower_case_table_names 值。如果非要将MySQL8设置为大小写不敏感，具体步骤为：

- 1、停止MySQL服务
- 2、删除数据目录，即删除 /var/lib/mysql 目录
- 3、在MySQL配置文件（ /etc/my.cnf ）中添加 lower_case_table_names=1
- 4、启动MySQL服务



**SQL**编写建议

如果你的变量名命名规范没有统一，就可能产生错误。这里有一个有关命名规范的建议：

- \1. 关键字和函数名称全部大写；
- \2. 数据库名、表名、表别名、字段名、字段别名等全部小写；
- \3. SQL 语句必须以分号结尾。

数据库名、表名和字段名在 Linux MySQL 环境下是区分大小写的，因此建议你统一这些字段的命名规则，比如全部采用小写的方式。

虽然关键字和函数名称在 SQL 中不区分大小写，也就是如果小写的话同样可以执行。但是同时将关键词和函数名称全部大写，以便于区分数据库名、表名、字段名。





# MySQL的数据目录

> 具体看尚硅谷笔记高级篇《第02章_MySQL的数据目录》

## MySQL8的主要目录结构

```shell
find / -name mysql
```

**1.1** **数据库文件的存放路径** 

mysql下：

```mysql
show variables like 'datadir'; # /var/lib/mysql/
```

**1.2** **相关命令目录**

**相关命令目录：/usr/bin 和/usr/sbin。**

**1.3** **配置文件目录**

**配置文件目录：/usr/share/mysql-8.0（命令及配置文件），/etc/mysql（如my.cnf）**



## 数据库和表在文件系统中的表示



**数据库在文件系统中的表示**

看一下我的计算机上的数据目录下的内容：

```shell
cd /var/lib/mysql
ll
```



### InnoDB存储引擎模式

**1.** **表结构**

为了保存表结构，`InnoDB`在`数据目录`下对应的数据库子目录下创建了一个专门用于`描述表结构的文件`

先进入某个数据库文件，执行ll，可以看到如下结构

```shell
 MySQL5
-rw-r-----. 1 mysql mysql 61 8月 18 11:32 db.opt
-rw-r-----. 1 mysql mysql 8716 8月 18 11:32 departments.frm		#表结构
-rw-r-----. 1 mysql mysql 147456 8月 18 11:32 departments.ibd	#表数据
-rw-r-----. 1 mysql mysql 8982 8月 18 11:32 employees.frm
-rw-r-----. 1 mysql mysql 180224 8月 18 11:32 employees.ibd

 MySQL8
-rw-r-----. 1 mysql mysql 163840 7月 29 23:10 departments.ibd
-rw-r-----. 1 mysql mysql 196608 7月 29 23:10 employees.ibd
```

可以发现mysql8中只有ibd文件

- db.opt文件放到具体的表.ibd文件中了

- .frm和.ibd文件合并

- 可以使用以下命令查看ibd文件：

  - ```shell
    ibd2sdi --dump-file=xxx.txt xxx.ibd
    ```



**2.** **表中数据和索引**

**① 系统表空间（system tablespace）**

默认情况下，InnoDB会在数据目录下创建一个名为`ibdata1`、大小为`12M`的`自拓展`文件，这个文件就是对应的`系统表空间`在文件系统上的表示。在**MySQL5.5.7~5.6.6之间会将表数据存储在系统表空间**

**② 独立表空间(file-per-table tablespace)** 

**在MySQL5.6.6以及之后的版本中，InnoDB并不会默认的把各个表的数据存储到系统表空间中**，而是为`每一个表建立一个独立表空间`，也就是说我们创建了多少个表，就有多少个独立表空间。使用`独立表空间`来存储表数据的话，会在该表所属数据库对应的子目录下创建一个表示该独立表空间的文件，文件名和表名相同。

```
表名.ibd
```

> <font color='red'>MySQL8.0中不再单独提供`表名.frm`，而是合并在`表名.ibd`文件中。</font>



**③ 系统表空间与独立表空间的设置**

我们可以自己指定使用`系统表空间`还是`独立表空间`来存储数据，这个功能由启动参数`innodb_file_per_table`控制

```ini
[server] 
innodb_file_per_table=0 # 0：代表使用系统表空间； 1：代表使用独立表空间
```

**④ 其他类型的表空间**

随着MySQL的发展，除了上述两种老牌表空间之外，现在还新提出了一些不同类型的表空间，比如通用表空间（general tablespace）、临时表空间（temporary tablespace）等。



### MyISAM存储引擎模式

**1.** **表结构**

在存储表结构方面， MyISAM 和 InnoDB 一样，也是在`数据目录`下对应的数据库子目录下创建了一个专门用于描述表结构的文件

```shell
test.frm 存储表结构 #MySQL8.0 改为了 b.xxx.sdi
test.MYD 存储数据 (MYData) 		#.MYD和.MYI两个一起也就是8.0 InnoDB中的.ibd文件
test.MYI 存储索引 (MYIndex)
```

**2.** **表中数据和索引**

在MyISAM中的索引全部都是`二级索引`，该存储引擎的<font color='cornflowerblue'>`数据和索引是分开存放`</font>的。所以在文件系统中也是使用不同的文件来存储数据文件和索引文件，同时表数据都存放在对应的数据库子目录下。

```shell
test.frm 存储表结构 #MySQL8.0 改为了 b.xxx.sdi
test.MYD 存储数据 (MYData) 
test.MYI 存储索引 (MYIndex)
```



### 小结

举例： 数据库a ， 表b 。

1、如果表b采用 **InnoDB** ，data\a中会产生1个或者2个文件：

- **b.frm** ：描述表结构文件，字段长度等
- 如果采用 **系统表空间** 模式的，数据信息和索引信息都存储在 **ibdata1** 中
- 如果采用 **独立表空间** 存储模式，data\a中还会产生 **b.ibd** 文件（存储数据信息和索引信息）

此外：

① MySQL5.7 中会在data/a的目录下生成 **db.opt** 文件用于保存数据库的相关配置。比如：字符集、比较

规则。而MySQL8.0不再提供db.opt文件。

② MySQL8.0中不再单独提供b.frm，而是合并在b.ibd文件中。

2、如果表b采用 **MyISAM** ，data\a中会产生3个文件：

- MySQL5.7 中： **b.frm** ：描述表结构文件，字段长度等。

  MySQL8.0 中 **b.xxx.sdi** ：描述表结构文件，字段长度等

- **b.MYD** (MYData)：数据信息文件，存储数据信息(如果采用独立表存储模式)

- **b.MYI** (MYIndex)：存放索引信息文件	



# MySQL逻辑架构



## 1. 逻辑架构剖析

### 1.1 服务器处理客户端请求



首先MySQL是典型的C/S架构，即`Clinet/Server 架构`，服务端程序使用的`mysqld`。

不论客户端进程和服务器进程是采用哪种方式进行通信，最后实现的效果是：<font color='red'>**客户端进程向服务器进程发送一段文本（SQL语句），服务器进程处理后再向客户端进程发送一段文本（处理结果）**。</font>

那服务器进程对客户端进程发送的请求做了什么处理，才能产生最后的处理结果呢？这里以查询请求为 例展示：

![image-20230816163801104](image/MySQL(2)架构篇.assets/image-20230816163801104.png)

下面具体展开如下：

![MySQL服务器端的逻辑架构说明](image/MySQL(2)架构篇.assets/MySQL服务器端的逻辑架构说明.png)



接下来分别对上图的五层架构进行说明：



### 1.2 Connectors

Connectors, 指的是不同语言中与SQL的交互。MySQL首先是一个网络程序，在TCP之上定义了自己的应用层协议。所以要使用MySQL，我们可以编写代码，跟MySQL Server `建立TCP连接`，之后按照其定义好的协议进行交互。或者比较方便的方法是调用SDK，比如Native C API、JDBC、PHP等各语言MySQL Connecotr,或者通过ODBC。但**通过SDK来访问MySQL，本质上还是在TCP连接上通过MySQL协议跟MySQL进行交互**

**接下来的MySQL Server结构可以分为如下三层：**



### 1.3 第一层：连接层

系统（客户端）访问 MySQL 服务器前，做的第一件事就是建立 TCP 连接。 经过三次握手建立连接成功后， MySQL 服务器对 TCP 传输过来的账号密码做身份认证、权限获取。

- 用户名或密码不对，会收到一个Access denied for user错误，客户端程序结束执行
- 用户名密码认证通过，会从权限表查出账号拥有的权限与连接关联，之后的权限判断逻辑，都将依赖于此时读到的权限

TCP 连接收到请求后，必须要分配给一个线程专门与这个客户端的交互。所以还会有个线程池，去走后面的流程。每一个连接从线程池中获取线程，省去了创建和销毁线程的开销。

所以**连接管理**的职责是负责认证、管理连接、获取权限信息。

### 1.4 第二层：服务层

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成`缓存的查询`，SQL的分析和优化及部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。

在该层，服务器会`解析查询`并创建相应的内部`解析树`，并对其完成相应的`优化`：如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。

如果是SELECT语句，服务器还会`查询内部的缓存`。如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

- SQL Interface: SQL接口

  - 接收用户的SQL命令，并且返回用户需要查询的结果。比如SELECT ... FROM就是调用SQL Interface
  - MySQL支持DML（数据操作语言）、DDL（数据定义语言）、存储过程、视图、触发器、自定 义函数等多种SQL语言接口

- Parser: 解析器

  - 在解析器中对 SQL 语句进行语法分析、语义分析。将SQL语句分解成数据结构，并将这个结构 传递到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。如果在分解构成中遇到错 误，那么就说明这个SQL语句是不合理的。
  - 在SQL命令传递到解析器的时候会被解析器验证和解析，并为其创建 语法树 ，并根据数据字 典丰富查询语法树，会 验证该客户端是否具有执行该查询的权限 。创建好语法树后，MySQL还 会对SQl查询进行语法上的优化，进行查询重写。

- Optimizer: 查询优化器

  - SQL语句在语法解析之后、查询之前会使用查询优化器确定 SQL 语句的执行路径，生成一个 执行计划 。
  - 这个执行计划表明应该 使用哪些索引 进行查询（全表检索还是使用索引检索），表之间的连 接顺序如何，最后会按照执行计划中的步骤调用存储引擎提供的方法来真正的执行查询，并将 查询结果返回给用户。
  - 它使用“ 选取-投影-连接 ”策略进行查询。例如：

  ```
  SELECT id,name FROM student WHERE gender = '女';
  ```

  

  这个SELECT查询先根据WHERE语句进行 选取 ，而不是将表全部查询出来以后再进行gender过 滤。 这个SELECT查询先根据id和name进行属性 投影 ，而不是将属性全部取出以后再进行过 滤，将这两个查询条件 连接 起来生成最终查询结果。

- Caches & Buffers： 查询缓存组件

  - MySQL内部维持着一些Cache和Buffer，比如Query Cache用来缓存一条SELECT语句的执行结 果，如果能够在其中找到对应的查询结果，那么就不必再进行查询解析、优化和执行的整个过 程了，直接将结果反馈给客户端。
  - 这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等 。 这个查询缓存可以在 不同客户端之间共享 。
  - 从MySQL 5.7.20开始，不推荐使用查询缓存，并在 MySQL 8.0中删除 。

### 1.5 第三层：引擎层

插件式存储引擎层（ Storage Engines），**真正的负责了MySQL中数据的存储和提取，对物理服务器级别维护的底层数据执行操作**，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样 我们可以根据自己的实际需要进行选取。

MySQL 8.0.25默认支持的存储引擎如下：

[![image-20220615140556893](image/MySQL(2)架构篇.assets/image-20220615140556893.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615140556893.png)

### 1.6 存储层

所有的数据，数据库、表的定义，表的每一行的内容，索引，都是存在文件系统 上，以`文件`的方式存在的，并完成与存储引擎的交互。当然有些存储引擎比如InnoDB，也支持不使用文件系统直接管理裸设备，但现代文件系统的实现使得这样做没有必要了。在文件系统之下，可以使用本地磁盘，可以使用 DAS、NAS、SAN等各种存储系统。

### 1.7 小结

MySQL架构图本节开篇所示。下面为了熟悉SQL执行流程方便，我们可以简化如下：

[![image-20220615140710351](image/MySQL(2)架构篇.assets/image-20220615140710351.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615140710351.png)

简化为三层结构：

1. 连接层：客户端和服务器端建立连接，客户端发送 SQL 至服务器端；
2. SQL 层（服务层）：对 SQL 语句进行查询处理；与数据库文件的存储方式无关；
3. 存储引擎层：与数据库文件打交道，负责数据的存储和读取。



## 2. SQL执行流程

### 2.1 MySQL中的SQL执行流程

[![image-20220615141934531](image/MySQL(2)架构篇.assets/image-20220615141934531.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615141934531.png)

**MySQL的查询流程：**

#### 1) 查询缓存

- **(1) 查询缓存**：Server 如果在查询缓存中发现了这条 SQL 语句，就会直接将结果返回给客户端；如果没 有，就进入到解析器阶段。需要说明的是，因为查询缓存往往效率不高，所以在 MySQL8.0 之后就抛弃了这个功能。5.7中默认关闭

MySQL拿到一个查询请求后，会先到查询缓存看看，之前是不是执行过这条语句。`之前执行过的语句及其结果可能会以key-value对的形式，被直接缓存在内存中`。key是查询的语句，value是查询的结果。如果你的查询能够直接在这个缓存中找到key,那么这个value就会被直接返回给客户端。如果语句不在查询缓存中，就会继续后面的执行阶段。`执行完成后，执行结果会被存入查询缓存中`。所以，如果查询命中缓存，MySQL不需要执行后面的复杂操作，就可以直接返回结果，这个效率会很高。

**大多数情况查缓存就是个鸡肋，为什么呢？**

查询缓存是提前把查询结果缓存起来，这样下次不需要执行就可以直接拿到结果。需要说明的是，在MySQL中的
查询缓存，不是缓存查询计划，而是查询对应的结果。这就意味着查询匹配的`鲁棒性大大降低`，只有`相同的查询
操作才会命中查询缓存`。两个查询请求在任何字符上的不同**（例如：空格、注释、大小写）**，都会导致缓存不会命
中。因此MySQL的`查询缓存命中率不高`。

> 即需要sql语句完全一致才能命中缓存，有一点点区别都不行

同时，如果查询请求中包含某些系统函数、用户自定义变量和函数、一些系统表，如mysql、information_schema、performance_schema数据库中的表，那这个请求就不会被缓存。以某些系统函数举例，可能同样的函数的两次调用会产生不一样的结果，比如函数`NOW`,每次调用都会产生最新的当前时间，如果在一个查询请求中调用了这个函数，那即使查询请求的文本信息都一样，那不同时间的两次查询也应该得到不同的结果，如果在第一次查询时就缓存了，那第二次查询的时候直接使用第一次查询的结果就是错误的！

> 比如调用某些函数例如NOW()时使那么每次都查询语句都将不一样

此外，既然是缓存，那就有它`缓存失效的时候`。MySQL的缓存系统会监测涉及到的每张表，只要该表的结构或者数据被修改，如对该表使用了`INSERT`、`UPDATE`、`DELETE`、`TRUNCATE TABLE`、`ALTER TABLE`、`DROP TABLE`或`DROP DATABASE`语句，那使用该表的所有高速缓存查询都将变为无效并从高速缓存中删除！对于`更新压力大的数据库`来说，查询缓存的命中率会非常低。

> 当表中数据发生增删改时，可能会出现缓存失效

**总之，因为查询缓存往往弊大于利，查询缓存的失效非常频繁。**

**什么时候建议使用查询缓存呢？**

一般建议大家在静态表里使用查询缓存，什么叫`静态表`呢？就是一般我们极少更新的表。比如，一个系统配置表、字典表，这张表上的查询才适合使用查询缓存。好在MySQL也提供了这种“`按需使用`”的方式。你可以将 my.cnf 参数 query_cache_type 设置成 DEMAND，代表当 sql 语句中有 SQL_CACHE关键字时才缓存。比如：

```mysql
 query_cache_type 有3个值。 0代表关闭查询缓存OFF，1代表开启ON，2代表(DEMAND)
query_cache_type=2
```

若设置为query_cache_type=2：

这样对于默认的SQL语句都不使用查询缓存。而对于你确定要使用查询缓存的语句，可以供SQL_CACHE显示指定，像下面这个语句一样：

```
SELECT SQl_CACHE * FROM test WHERE ID=5;
```

SQl_NO_CACHE为不使用



查看当前 mysql 实例是否开启缓存机制

```
 MySQL5.7中：
show global variables like "%query_cache_type%";
```



监控查询缓存的命中率：

```
show status like '%Qcache%';
```



[![image-20220615144537260](image/MySQL(2)架构篇.assets/image-20220615144537260.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615144537260.png)

> 运行结果解析：
>
> `Qcache_free_blocks`: 表示查询缓存中海油多少剩余的blocks，如果该值显示较大，则说明查询缓存中的`内部碎片`过多了，可能在一定的时间进行整理。
>
> `Qcache_free_memory`: 查询缓存的内存大小，通过这个参数可以很清晰的知道当前系统的查询内存是否够用，DBA可以根据实际情况做出调整。
>
> `Qcache_hits`: 表示有 `多少次命中缓存`。我们主要可以通过该值来验证我们的查询缓存的效果。数字越大，缓存效果越理想。
>
> `Qcache_inserts`: 表示`多少次未命中然后插入`，意思是新来的SQL请求在缓存中未找到，不得不执行查询处理，执行查询处理后把结果insert到查询缓存中。这样的情况的次数越多，表示查询缓存应用到的比较少，效果也就不理想。当然系统刚启动后，查询缓存是空的，这也正常。
>
> `Qcache_lowmem_prunes`: 该参数记录有`多少条查询因为内存不足而被移除`出查询缓存。通过这个值，用户可以适当的调整缓存大小。
>
> `Qcache_not_cached`: 表示因为query_cache_type的设置而没有被缓存的查询数量。
>
> `Qcache_queries_in_cache`: 当前缓存中`缓存的查询数量`。
>
> `Qcache_total_blocks`: 当前缓存的block数量。

#### 2) 解析器

- **(2) 解析器**：在解析器中对 SQL 语句进行语法分析、语义分析。

[![image-20220615142301226](image/MySQL(2)架构篇.assets/image-20220615142301226.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615142301226.png)

**如果没有命中查询缓存，就要开始真正执行语句了**。首先，MySQL需要知道你要做什么，因此需要对SQL语句做解析。SQL语句的分析分为词法分析与语法分析。

分析器先做“ `词法分析` ”。你输入的是由多个字符串和空格组成的一条 SQL 语句，MySQL 需要识别出里面 的字符串分别是什么，代表什么。

MySQL 从你输入的"select"这个关键字识别出来，这是一个查询语 句。它也要把字符串“T”识别成“表名 T”，把字符串“ID”识别成“列 ID”。

接着，要做“ `语法分析` ”。根据词法分析的结果，语法分析器（比如：Bison）会根据语法规则，判断你输 入的这个 SQL 语句是否 `满足 MySQL 语法` 。

```mysql
select department_id,job_id, avg(salary) from employees group by department_id;
```

例如上一句，如果你的语法不对，将会放回错误提醒，



如果SQL语句正确，则会生成一个这样的语法树：

[![image-20220615162031427](image/MySQL(2)架构篇.assets/image-20220615162031427.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615162031427.png)

下图是SQL分词分析的过程步骤:

[![image-20220615163338495](image/MySQL(2)架构篇.assets/image-20220615163338495.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615163338495.png)

至此解析器的工作任务也基本圆满了。

#### 3) 优化器

- **(3) 优化器**：在优化器中会确定 SQL 语句的执行路径，比如是根据 `全表检索` ，还是根据 `索引检索` 等。

  经过解释器，MySQL就知道你要做什么了。在开始执行之前，还要先经过优化器的处理。**一条查询可以有很多种执行方式，最后都返回相同的结果。优化器的作用就是找到这其中最好的执行计划**。

  比如：优化器是在表里面有多个索引的时候，决定使用哪个索引；或者在一个语句有多表关联 (join) 的时候，决定各个表的连接顺序，还有表达式简化、子查询转为连接、外连接转为内连接等。

  举例：如下语句是执行两个表的 join：

```
select * from test1 join test2 using(ID)
where test1.name='zhangwei' and test2.name='mysql高级课程';
```



```
方案1：可以先从表 test1 里面取出 name='zhangwei'的记录的 ID 值，再根据 ID 值关联到表 test2，再判
断 test2 里面 name的值是否等于 'mysql高级课程'。

方案2：可以先从表 test2 里面取出 name='mysql高级课程' 的记录的 ID 值，再根据 ID 值关联到 test1，
再判断 test1 里面 name的值是否等于 zhangwei。

这两种执行方法的逻辑结果是一样的，但是执行的效率会有不同，而优化器的作用就是决定选择使用哪一个方案。优化
器阶段完成后，这个语句的执行方案就确定下来了，然后进入执行器阶段。
如果你还有一些疑问，比如优化器是怎么选择索引的，有没有可能选择错等。后面讲到索引我们再谈。
```



在查询优化器中，可以分为 `逻辑查询` 优化阶段和 `物理查询` 优化阶段。

逻辑查询优化就是通过改变SQL语句的内容来使得SQL查询更高效，同时为物理查询优化提供更多的候选执行计划。通常采用的方式是对SQL语句进行`等价变换`，对查询进行`重写`，而查询重写的数学基础就是关系代数。对条件表达式进行等价谓词重写、条件简化，对视图进行重写，对子查询进行优化，对连接语义进行了外连接消除、嵌套连接消除等。

物理查询优化是基于关系代数进行的查询重写，而关系代数的每一步都对应着物理计算，这些物理计算往往存在多种算法，因此需要计算各种物理路径的代价，从中选择代价最小的作为执行计划。在这个阶段里，对于单表和多表连接的操作，需要高效地`使用索引`，提升查询效率。

#### 4) 执行器

- **(4) 执行器**：

截止到现在，还没有真正去读写真实的表，仅仅只是产出了一个执行计划。于是就进入了执行器阶段 。

[![image-20220615162613806](image/MySQL(2)架构篇.assets/image-20220615162613806.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615162613806.png)

在执行之前需要判断该用户是否 `具备权限` 。如果没有，就会返回权限错误。如果具备权限，就执行 SQL 查询并返回结果。在 MySQL8.0 以下的版本，如果设置了查询缓存，这时会将查询结果进行缓存。

```
select * from test where id=1;
```



比如：表 test 中，ID 字段没有索引，那么执行器的执行流程是这样的：

```
调用 InnoDB 引擎接口取这个表的第一行，判断 ID 值是不是1，如果不是则跳过，如果是则将这行存在结果集中；
调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行。
执行器将上述遍历过程中所有满足条件的行组成的记录集作为结果集返回给客户端。
```

#### 小结

至此，这个语句就执行完成了。对于有索引的表，执行的逻辑也差不多。

SQL 语句在 MySQL 中的流程是： `SQL语句`→`查询缓存`→`解析器`→`优化器`→`执行器` 。

[![image-20220615164722975](image/MySQL(2)架构篇.assets/image-20220615164722975.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615164722975.png)

### 2.2 验证MySQL中SQL执行流程

#### 1) MySQL8.0中

1) 确认profiling是否开启

了解查询语句底层执行的过程：`select @profiling` 或者 `show variables like '%profiling'` 查看是否开启计划。开启它可以让MySQL收集在SQL

执行时所使用的资源情况，命令如下：

```
mysql> select @@profiling;
mysql> show variables like 'profiling';
```



profiling=0 代表关闭，我们需要把 profiling 打开，即设置为 1：

```
mysql> set profiling=1;
```



2) 多次执行相同SQL查询

然后我们执行一个 SQL 查询（你可以执行任何一个 SQL 查询）：

```
mysql> select * from employees;
```



3) 查看profiles

查看当前会话所产生的所有 profiles：

```
mysql> show profiles; # 显示最近的几次查询
```



4) 查看profile

显示执行计划，查看程序的执行步骤：

```
mysql> show profile;
```



[![image-20220615172149919](image/MySQL(2)架构篇.assets/image-20220615172149919.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615172149919.png)

当然你也可以查询指定的 Query ID，比如：

```
mysql> show profile for query 7;
```



查询 SQL 的执行时间结果和上面是一样的。

此外，还可以查询更丰富的内容：

```
mysql> show profile cpu,block io for query 6;
```



[![image-20220615172409967](image/MySQL(2)架构篇.assets/image-20220615172409967.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615172409967.png)

继续：

```
mysql> show profile cpu,block io for query 7;
```



[![image-20220615172438338](image/MySQL(2)架构篇.assets/image-20220615172438338.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615172438338.png)

1、除了查看cpu、io阻塞等参数情况，还可以查询下列参数的利用情况。

```
Syntax:
SHOW PROFILE [type [, type] ... ]
	[FOR QUERY n]
	[LIMIT row_count [OFFSET offset]]

type: {
	| ALL -- 显示所有参数的开销信息
	| BLOCK IO -- 显示IO的相关开销
	| CONTEXT SWITCHES -- 上下文切换相关开销
	| CPU -- 显示CPU相关开销信息
	| IPC -- 显示发送和接收相关开销信息
	| MEMORY -- 显示内存相关开销信息
	| PAGE FAULTS -- 显示页面错误相关开销信息
	| SOURCE -- 显示和Source_function,Source_file,Source_line 相关的开销信息
	| SWAPS -- 显示交换次数相关的开销信息
}
```



2、发现两次查询当前情况都一致，说明没有缓存。

`在 8.0 版本之后，MySQL 不再支持缓存的查询`。一旦数据表有更新，缓存都将清空，因此只有数据表是静态的时候，或者数据表很少发生变化时，使用缓存查询才有价值，否则如果数据表经常更新，反而增加了 SQL 的查询时间。

#### 2) MySQL5.7中

上述操作在MySQL5.7中测试，发现前后两次相同的sql语句，执行的查询过程仍然是相同的。不是会使用 缓存吗？这里我们需要 显式开启查询缓存模式 。在MySQL5.7中如下设置：

1) 配置文件中开启查询缓存

在 /etc/my.cnf 中新增一行：

```
query_cache_type=1
```



2) 重启mysql服务

```
systemctl restart mysqld
```



3) 开启查询执行计划

由于重启过服务，需要重新执行如下指令，开启profiling。

```
mysql> set profiling=1;
```



4) 执行语句两次：

```
mysql> select * from locations;
```



5) 查看profiles

[![image-20220615173727345](image/MySQL(2)架构篇.assets/image-20220615173727345.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615173727345.png)

6) 查看profile

显示执行计划，查看程序的执行步骤：

```
mysql> show profile for query 1;
```



[![image-20220615173803835](image/MySQL(2)架构篇.assets/image-20220615173803835.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615173803835.png)

```
mysql> show profile for query 2;
```



[![image-20220615173822079](image/MySQL(2)架构篇.assets/image-20220615173822079.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615173822079.png)

结论不言而喻。执行编号2时，比执行编号1时少了很多信息，从截图中可以看出查询语句直接从缓存中 获取数据。



### 2.3 SQL语法顺序

随着Mysql版本的更新换代，其优化器也在不断的升级，优化器会分析不同执行顺序产生的性能消耗不同 而动态调整执行顺序。

![image-20230816223305552](image/MySQL(2)架构篇.assets/image-20230816223305552.png)



## 3. 数据库缓冲池（buffer pool）

`InnoDB` 存储引擎是**以页为单位**来管理存储空间的，我们进行的增删改查操作其实本质上都是在访问页面（包括读页面、写页面、创建新页面等操作）。而磁盘 I/O 需要消耗的时间很多，而在内存中进行操作，效率则会高很多，为了能让数据表或者索引中的数据随时被我们所用，DBMS 会申请`占用内存来作为数据缓冲池` ，**在真正访问页面之前，需要把在磁盘上的页缓存到内存中的 Buffer Pool 之后才可以访问。**

这样做的好处是可以让磁盘活动最小化，从而 `减少与磁盘直接进行 I/O 的时间 `。要知道，这种策略对提升 SQL 语句的查询性能来说至关重要。如果索引的数据在缓冲池里，那么访问的成本就会降低很多。

> MySQL数据页的大小默认是16KB，操作系统的默认大小是4KB

### 3.1 缓冲池 vs 查询缓存

缓冲池和查询缓存是一个东西吗？不是。

#### 1) 缓冲池（Buffer Pool）

首先我们需要了解在 InnoDB 存储引擎中，缓冲池都包括了哪些。

在 InnoDB 存储引擎中有一部分数据会放到内存中，缓冲池则占了这部分内存的大部分，它用来存储各种数据的缓存，如下图所示：

[![image-20220615175309751](image/MySQL(2)架构篇.assets/image-20220615175309751.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615175309751.png)

从图中，你能看到 InnoDB 缓冲池包括了数据页、索引页、插入缓冲、锁信息、自适应 Hash 和数据字典信息等。

**缓存池的重要性：**

InnoDB存储引擎在处理客户端的请求时，当需要访问某个页的数据时，就会`把完整的页的数据全部加载到内存`中，也就是说即使我们只需要访问一个页的一条记录，那也需要先把整个页的数据加载到内存中。将整个页加载到内存中后就可以进行读写访问了，在进行完读写访问之后并不着急把该页对应的内存空间释放掉，而是将其`缓存`起来，这样将来有请求再次访问该页面时，就可以`省去磁盘IO`的开销了。

**缓存原则：**

“ `位置 * 频次` ”这个原则，可以帮我们对 I/O 访问效率进行优化。

首先，位置决定效率，提供缓冲池就是为了在内存中可以直接访问数据。

其次，频次决定优先级顺序。因为缓冲池的大小是有限的，比如磁盘有 200G，但是内存只有 16G，缓冲池大小只有 1G，就无法将所有数据都加载到缓冲池里，这时就涉及到优先级顺序，会`优先对使用频次高的热数据进行加载 `。

**缓冲池的预读特性:**

缓冲池的作用就是提升 I/O 效率，而我们进行读取数据的时候存在一个“局部性原理”，也就是说我们使用了一些数据，`大概率还会使用它周围的一些数据`，因此采用“预读”的机制提前加载，可以减少未来可能的磁盘 I/O 操作。

#### 2) 查询缓存

那么什么是查询缓存呢？

查询缓存是提前把 查询结果缓存起来，这样下次不需要执行就可以直接拿到结果。需要说明的是，在 MySQL 中的查询缓存，不是缓存查询计划，而是查询对应的结果。因为命中条件苛刻，而且只要数据表 发生变化，查询缓存就会失效，因此命中率低。



### 3.2 缓冲池如何读取数据

缓冲池管理器会尽量将经常使用的数据保存起来，在数据库进行页面读操作的时候，首先会判断该页面 是否在缓冲池中，如果存在就直接读取，如果不存在，就会通过内存或磁盘将页面存放到缓冲池中再进行读取。

缓存在数据库中的结构和作用如下图所示：

[![image-20220615193131719](image/MySQL(2)架构篇.assets/image-20220615193131719.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615193131719.png)

**如果我们执行 SQL 语句的时候更新了缓存池中的数据，那么这些数据会马上同步到磁盘上吗？**

实际上，当我们对数据库中的记录进行修改的时候，首先会修改缓冲池中页里面的记录信息，然后数据库会`以一定的频率刷新`到磁盘中。注意**并不是每次发生更新操作，都会立即进行磁盘回写**。缓冲池会采用一种叫做 `checkpoint 的机制` 将数据回写到磁盘上，这样做的好处就是提升了数据库的整体性能。

比如，当`缓冲池不够用`时，需要释放掉一些不常用的页，此时就可以`强行采用checkpoint的方式`，将不常用的脏页回写到磁盘上，然后再从缓存池中将这些页释放掉。这里的脏页 (dirty page) 指的是缓冲池中被修改过的页，与磁盘上的数据页不一致。



### 3.3 查看/设置缓冲池的大小

如果你使用的是 MySQL MyISAM 存储引擎，它只缓存索引，不缓存数据，对应的键缓存参数为`key_buffer_size`，你可以用它进行查看。

如果你使用的是 InnoDB 存储引擎，可以通过查看 innodb_buffer_pool_size 变量来查看缓冲池的大小。命令如下：

```
show variables like 'innodb_buffer_pool_size';
```



[![image-20220615214847480](image/MySQL(2)架构篇.assets/image-20220615214847480.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615214847480.png)

你能看到此时 InnoDB 的缓冲池大小只有 134217728/1024/1024=128MB。我们可以修改缓冲池大小，比如改为256MB，方法如下：

```
set global innodb_buffer_pool_size = 268435456;
```



或者：

```
[server]
innodb_buffer_pool_size = 268435456
```



### 3.4 多个Buffer Pool实例

Buffer Poolz本质是InnoDB向操作系统申请的一块`连续的内存空间`，在多线程环境下，方问Buffer Pool中的数据都需要`加锁`处理。在Buffer Pool特别大而且**多线程并发访问特别高**的情况下，单一的Buffer Pool可能会影响请求的处理速度。所以在Buffer Pool特别大的时候，我们可以把它们`拆分成若干个小的Buffer Pool`,每个BufferPool都称为一个实例，它们都是独立的，独立的去申请内存空间，独立的管理各种链表。**所以在多线程并发访问时并不会相互影响，从而提高并发处理能力。**

我们可以在服务器启动的时候通过设置innodb_buffer_pool_instances的值来修改Buffer Pool3实例的个数，比方说这样：

```
[server]
innodb_buffer_pool_instances = 2
```

这样就表明我们要创建2个 `Buffer Pool` 实例。

我们看下如何查看缓冲池的个数，使用命令：

```
show variables like 'innodb_buffer_pool_instances';
```

**默认情况下缓冲池的个数为1**



那每个 `Buffer Pool` 实例实际占多少内存空间呢？其实使用这个公式算出来的：

```
innodb_buffer_pool_size/innodb_buffer_pool_instances
```



也就是总共的大小除以实例的个数，结果就是每个 `Buffer Pool` 实例占用的大小。

不过也不是说 Buffer Pool 实例创建的越多越好，分别`管理各个 Buffer Pool 也是需要性能开销`的，InnDB规定：**当innodb_buffer_pool_size的值小于1G的时候设置多个实例是无效的**，InnoDB会默认把innodb_buffer_pool_instances的值修改为1。而我们鼓励在 Buffer Pool 大于等于 1G 的时候设置多个 Buffer Pool 实例。



### 3.5 引申问题

Buffer Pool是MySQL内存结构中十分核心的一个组成，你可以先把它想象成一个黑盒子。

**黑盒下的更新数据流程**

当我们查询数据的时候，会先去 Buffer Pool 中查询。如果 Buffer Pool 中不存在，存储引擎会先将数据从磁盘加载到 Buffer Pool 中，然后将数据返回给客户端；同理，当我们更新某个数据的时候，如果这个数据不存在于 Buffer Pool，同样会先数据加载进来，然后修改内存的数据。被修改的数据会在之后统一刷入磁盘。

[![image-20220615222455867](image/MySQL(2)架构篇.assets/image-20220615222455867.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615222455867.png)

我更新到一半突然发生错误了，想要回滚到更新之前的版本，该怎么办？连数据持久化的保证、事务回滚都做不到还谈什么崩溃恢复？

答案：**Redo Log** & **Undo Log**

# 存储引擎概述

- 为了管理方便，人们把`连接管理`、`查询缓存`、`语法解析`、`查询优化`这些并不涉及真实数据存储的功能划分为MySQL server的功能，
- 把真实存取数据的功能划分为`存储引擎`的功能。所以在`MySQL server`完成了查询优化后，只需按照生成的`执行计划`调用底层存储引擎提供的API,获取到数据后返回给客户端就好了。
- MySQL中提到了存储引擎的概念。简而言之，`存储引擎就是指表的类型`。其实存储引擎以前叫做**`表处理器`**，后来改名为`存储引擎`，它的功能就是接收上层传下来的指令，然后对表中的数据进行提取或写入操作。

## 1. 查看存储引擎

- 查看mysql提供什么存储引擎

```
show engines;
```

![image-20220615223831995](image/MySQL(2)架构篇.assets/image-20220615223831995.png)

> 只有InnoDB支持事务、分布式事务（XA）、savepoints

## 2. 设置系统默认的存储引擎

- 查看默认的存储引擎

```
show variables like '%storage_engine%';
或
SELECT @@default_storage_engine;
```



[![image-20220615224249491](image/MySQL(2)架构篇.assets/image-20220615224249491.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220615224249491.png)

- 修改默认的存储引擎

如果在创建表的语句中没有显式指定表的存储引擎的话，那就会默认使用 InnoDB 作为表的存储引擎。 如果我们想改变表的默认存储引擎的话，可以这样写启动服务器的命令行：

```
SET DEFAULT_STORAGE_ENGINE=MyISAM;
```



或者修改 my.cnf 文件：

```
default-storage-engine=MyISAM
 重启服务
systemctl restart mysqld.service
```



## 3. 设置表的存储引擎

存储引擎是负责对表中的数据进行提取和写入工作的，我们可以为 不同的表设置不同的存储引擎 ，也就是 说不同的表可以有不同的物理存储结构，不同的提取和写入方式。

### 3.1 创建表时指定存储引擎

我们之前创建表的语句都没有指定表的存储引擎，那就会使用默认的存储引擎 InnoDB 。如果我们想显 式的指定一下表的存储引擎，那可以这么写：

```
CREATE TABLE 表名(
建表语句;
) ENGINE = 存储引擎名称;
```



### 3.2 修改表的存储引擎

如果表已经建好了，我们也可以使用下边这个语句来修改表的存储引擎：

```
ALTER TABLE 表名 ENGINE = 存储引擎名称;
```



比如我们修改一下 engine_demo_table 表的存储引擎：

```mysql
ALTER TABLE engine_demo_table ENGINE = InnoDB;
```



这时我们再查看一下 engine_demo_table 的表结构：

```
mysql> SHOW CREATE TABLE engine_demo_table\G
*************************** 1. row ***************************
Table: engine_demo_table
Create Table: CREATE TABLE `engine_demo_table` (
`i` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.01 sec)
```



## 4. 引擎介绍

### 4.1 InnoDB 引擎：具备外键支持功能的事务存储引擎

- MySQL从3.23.34a开始就包含InnoDB存储引擎。 `大于等于5.5之后，默认采用InnoDB引擎` 。

- InnoDB是MySQL的 `默认事务型引擎` ，它被设计用来处理大量的短期(short-lived)事务。可以确保事务的完整提交(Commit)和回滚(Rollback)。

- 除了增加和查询外，还需要更新、删除操作，那么，应优先选择InnoDB存储引擎。 

- **除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑InnoDB引擎。**

- 数据文件结构：（在《第02章_MySQL数据目录》章节已讲）
  - 表名.frm 存储表结构（MySQL8.0时，合并在表名.ibd中）
  - 表名.ibd 存储数据和索引
  
- InnoDB是 `为处理巨大数据量的最大性能设计` 。
  - 在以前的版本中，字典数据以元数据文件、非事务表等来存储。现在这些元数据文件被删除了。
  
    比如： .frm ， .par ， .trn ， .isl ， .db.opt 等都在MySQL8.0中不存在了。
  
- 对比MyISAM的存储引擎， `InnoDB写的处理效率差一些` ，并且会占用更多的磁盘空间以保存数据和索引。

- MyISAM只缓存索引，不缓存真实数据；InnoDB不仅缓存索引还要缓存真实数据， `对内存要求较高` ，而且内存大小对性能有决定性的影响。

- `支持行锁、事务、外键`

### 4.2 MyISAM 引擎：主要的非事务处理存储引擎

- MyISAM提供了大量的特性，包括全文索引、压缩、空间函数(GIS)等，但MyISAM`不支持事务、行级锁、外键` ，有一个毫无疑问的缺陷就是`崩溃后无法安全恢复 `。
- `5.5之前默认的存储引擎`
- 对事务完整性没有要求或者以`SELECT、INSERT`为主的应用，优势是访问的`速度快`
- **针对数据统计有额外的常数存储。故而 count(*) 的查询效率很高** 数据文件结构：（在《第02章_MySQL数据目录》章节已讲）
  - 表名.frm(8.0后.sdi) 存储表结构
  - 表名.MYD 存储数据 (MYData)
  - 表名.MYI 存储索引 (MYIndex)
- 应用场景：只读应用或者以读为主的业务

### 4.3 Archive 引擎：用于数据存档

- `archive`是`归档`的意思，仅仅支持`插入`和`查询`两种功能（行插入后不能被修改）。
- 在MySQL5.5以后`支持索引`功能
- 拥有很好的压缩机制，使用`zlib压缩库`，在记录请求的时候实时的进行压缩，经常被用来作为仓库使用。
- 创建ARCHIVE表时，存储引擎会创建名称以表名开头的文件。数据文件的扩展名为`.ARZ`。
- 根据英文的测试结论来看，同样数据量下，`Archive表比MyISAM表要小大约75%，比支持事务处理的InnoDB表小大约83%。`
- ARCHIVE:存储引擎采用了`行级锁`。该ARCHIVE引擎支持`AUTO_INCREMENT`列属性。AUTO_INCREMENT列可以具有唯一索引或非唯一索引。尝试在任何其他列上创建索引会导致错误。
- Archive表`适合日志和数据采集（档案）`类应用；**适合存储大量的独立的作为历史记录的数据**。拥有`很高的插入
  速度`，但是对查询的支持较差。
- **下表展示了ARCHIVE存储号引擎功能**

[![image-20220616124743732](image/MySQL(2)架构篇.assets/image-20220616124743732.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220616124743732.png)

### 4.4 Blackhole 引擎：丢弃写操作，读操作会返回空内容

- Blackhole引擎没有实现任何存储机制，它会`丢弃所有插入的数据`，不做任何保存。
- 但服务器会记录Blackhole表的日志，所以可以用于复制数据到备库，或者简单地记录到目志。但这种应用方式会碰到很多问题，**因此并不推荐。**

### 4.5 CSV 引擎：存储数据时，以逗号分隔各个数据项

- CSV引擎可以将`普通的CSV文件作为MySQL的表来处理`，但不支持索引。
- CSV引擎可以作为一种`数据交换的机制`，非常有用。
- CSV存储的数据直接可以在操作系统里，用文本编辑器，或者exceli读取。
- 对于数据的快速导入、导出是有明显优势的。

创建CSV表时，服务器会创建一个纯文本数据文件，其名称以表名开头并带有`.CSV`扩展名。当你将数据存储到表中时，存储引擎将其以逗号分隔值格式保存到数据文件中。

使用案例如下

```
mysql> CREATE TABLE test (i INT NOT NULL, c CHAR(10) NOT NULL) ENGINE = CSV;
Query OK, 0 rows affected (0.06 sec)
mysql> INSERT INTO test VALUES(1,'record one'),(2,'record two');
Query OK, 2 rows affected (0.05 sec)
Records: 2 Duplicates: 0 Warnings: 0
mysql> SELECT * FROM test;
+---+------------+
| i |      c     |
+---+------------+
| 1 | record one |
| 2 | record two |
+---+------------+
2 rows in set (0.00 sec)
```



创建CSV表还会创建相应的元文件 ，用于 存储表的状态 和 表中存在的行数 。此文件的名称与表的名称相 同，后缀为 CSM 。如图所示

[![image-20220616125342599](image/MySQL(2)架构篇.assets/image-20220616125342599.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220616125342599.png)

如果检查 test.CSV 通过执行上述语句创建的数据库目录中的文件，其内容使用Notepad++打开如下：

```
"1","record one"
"2","record two"
```



这种格式可以被 Microsoft Excel 等电子表格应用程序读取，甚至写入。使用Microsoft Excel打开如图所示

[![image-20220616125448555](image/MySQL(2)架构篇.assets/image-20220616125448555.png)](https://github.com/codinglin/StudyNotes/blob/main/MySQL高级篇/MySQL架构篇.assets/image-20220616125448555.png)



### 4.6 Memory 引擎：置于内存的表

**概述：**

Memory采用的逻辑介质是`内存 ，响应速度很快` ，但是当mysqld守护进程崩溃的时候`数据会丢失` 。另外，要求存储的数据是数据长度不变的格式，比如，Blob和Text类型的数据不可用(长度不固定的)。

**主要特征：**

- Memory同时 `支持哈希（HASH）索引` 和 `B+树索引` 。
  - 哈希索引相等的比较快，但是对于范围的比较慢很多。
  - `默认使用哈希(HASH)索引`，其速度要比使用B型树(BTREE)索引快。
  - 如果希望使用B树索引，可以在创建索引时选择使用。

- Memory表至少比MyISAM表要`快一个数量级` 。
- MEMORY `表的大小是受到限制` 的。表的大小主要取决于两个参数，分别是 `max_rows` 和 `max_heap_table_size` 。其中，max_rows可以在创建表时指定；max_heap_table_size的大小默 认为16MB，可以按需要进行扩大。
- 数据文件与索引文件分开存储。
  - 每个基于MEMORY存储引擎的表实际对应一个磁盘文件，该文件的文件名与表名相同，类型为`frm类型`，该文件中只存储表的结构，而其`数据文件都是存储在内存中的`。
  - 这样有利于数据的快速处理，提供整个表的处理效率。

- 缺点：其数据易丢失，生命周期短。基于这个缺陷，选择MEMORY存储引擎时需要特别小心。

**使用Memory存储引擎的场景：**

1. `目标数据比较小` ，而且非常`频繁的进行访问` ，在内存中存放数据，如果太大的数据会造成`内存溢出` 。可以通过参数` max_heap_table_size` 控制Memory表的大小，限制Memory表的最大的大小。
2. 如果`数据是临时的` ，而且`必须立即可用`得到，那么就可以放在内存中。
3. 存储在Memory表中的数据如果突然间`丢失的话也没有太大的关系` 。



### 4.7 Federated 引擎：访问远程表

- Federated引擎是访问其他MySQL服务器的一个 `代理` ，尽管该引擎看起来提供了一种很好的 `跨服务 器的灵活性` ，但也经常带来问题，因此 `默认是禁用的` 。


### 4.8 Merge引擎：管理多个MyISAM表构成的表集合

### 4.9 NDB引擎：MySQL集群专用存储引擎

也叫做 NDB Cluster 存储引擎，主要用于 `MySQL Cluster 分布式集群` 环境，类似于 Oracle 的 RAC 集 群。

### 4.10 引擎对比

MySQL中同一个数据库，不同的表可以选择不同的存储引擎。如下表对常用存储引擎做出了对比。

![image-20220616125928861](image/MySQL(2)架构篇.assets/image-20220616125928861.png)]

![image-20220616125945304](image/MySQL(2)架构篇.assets/image-20220616125945304.png)]

其实这些东西大家没必要立即就给记住，列出来的目的就是想让大家明白不同的存储引擎支持不同的功能。

其实我们最常用的就是 InnoDB 和 MyISAM ，有时会提一下 Memory 。其中 InnoDB 是 MySQL 默认的存储引擎。

## 5. MyISAM和InnoDB对比汇总

很多人对 InnoDB 和 MyISAM 的取舍存在疑问，到底选择哪个比较好呢？	

MySQL5.5之前的默认存储引擎是MyISAM，5.5之后改为了InnoDB。

- **首先对于InnoDB存储引擎**，提供了良好的事务管理、崩溃修复能力和并发控制。因为InnoDB存储引擎`支持事务`，所以对于要求事务完整性的场合需要选择InnoDB,比如数据操作除了插入和查询以外还包，含有很多更新、删除操作，像财务系统等对数据准确性要求较高的系统。缺点是其`读写效率稍差`，`占用的数据空间相对比较大`。
- **其次对于MyISAM存储引擎**，如果是`小型应用`，系统以`读操作和插入操作为主`，只有很少的更新、删除操作，并且对事务的要求没有那么高，则可以选择这个存储引擎，且有先天的`count(*)优势`。MyISAM存储引擎的优势在于`占用空间小`，`处理速度快`；缺点是`不支持事务`的完整性和并发性。

这两种引擎各有特点，当然你也可以在MySQL中，针对不同的数据表，可以选择不同的存储引擎。

| 对比项         | MyISAM                                                   | InnoDB                                                       |
| :------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 外键           | 不支持                                                   | 支持                                                         |
| 事务           | 不支持                                                   | 支持                                                         |
| 行表锁         | 表锁，及时操作一条记录也会锁住整个表，不适合高并发的操作 | 行锁，操作时只锁某一行，不对其它行有影响，适合高并发的操作   |
| 缓存           | 只缓存索引，不缓存真实数据                               | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 自带系统表使用 | Y                                                        | N                                                            |
| 关注点         | 性能：节省资源、消耗少、简单业务                         | 事务：并发写、事务、更大资源                                 |
| 默认安装       | Y                                                        | Y                                                            |
| 默认使用       | N                                                        | Y                                                            |









































































































