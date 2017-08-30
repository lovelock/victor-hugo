+++
date = "2017-08-30T17:00:00+08:00"
title = "到底用TIMESTAMP还是DATETIME还是INT存储时间？"
tags = ["mysql","TIMESTAMP", "DATETIME"]
isCJKLanguage = true
categories = ["MySQL"]

+++

这里研究一下MYSQL存储时间到底应用该哪种时间格式。

首先看一下这里提到的几种时间格式的表现形式。

```
mysql> create table time_test (
    ->     ts TIMESTAMP,
    ->     dt DATETIME,
    ->     it INT
    -> );
Query OK, 0 rows affected (0.27 sec)

mysql> desc time_test;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| ts    | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| dt    | datetime  | YES  |     | NULL              |                             |
| it    | int(11)   | YES  |     | NULL              |                             |
+-------+-----------+------+-----+-------------------+-----------------------------+
3 rows in set (0.01 sec)
```

可以看到, TIMESTAMP创建时如果不做特别设置, 它会在改行数据有修改时同步修改.
mysql> insert into time_test set it = unix_timestamp();
Query OK, 1 row affected (0.08 sec)

```
mysql> select * from time_test;
+---------------------+------+------------+
| ts                  | dt   | it         |
+---------------------+------+------------+
| 2017-08-30 17:08:34 | NULL | 1504084114 |
+---------------------+------+------------+
1 row in set (0.00 sec)

mysql> update time_test set dt = '2017-08-30 17:08:34';
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from time_test;
+---------------------+---------------------+------------+
| ts                  | dt                  | it         |
+---------------------+---------------------+------------+
| 2017-08-30 17:09:24 | 2017-08-30 17:08:34 | 1504084114 |
+---------------------+---------------------+------------+
1 row in set (0.00 sec)
```

实验也证实了这一点. 这里也提一点,`unix_timestamp()`函数生成的数字正是10位的unix时间戳, 这和PHP默认的`time()`的输出完全一致. 所以其实我是倾向于以int型存储时间的, 只需要在应用中写一套按系统中约定的时间格式来格式化时间的方法即可. 但缺点也很明显:

1. 如果需要保存一个更新时间, 就需要从应用中传入
1. 在**MySQL的客户端**中查看时不直观, 或者说不那么直观, 因为是可以通过`from_unixtime()`方法来把它转成我们熟悉的时间格式的.
1. 也正是因为它是int型, 所以它的`from_unixtime()`支持的上限就和TIMESTAMP一样, 是2038-01-19 11:14:07, 现在是2017年, 后面的20年会发生什么事很不好说啊. 如果不考虑转换, 并且希望它能表达2038年之后的时间, 可以用int unsigned

而优点是很明显的:

1. 可以在应用中方便的比较大小, 因为是int型嘛
1. 通用, 因为本质上所有时间都是一个int型, 只是表现形式不同, 如果当下存了一个2038-01-19 11:14:07这种格式的,后面产品经理觉得这种格式不够酷炫了, 那就需要把它先转成int再转成相应的格式
1. 无关时区, 完全从应用控制时间

这个和TIMESTAMP与DATETIME的相关性不大, 就放在前面说了, 下面详细分析一下这二者的区别和优缺点.

```
mysql> create table time_test2 (
    ->     id int not null auto_increment primary key,
    ->     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ->     updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    -> );
Query OK, 0 rows affected (0.27 sec)

mysql> desc time_test2;
+------------+-----------+------+-----+-------------------+-----------------------------+
| Field      | Type      | Null | Key | Default           | Extra                       |
+------------+-----------+------+-----+-------------------+-----------------------------+
| id         | int(11)   | NO   | PRI | NULL              | auto_increment              |
| created_at | timestamp | NO   |     | CURRENT_TIMESTAMP |                             |
| updated_at | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+------------+-----------+------+-----+-------------------+-----------------------------+
3 rows in set (0.00 sec)

mysql> insert into time_test2 set id = 1;
Query OK, 1 row affected (0.03 sec)

mysql> select * from time_test2;
+----+---------------------+---------------------+
| id | created_at          | updated_at          |
+----+---------------------+---------------------+
|  1 | 2017-08-30 17:30:38 | 2017-08-30 17:30:38 |
+----+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> update time_test2 set id = 3;
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from time_test2;
+----+---------------------+---------------------+
| id | created_at          | updated_at          |
+----+---------------------+---------------------+
|  3 | 2017-08-30 17:30:38 | 2017-08-30 17:30:56 |
+----+---------------------+---------------------+
1 row in set (0.00 sec)

```

这样看起来好像没什么问题, 占用4个字节, 比DATETIME的8个字节使用更少的空间, 还可以在客户端里直观的展示. 它的问题主要在于两点:

1. 时间上限和int型一样, 到2038年
1. 和时区相关, 如果修改了系统时区, 当前已经存储的值会发生变化. 或者说它内部存储的值并没有变化, 但这种类型的特点是写入时根据时区做一次转换, 取出时再根据时区做一次转换, 也就是说即使它内部存储的值没有变化, 但你是无法拿到这个没有变化的值的, 你看到的结果就是它变化了. 还以上面这条数据为例, 这里修改一下MySQL的时区设置

```
mysql> select * from time_test2;
+----+---------------------+---------------------+
| id | created_at          | updated_at          |
+----+---------------------+---------------------+
|  3 | 2017-08-30 17:30:38 | 2017-08-30 17:30:56 |
+----+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> set time_zone = '+9:00';
Query OK, 0 rows affected (0.00 sec)

mysql> select * from time_test2;
+----+---------------------+---------------------+
| id | created_at          | updated_at          |
+----+---------------------+---------------------+
|  3 | 2017-08-30 18:30:38 | 2017-08-30 18:30:56 |
+----+---------------------+---------------------+
1 row in set (0.00 sec)
```

这个问题虽说存在, 但我觉得对于很多个人或企业来说其实是完全不必担心的, 因为上线之初肯定会设置好系统时区设置, 后面也不会随意变动. 但如果要处理跨国(跨时区)的业务, 那这个问题就不得不考虑了. 

想象这样一个场景, 你的数据库服务器设在中国, 那很可能你的设置是这样的

```
mysql> show variables like '%zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.01 sec)
```

这时有中国客户访问是没有问题的, 如果有一个日本客户的访问导致在数据库插入一条数据, 他的当地时间是 2017-08-30 17:39:00, 按照这种设计, 它写入数据库的时间会是北京时间, 也就是 2017-08-30 16:39:00, 那么它取出的时间也是北京时间, 当他看到这个时间时肯定会疑惑, 我刚刚注册的帐号,为什么注册时间会是1个小时以前? 如果考虑到这种情况, 其实即便是把时间计算的问题放在应用层解决也并没有任何效果, 除非应用服务器根据当地时间转换成北京时间, 然后读取时再根据所在时区转换成当地时间. 那么问题来了, 拿CST作为标准也有潜在的问题, 如果在中国机房的数据库在北美机房有备库, 在做备份之前还需要将北美当地的机器系统时间设置成北京时间, 当我觉得不如用世界通用的UTC时间. 具体到Linux系统中, 以CentOS为例, 执行`sudo ln -s -f /usr/share/timezone/UTC /etc/localtime`然后重启MySQL服务器即可. 这样的缺点也很明显, 就是除了在0时区, 无论机器在任何地方, 你在系统中看到的时间都和你身边的手表上的时间不同.

说完了时区的问题, 根据[这篇答案](https://stackoverflow.com/questions/409286/should-i-use-field-datetime-or-timestamp?page=1&tab=active#tab-top)的说法, TIMESTAMP存在的本意就是为了追踪一行数据的更新时间, 而DATETIME则是为了保存一个时间, 比如一件商品的生产时间, 这个时间是作为这行数据的一个属性存在的, 不会改变的. 而后者的优点则是存储的时间更长, 能存储最大到9999年的时间, 当然代价是它的存储占用了8个字节, 比TIMESTAMP和int多一倍. 不过我觉得存储空间是最不重要的考量因素.

我的结论就是如果要存储创建时间更新时间, 可以用TIMESTAMP, 至于2038年的问题, 就像当年的千年虫一样, 没什么可怕的, 如果你的公司能活到20年以后, MySQL肯定也能解决这个问题. 如果要存储不会改变的时间, 如生产时间, 可以用DATETIME.