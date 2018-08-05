# 数据库整理

1、mysql如何做分页

mysql数据库做分页用limit关键字，它后面跟两个参数startIndex和pageSize


2、mysql引擎有哪些，各自的特点是什么？

http://www.cnblogs.com/ctztake/p/8453990.html


3、数据库怎么建立索引

create index account_index on `table name `(`字段名`(length)


4、一张表多个字段，怎么创建组合索引

create index account_index on `table name `(`字段名`，'字段名')


5、如何应对数据的高并发，大量的数据计算

1.创建索引
2.数据库读写分离，两个数据库，一个作为写，一个作为读
3. 外键去掉


4.django中orm表性能相关的


select_related：

对于一对一字段（OneToOneField）和外键字段（ForeignKey），可以使用select_related 来对QuerySet进行优化。

复制代码
select_related主要针一对一和多对一关系进行优化。
select_related使用SQL的JOIN语句进行优化，通过减少SQL查询的次数来进行优化、提高性能。
可以通过可变长参数指定需要select_related的字段名。也可以通过使用双下划线“__”连接字段名来实现指定的递归查询。
没有指定的字段不会缓存，没有指定的深度不会缓存，如果要访问的话Django会再次进行SQL查询。
也可以通过depth参数指定递归的深度，Django会自动缓存指定深度内所有的字段。如果要访问指定深度外的字段，Django会再次进行SQL查询。
也接受无参数的调用，Django会尽可能深的递归查询所有的字段。但注意有Django递归的限制和性能的浪费。
Django >= 1.7，链式调用的select_related相当于使用可变长参数。Django < 1.7，链式调用会导致前边的select_related失效，只保留最后一个。
复制代码
 

prefetch_related：

对于多对多字段（ManyToManyField）和一对多字段，可以使用prefetch_related()来进行优化。

复制代码
prefetch_related()和select_related()的设计目的很相似，都是为了减少SQL查询的数量，但是实现的方式不一样。后者是通过JOIN语句，在SQL查询内解决问题。
但是对于多对多关系，使用SQL语句解决就显得有些不太明智，因为JOIN得到的表将会很长，会导致SQL语句运行时间的增加和内存占用的增加。若有n个对象，每个对象的多对多字段对应Mi条
，就会生成Σ(n)Mi 行的结果表。

prefetch_related()的解决方法是，分别查询每个表，然后用Python处理他们之间的关系。
复制代码
复制代码
因为select_related()总是在单次SQL查询中解决问题，而prefetch_related()会对每个相关表进行SQL查询，因此select_related()的效率通常比后者高。
鉴于第一条，尽可能的用select_related()解决问题。只有在select_related()不能解决问题的时候再去想prefetch_related()。
你可以在一个QuerySet中同时使用select_related()和prefetch_related()，从而减少SQL查询的次数。
只有prefetch_related()之前的select_related()是有效的，之后的将会被无视掉
复制代码
6、数据库内连表、左连表、右连表

内连接是根据某个条件连接两个表共有的数据
左连接是根据某个条件以及左边的表连接数据，右边的表没有数据的话则为null
右连接是根据某个条件以及右边的表连接数据，左边的表没有数据的话则为null

7、视图和表的区别

视图是已经编译好的sql语句，是基于sql语句的结果集的可视化的表，而表不是
视图是窗口，表示内容
视图没有实际的物理记录，而表有

视图是虚表,表是实表

视图的建立和删除只影响视图本身，不影响对应的表

8、关系型数据库的特点

数据集中控制
数据独立性高
数据共享性好
数据冗余度小
数据结构化
统一的数据保护能力

 

9、mysql数据库都有哪些索引

普通索引：普通索引仅有一个功能：加速查找
唯一索引：唯一索引两个功能：加速查找和唯一约束（可含null）
外键索引：外键索引两个功能：加速查找和唯一约束（不可为null）
联合索引：联合索引是将n个列组合成一个索引，应用场景：同时使用n列来进行查询

 

触发器

使用触发器可以定制用户对表进行【增、删、改】操作时前后的行为，注意：没有查询


复制代码
# 插入前
CREATE TRIGGER tri_before_insert_tb1 BEFORE INSERT ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 插入后
CREATE TRIGGER tri_after_insert_tb1 AFTER INSERT ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 删除前
CREATE TRIGGER tri_before_delete_tb1 BEFORE DELETE ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 删除后
CREATE TRIGGER tri_after_delete_tb1 AFTER DELETE ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 更新前
CREATE TRIGGER tri_before_update_tb1 BEFORE UPDATE ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 更新后
CREATE TRIGGER tri_after_update_tb1 AFTER UPDATE ON tb1 FOR EACH ROW
BEGIN
    ...
END
复制代码

复制代码
#准备表
CREATE TABLE cmd (
    id INT PRIMARY KEY auto_increment,
    USER CHAR (32),
    priv CHAR (10),
    cmd CHAR (64),
    sub_time datetime, #提交时间
    success enum ('yes', 'no') #0代表执行失败
);

CREATE TABLE errlog (
    id INT PRIMARY KEY auto_increment,
    err_cmd CHAR (64),
    err_time datetime
);

#创建触发器
delimiter //
CREATE TRIGGER tri_after_insert_cmd AFTER INSERT ON cmd FOR EACH ROW
BEGIN
    IF NEW.success = 'no' THEN #等值判断只有一个等号
            INSERT INTO errlog(err_cmd, err_time) VALUES(NEW.cmd, NEW.sub_time) ; #必须加分号
      END IF ; #必须加分号
END//
delimiter ;


#往表cmd中插入记录，触发触发器，根据IF的条件决定是否插入错误日志
INSERT INTO cmd (
    USER,
    priv,
    cmd,
    sub_time,
    success
)
VALUES
    ('egon','0755','ls -l /etc',NOW(),'yes'),
    ('egon','0755','cat /etc/passwd',NOW(),'no'),
    ('egon','0755','useradd xxx',NOW(),'no'),
    ('egon','0755','ps aux',NOW(),'yes');


#查询错误日志，发现有两条
mysql> select * from errlog;
+----+-----------------+---------------------+
| id | err_cmd         | err_time            |
+----+-----------------+---------------------+
|  1 | cat /etc/passwd | 2017-09-14 22:18:48 |
|  2 | useradd xxx     | 2017-09-14 22:18:48 |
+----+-----------------+---------------------+
rows in set (0.00 sec)

插入后触发触发器
复制代码
特别的：NEW表示即将插入的数据行，OLD表示即将删除的数据行。

二 使用触发器

触发器无法由用户直接调用，而知由于对表的【增/删/改】操作被动引发的。

三 删除触发器

drop trigger tri_after_insert_cmd;
10、存储过程

http://www.cnblogs.com/ctztake/p/7544559.html

存储过程不允许执行return语句，但是可以通过out参数返回多个值，存储过程一般是作为一个独立的部分来执行，存储过程是一个预编译的SQL语句。

11、sql优化：

选取最适用的字段属性

使用连接（JOIN）来代替子查询(Sub-Queries)

select句中避免使用 '*'

减少访问数据库的次数
删除重复记录
用where子句替代having子句
减少对表的查询
explain 

12、char和vachar区别：

char是固定长度，存储需要空间12个字节，处理速度比vachar快，费内存空间，当存储的值没有达到指定的范围时，会用空格替代
vachar是不固定长度，需要存储空间13个字节，节约存储空间，存储的是真实的值，会在存储的值前面加上1-2个字节，用来表示真实数据的大小

13、Mechached与redis

mechached：只支持字符串，不能持久化，数据仅存在内存中，宕机或重启数据将全部失效
不能进行分布式扩展，文件无法异步法。
优点：mechached进程运行之后，会预申请一块较大的内存空间，自己进行管理。
redis：支持服务器端的数据类型，redis与memcached相比来说，拥有更多的数据结构和并发支持更丰富的数据操作，可持久化。
五大类型数据：string、hash、list、set和有序集合，redis是单进程单线程的。
缺点：数据库的容量受到物理内存的限制。

14、sql注入

sql注入是比较常见的攻击方式之一，针对编程员编程的疏忽，通过sql语句，实现账号无法登陆，甚至篡改数据库。
防止：凡涉及到执行sql中有变量时，切记不要用拼接字符串的方法


16、游标是什么？

是对查询出来的结果集作为一个单元来有效的处理，游标可以定在该单元中的特定行，从结果集的当前行检索一行或多行，可以对结果集当前行做修改，
一般不使用游标，但是需要逐条处理数据的时候，游标显得十分重要

17、 数据库支持多有标准的SQL数据类型，重要分为三类

数值类型（tinyint，int，bigint，浮点数，bit）
字符串类型（char和vachar，enum，text，set）
日期类型（date，datetime，timestamp）

枚举类型与集合类型

18、mysql慢查询

慢查询对于跟踪有问题的查询很有用，可以分析出当前程序里哪些sql语句比较耗费资源
慢查询定义：
指mysql记录所有执行超过long_query_time参数设定的时间值的sql语句，慢查询日志就是记录这些sql的日志。
mysql在windows系统中的配置文件一般是my.ini找到mysqld
log-slow-queries = F:\MySQL\log\mysqlslowquery.log 为慢查询日志存放的位置，一般要有可写权限
long_query_time = 2 2表示查询超过两秒才记录

19、memcached命中率

命中：可以直接通过缓存获取到需要的数据
不命中：无法直接通过缓存获取到想要的数据，需要再次查询数据库或者执行其他的操作，原因可能是由于缓存中根本不存在，或者缓存已经过期
缓存的命中率越高则表示使用缓存的收益越高，应额用的性能越好，抗病发能力越强
运行state命令可以查看memcached服务的状态信息，其中cmd—get表示总的get次数，get—hits表示命中次数，命中率=get—hits / cmd—get

20、Oracle和MySQL该如何选择，为什么？

他们都有各自的优点和缺点。考虑到时间因素，我倾向于MySQL
选择MySQL而不选Oracle的原因
MySQL开源
MySQL轻便快捷
MySQL对命令行和图形界面的支持都很好
MySQL支持通过Query Browser进行管理

21、什么情况下适合建立索引？

1.为经常出现在关键字order by、group by、distinct后面的字段，建立索引
2.在union等集合操作的结果集字段上，建立索引，其建立索引的目的同上
3.为经常用作查询选择的字段，建立索引
4.在经常用作表连接的属性上，建立索引

22、数据库底层是用什么结构实现的，你大致画一下：

底层用B+数实现，结构图参考： 
http://blog.csdn.net/cjfeii/article/details/10858721
http://blog.csdn.net/tonyxf121/article/details/8393545

23、sql语句应该考虑哪些安全性？

1.防止sql注入，对特殊字符进行转义，过滤或者使用预编译的sql语句绑定变量
2.最小权限原则，特别是不要用root账户，为不同的类型的动作或者组建使用不同的账户
3.当sql运行出错时，不要把数据库返回的错误信息全部显示给用户，以防止泄漏服务器和数据库相关信息

24、数据库事物有哪几种？

隔离性、持续性、一致性、原子性

25、MySQ数据表在什么情况下容易损坏？

服务器突然断电导致数据文件损坏
强制关机，没有先关闭mysq服务器等

26、drop、delete与truncate的区别

当表被TRUNCATE 后，这个表和索引所占用的空间会恢复到初始大小，

DELETE操作不会减少表或索引所占用的空间。

 drop语句将表所占用的空间全释放掉。

复制代码
一、delete

1、delete是DML，执行delete操作时，每次从表中删除一行，并且同时将该行的的删除操作记录在redo和undo表空间中以便进行回滚（rollback）和重做操作，但要注意表空间要足够大，
需要手动提交（commit）操作才能生效，可以通过rollback撤消操作。

2、delete可根据条件删除表中满足条件的数据，如果不指定where子句，那么删除表中所有记录。

3、delete语句不影响表所占用的extent，高水线(high watermark)保持原位置不变。

二、truncate

1、truncate是DDL，会隐式提交，所以，不能回滚，不会触发触发器。

2、truncate会删除表中所有记录，并且将重新设置高水线和所有的索引，缺省情况下将空间释放到minextents个extent，除非使用reuse storage，。不会记录日志，所以执行速度很快，
但不能通过rollback撤消操作（如果一不小心把一个表truncate掉，也是可以恢复的，只是不能通过rollback来恢复）。

3、对于外键（foreignkey ）约束引用的表，不能使用 truncate table，而应使用不带 where 子句的 delete 语句。

4、truncatetable不能用于参与了索引视图的表。

三、drop

1、drop是DDL，会隐式提交，所以，不能回滚，不会触发触发器。

2、drop语句删除表结构及所有数据，并将表所占用的空间全部释放。

3、drop语句将删除表的结构所依赖的约束，触发器，索引，依赖于该表的存储过程/函数将保留,但是变为invalid状态。
复制代码
 

27、数据库范式

1.第一范式：就是无重复的列
2.第二范式：就是非主属性非部分依赖于主关键字
3.第三范式：就是属性不依赖于其他非主属性（消除冗余）

28、MySQL锁类型

根据锁的类型分：可以分为共享锁、排他锁、意向共享锁和意向排他锁
根据锁的粒度分：可以分为行锁、表锁
对于mysql而言，事务机制更多是靠底层的存储引擎来实现的，因此，mysql层面只有表锁，
而支持事物的innodb存储引起则实现了行锁（在行相应的索引记录上的锁）
说明：对于更新操作（读不上锁），只有走索引才可能上行锁
MVCC（多版本并发控制）并发控制机制下，任何操作都不会阻塞读取操作，
读取操作也不会阻塞任何操作，只因为读不上锁
共享锁：由读表操作加上的锁，加锁后其他用户只能获取该表或行的共享锁，不能获取排他锁，
也就是说只能读不能写
排他锁：由写表操作加上的锁，加锁后其他用户不能获取该表或该行的任何锁，典型mysql事物中的更新操作
意向共享锁（IS）：事物打算给数据行加行共享锁，事物在给一个数据行加共享锁前必须先取得该表的IS锁
意向排他锁（IX）：事物打算给数据行加行排他锁，事物在给一个数据行家排他锁前必须先取得该表的IX锁
29、如何解决MYSQL数据库中文乱码问题？

1.在数据库安装的时候指定字符集
2.如果在按完了以后可以更改配置文件
3.建立数据库时候：指定字符集类型
4.建表的时候也指定字符集

30、数据库应用系统设计

1.规划 
2.需求分析
3.概念模型设计
4.逻辑设计
5.物理设计
6. 程序编制及调试
7.运行及维护 
ps：数据库常见面试问题总结
https://yq.aliyun.com/wenji/