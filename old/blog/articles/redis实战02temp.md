# Redis实战02：Redis命令及数据类型
----------------------------------------

本文是Redis学习的第二部分，主要介绍Redis的数据类型及其操作。


-----------------------------------------

## 注意点
1. Redis命令不区分大小写，我们约定一般命令为大写，便于区分。
2. Keys命令： 该命令需要遍历Redis中的所有键，当键的数量比较多时会影响性能，不建议在生产环境使用。
3. DEL: 删除一个或者多个键，返回删除的键的个数。
4. TYPE命令： 获取键值的数据类型，例如

	redis 127.0.0.1:6379> SET FOO 123
	OK
	redis 127.0.0.1:6379> type FOO
	string

## 命令详解

### 赋值取值

		set key value 赋值
		get key		  取值
*当键不存在则返回空结果*

### 递增数字
Redis提供了一个实用的命令INCR，其作用是让当前键值递增，并返回递增后端值，如下

当键不存在时默认的键值为0
	redis 127.0.0.1:6379> INCR NUM
	(integer) 1
	redis 127.0.0.1:6379> INCR NUM
	(integer) 2
	redis 127.0.0.1:6379> INCR NUM
	(integer) 3
	redis 127.0.0.1:6379>
	redis 127.0.0.1:6379> INCR NUM
	(integer) 4
	redis 127.0.0.1:6379> INCR NUM
	(integer) 5

### 字符串尾部追加值
append key value ，例如

		redis 127.0.0.1:6379> GET name
		"snowalker"
		redis 127.0.0.1:6379> append name 23333
		(integer) 14
		redis 127.0.0.1:6379> GET name
		"snowalker23333"

### MGET/MSET批量获取、添加键值

		redis 127.0.0.1:6379> MSET A 1 B 2 C 3 D 4
		OK
		redis 127.0.0.1:6379> MGET A B C D
		1) "1"
		2) "2"
		3) "3"
		4) "4"
## 散列类型

### 注意

散列类型不能嵌套其他数据类型，字段值只能是字符串；

除了散列类型，Redis的其他数据类型同样不支持数据类型嵌套，比如集合类型的每个元素只能是字符串，不能是另外一个集合或者散列表等。

### HSET/HGET/HMGET/HMSET/HGETALL
*散列类型适合存储对象*，这对于面向对象编程很是重要。区别于传统的SQL数据库，如果要增加字段，可能会面临字段冗余的可能，而且要增加字段需要重新启动服务，这在Redis中是完全可以避免的。

Redis并不要求每个键都依据传统的SQL形式的二维表进行存储，我们完全可以自由地为任何键增减字段而不影响其他键。

1. 通过HSET建立散列类型，它不区分插入和更新操作，如果是插入HSET返回1，如果是更新操作HSET返回0；
2. 当键不存在，HSET会建立它

#### HSET/HGET给字段赋值

	redis 127.0.0.1:6379> HSET car price 100
	(integer) 1
	redis 127.0.0.1:6379> HSET movie name naruto
	(integer) 1
	redis 127.0.0.1:6379> HSET movie time 20minutes
	(integer) 1
	redis 127.0.0.1:6379> HSET movie date 1999-2016
	(integer) 1
	redis 127.0.0.1:6379> HGET movie name
	"naruto"
	redis 127.0.0.1:6379> HGET movie time
	"20minutes"
	redis 127.0.0.1:6379> HGET movie date
	"1999-2016"
### HMSET/HMGET批量数据操作
这里建立两个movie对象分别命名为movie-01，movie-02

	redis 127.0.0.1:6379> HMSET movie-01   name bleach date 2001-2016 time 21minute
	OK
	redis 127.0.0.1:6379> HMSET movie-02   name naruto date 1999-2016 time 20minute
	OK

分别批量获取两个对象的属性值

	redis 127.0.0.1:6379> HMGET movie-01 name date time
	1) "bleach"
	2) "2001-2016"
	3) "21minute"
	redis 127.0.0.1:6379> HMGET movie-02 name date time
	1) "naruto"
	2) "1999-2016"
	3) "20minute"

### HGETALL
如果想要获取键中所有字段即字段值却不知道有哪些字段时，应该使用HGETALL命令

	redis 127.0.0.1:6379> HGETALL movie
	1) "name"
	2) "bleach"
	3) "time"
	4) "21minute"
	5) "date"
	6) "2001-2016"

### 判断字段是否存在 HEXISTS KEY fields

返回0表示不存在该字段，返回1表字段存在

	redis 127.0.0.1:6379> hexists movie model
	(integer) 0
	redis 127.0.0.1:6379> hexists movie name
	(integer) 1

### 当字段不存在则赋值 HSETNX KEY fields value

	redis 127.0.0.1:6379> hsetnx movie model newMovie
	(integer) 1

### 删除字段 HDEL KEY field [fields...]
这里以删除movie的model字段为例
	redis 127.0.0.1:6379> HDEL movie model
	(integer) 1
	redis 127.0.0.1:6379> HGETALL movie
	1) "name"
	2) "bleach"
	3) "time"
	4) "21minute"
	5) "date"
	6) "2001-2016"