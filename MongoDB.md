# 概念

非关系型数据库，文档型数据库

文档类似于mysql中的行

数据库由一个个集合组成，集合对应mysql中的表，每个集合包含多个文档，每个文档对应mysql中的一行，文档使用json的数据格式来组织和存储数据，不同的文档中间不需要有相同的结构和字段

MongoDB 不需要事先创建好数据库和集合然后再插入数据，也不需要预先定义好集合中字段的数据类型和长度，同一个集合中的数据也不需要有相同的结构







# 常用命令

显示所有数据库`databases` 或 `show dbs`

切换当前数据库 `use 数据库名称`

只有在插入集合之后, 数据库才会被真正地创建

插入一个文档到集合中 `db.集合名.insertOne({key:"value"})`

查看数据 `db.集合名.find()`

插入多条数据到集合中 `db.集合名.insertMany([{key:"value"},{key:"value"}])`

# 查询操作

limit() —— 返回n条数据

~~~MongoDB
db.集合名.find().limit(n)
~~~



sort() —— 排序 

~~~
-- 1表示按升序排序，-1表示降序
db.集合名.find().sort({level:1}) 

按照等级和名字排序
db.集合名.find().sort({level: 1, name: -1})
~~~

skip() —— 跳过一些查询数据

~~~
返回第二名开始的数据
db.集合名.find().sort({level:1}).limit(2).skip(1)

一般skip函数和sort函数一起使用实现分页功能
~~~

find({field:value}) —— 条件查询

~~~
db.集合名.find({level:3})  --查询等级为3的用户

db.集合名.find({level:3},{name:1,email:1})  --只返回name和email字段

db.集合名.find({level:3},{email:0})  --返回除了email之外的所有字段

db.集合名.find({level:{$gt:3}})  --查询等级大于3的用户
还有 lt小于，eq等于

查询某个字段的值是否在一个数组中
db.集合名.find({level:{$in:[1,3]}})  --返回等级为1或者3的用户
Not IN:表示不再这个数组中的值

判断某个字段是否存在，不能查看某个字段的值是否存在，
db.集合名.find({email:{$exists:true}})  --查询有邮箱的用户

多条件查询，默认会把多个条件组成一个and条件的查询
db.集合名.find({level:{$gte:3,$lte:5}})  -- 查询等级大于等于3小于等于5的用户
也可以显示指定and，把两个条件放到一个数组里面
db.集合名.find({$and:[{level:{$gte:3}},{level:{$lte,5}}]})  -- 查询等级大于等于3小于等于5的用户，把and换成 or 表示或者关系

db.集合名.find({level:{$not:{$eq:3}}})  --查询等级不等于3的用户

db.集合名.find({name:{$reges:/张/}})  --查询名字中姓张的用户，张所在的位置为正则表达式
~~~

countDocuments() —— 统计文档数量

可以传入查询条件

findOne() —— 查询满足条件的第一条数据

updateOne() —— 更新一条数据

updateMany() —— 更新多条数据

*如果字段不存在，会创建这个字段，并把值写入这个字段*

~~~
把等级为1 的用户的钱更新为100
db.集合名.updataOne({level:1},{$set:{money:100}})
~~~

deleteOne() —— 删除一条数据

deleteMany() —— 删除满足条件的多条数据

~~~
删除等级为1的用户
db.集合名.deleteOne({level:1})
~~~

