# MongoDB基础

> 一切以 [官方文档](https://www.mongodb.com/zh-cn/docs/) 为主，本笔记只提供常用的 MongoDB 操作方式参考

## 1 MongoDB 相关概念

### 1.1 业务场景

传统的关系型数据库 (比如 MySQL), 在数据操作的”三高”需求以及对应的 Web 2.0 网站需求面前, 会有”力不从心”的感觉

所谓的三高需求:

**高并发, 高性能, 高可用**, 简称三高

- High Performance: 对数据库的高并发读写的要求
- High Storage: 对海量数据的高效率存储和访问的需求
- High Scalability && High Available: 对数据的高扩展性和高可用性的需求

**而 MongoDB 可以应对三高需求**

具体的应用场景:

- 社交场景, 使用 MongoDB 存储存储用户信息, 以及用户发表的朋友圈信息, 通过地理位置索引实现附近的人, 地点等功能.
- 游戏场景, 使用 MongoDB 存储游戏用户信息, 用户的装备, 积分等直接以内嵌文档的形式存储, 方便查询, 高效率存储和访问.
- 物流场景, 使用 MongoDB 存储订单信息, 订单状态在运送过程中会不断更新, 以 MongoDB 内嵌数组的形式来存储, 一次查询就能将订单所有的变更读取出来.
- 物联网场景, 使用 MongoDB 存储所有接入的智能设备信息, 以及设备汇报的日志信息, 并对这些信息进行多维度的分析.
- 视频直播, 使用 MongoDB 存储用户信息, 点赞互动信息等.

这些应用场景中, 数据操作方面的共同点有:

1. 数据量大
2. 写入操作频繁
3. 价值较低的数据, 对**事务性**要求不高

对于这样的数据, 更适合用 MongoDB 来实现数据存储

那么我们**什么时候选择 MongoDB 呢?**

除了架构选型上, 除了上述三个特点之外, 还要考虑下面这些问题:

- 应用不需要事务及复杂 JOIN 支持
- 新应用, 需求会变, 数据模型无法确定, 想快速迭代开发
- 应用需要 2000 - 3000 以上的读写 QPS（更高也可以）
- 应用需要 TB 甚至 PB 级别数据存储
- 应用发展迅速, 需要能快速水平扩展
- 应用要求存储的数据不丢失
- 应用需要 `99.999%` 高可用
- 应用需要大量的地理位置查询, 文本查询

如果上述有 1 个符合, 可以考虑 MongoDB, 2 个及以上的符合, 选择 MongoDB 绝不会后悔.

> 如果用 MySQL 呢?
>
> 相对 MySQL, 可以以更低的成本解决问题（包括学习, 开发, 运维等成本）

### 1.2 MongoDB 简介

> MongoDB 是一个开源, 高性能, 无模式的文档型数据库, 当初的设计就是用于简化开发和方便扩展, 是 NoSQL 数据库产品中的一种.是最 像关系型数据库（MySQL）的非关系型数据库. 它支持的数据结构非常松散, 是一种类似于 JSON 的 格式叫 BSON, 所以它既可以存储比较复杂的数据类型, 又相当的灵活. MongoDB 中的记录是一个文档, 它是一个由字段和值对（ﬁeld:value）组成的数据结构.MongoDB 文档类似于 JSON 对象, 即一个文档认 为就是一个对象.字段的数据类型是字符型, 它的值除了使用基本的一些类型外, 还可以包括其他文档, 普通数组和文档数组.

**“最像****关系型数据库****的** **NoSQL** **数据库”**. MongoDB 中的记录是一个文档, 是一个 key-value pair. 字段的数据类型是字符型, 值除了使用基本的一些类型以外, 还包括其它文档, 普通数组以及文档数组

### 1.3 与 MySQL 的映射关系

![img](https://qi8z4a2rmeb.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGUxYmI2MjUxYWE2OTdhMGNmMWRmYmI4NzU2ZmQzNThfcFc2a1JvRGt3cm1WRlVPMGdGTlZUQjZuN21wWG53bjRfVG9rZW46U21VSWJFQjMwb3IycVB4UzBsb2NydVlkbjVjXzE3Mjg5ODk4ODA6MTcyODk5MzQ4MF9WNA)

| SQL 术语/概念 | MongoDB 术语/概念 | 解释/说明                             |
| ------------- | ----------------- | ------------------------------------- |
| database      | database          | 数据库                                |
| table         | collection        | 数据库表/集合                         |
| row           | document          | 数据记录行/文档 ww                    |
| column        | field             | 数据字段/域                           |
| index         | index             | 索引                                  |
| table joins   |                   | 表连接,MongoDB 不支持                 |
|               | 嵌入文档          | MongoDB 通过嵌入式文档来替代多表连接  |
| primary key   | primary key       | 主键,MongoDB 自动将_id 字段设置为主键 |

MongoDB 数据模型是面向文档的, 所谓文档就是一种类似于 JSON 的结构, 简单理解 MongoDB 这个数据库中存在的是各种各样的 JSON（BSON）

- 数据库 (database)
   - 数据库是一个仓库, 存储集合 (collection)
- 集合 (collection)
   - 类似于数组, 在集合中存放文档
- 文档 (document)
   - 文档型数据库的最小单位, 通常情况, 我们存储和操作的内容都是文档

在 MongoDB 中, 数据库和集合都不需要手动创建, 当我们创建文档时, 如果文档所在的集合或者数据库不存在, **则会自动创建数据库或者集合**

### 1.4 NoSQL 的特点

优点：

- 易于插入和检索数据，因为它们包含在一个块中，在一个 json 对象中
- 灵活的 schema，如果添加了新属性，则很容易直接添加 / 附加到对象
- 可伸缩性，水平分区数据（可用性 > 一致性）
- 聚合、查找指标等

缺点：

- Update = Delete + Insert, not built for update
- 不一致，ACID 不保证，不支持事务
- 未进行读取优化。读取整个块，找到属性。但是 SQL，只需要一列（读取时间相对较慢）
- 关系不是隐式的
- JOINS 很难完成，全部手动完成

### 1.5 MongoDB 的特点

Mongo DB 主要有如下特点：

(1)高性能：

Mongo DB 提供高性能的数据持久性。特别是，对嵌入式数据摸型的支持减少了数据库系统上的 O 活动。

索引支持更快的查询，并且可以包含来自嵌入式文档和数组的键。（文本索引解决搜索的需求、TL 索引解决历史数据自动过期的需求、地理位置索引可用于构建各种 020 应用)

mmapv1、wiredtiger、mongorocks(rocksdb)、in-memory 等多引擎支持满足各种场景需求。

Gridfs 解决文件存储的需求。

(2)高可用性：

Mongo DB 的复制工具称为副本集(replica set),它可提供自动故障转移和数据冗余。

(3)高扩展性：

MongoDB:提供了水平可扩展性作为其核心功能的一部分。

分片将数据分布在一组集群的机器上。（海量数据存储，服务能力水平扩展）

从 3.4 开始，MongoDB 支持基于片键创建数据区域。在一个平衡的集群中，MongoDB 将一个区域所覆盖的读写只定向到该区域内的那些片。

(4)丰富的查询支持：

Mongo DB 支持丰富的查询语言，支持读和写操作(CRUD),比如数据聚合、文本搜索和地理空间查询等。

(5)其他特点：如无模式（动态模式）、灵活的文档模型、

## 2 基本常用命令

### 2.1 数据库 / 集合操作

默认保留的数据库

- **admin**: 从权限角度考虑, 这是 `root` 数据库, 如果将一个用户添加到这个数据库, 这个用户自动继承所有数据库的权限, 一些特定的服务器端命令也只能从这个数据库运行, 比如列出所有的数据库或者关闭服务器
- **local**: 数据永远不会被复制, 可以用来存储限于本地的单台服务器的集合 (部署集群, 分片等)
- **config**: Mongo 用于分片设置时, `config` 数据库在内部使用, 用来保存分片的相关信息

```js
# 展示所有数据库
show dbs

# 选择/创建 数据库
use articledb

# 删除当前数据库
db.dropDatabase()

# 显式的创建集合（insert时会隐式创建）
db.createCollection(name) 
db.createCollection("mycollection") 

# 展示所有集合
show collections 
或 
show tables

# 删除指定集合
db.<collection_name>.drop() 
```

> 当使用 `use articledb` 的时候. `articledb` 其实存放在内存之中, 当 `articledb` 中存在一个 collection 之后, mongo 才会将这个数据库持久化到硬盘之中.

### 2.2 文档基本 CRUD

> 官方文档: https://docs.mongodb.com/manual/crud/

#### 2.2.1 创建 insert

> Create or insert operations add new **[documents](https://docs.mongodb.com/manual/core/document/#bson-document-format)** to a **[collection](https://docs.mongodb.com/manual/core/databases-and-collections/#collections)**. If the collection does **not** currently exist, insert operations will create the collection automatically.

- 使用 `db.<collection_name>.insertOne()` 向集合中添加*一个文档*, 参数一个 json 格式的文档
- 使用 `db.<collection_name>.insertMany()` 向集合中添加*多个文档*, 参数为 json 文档数组

```js
db.collection.insert({
  <document or array of documents>,
  writeConcern: <document>,
  ordered: <boolean>
})


// 向集合中添加一个文档
db.collection.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
// 向集合中添加多个文档
db.collection.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```

注：当我们向 `collection` 中插入 `document` 文档时, 如果没有给文档指定 `_id` 属性, 那么数据库会为文档自动添加 `_id` field, 并且值类型是 `ObjectId(blablabla)`, 就是文档的唯一标识, 类似于 relational database 里的 `primary key`

> - mongo 中的数字, 默认情况下是 double 类型, 如果要存整型, 必须使用函数 `NumberInt(整型数字)`, 否则取出来就有问题了
> - 插入当前日期可以使用 `new Date()`

如果某条数据插入失败, 将会终止插入, 但已经插入成功的数据**不会回滚掉**. 因为批量插入由于数据较多容易出现失败, 因此, 可以使用 `try catch` 进行异常捕捉处理, 测试的时候可以不处理.如：

```js
try {
  db.comment.insertMany([
    {"_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在手机上, 健康很重要, 一杯温水幸福你我 他.","userid":"1002","nickname":"相忘于江湖","createdatetime":new Date("2019-0805T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"},
    {"_id":"2","articleid":"100001","content":"我夏天空腹喝凉开水, 冬天喝温开水","userid":"1005","nickname":"伊人憔 悴","createdatetime":new Date("2019-08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"},
    {"_id":"3","articleid":"100001","content":"我一直喝凉开水, 冬天夏天都喝.","userid":"1004","nickname":"杰克船 长","createdatetime":new Date("2019-08-06T01:05:06.321Z"),"likenum":NumberInt(666),"state":"1"},
    {"_id":"4","articleid":"100001","content":"专家说不能空腹吃饭, 影响健康.","userid":"1003","nickname":"凯 撒","createdatetime":new Date("2019-08-06T08:18:35.288Z"),"likenum":NumberInt(2000),"state":"1"},
    {"_id":"5","articleid":"100001","content":"研究表明, 刚烧开的水千万不能喝, 因为烫 嘴.","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-0806T11:01:02.521Z"),"likenum":NumberInt(3000),"state":"1"}

]);

} catch (e) {
  print (e);
}
```

#### 2.2.2 查询 find

- 使用 `db.<collection_name>.find()` 方法对集合进行查询, 接受一个 json 格式的查询条件. 返回的是一个**数组**
- `db.<collection_name>.findOne()` 查询集合中符合条件的第一个文档, 返回的是一个**对象**

![img](https://qi8z4a2rmeb.feishu.cn/space/api/box/stream/download/asynccode/?code=YzM3Mzg2YTExZDM1OGVlYjQ5NTIyNjk3OWU4MjJmOWNfUzc5dnluMXRZTnlXb2J3OXdrRWY3N2hmcW1LRTNZcG9fVG9rZW46WXpDU2JYZWE2b0RvTjd4Y2tVcWNBNTFYbjNkXzE3Mjg5ODk4ODA6MTcyODk5MzQ4MF9WNA)

可以使用 `$in` 操作符表示*范围查询*

```js
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
```

多个查询条件用逗号分隔, 表示 `AND` 的关系

```js
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```

等价于下面 sql 语句

```js
SELECT * FROM inventory WHERE status = "A" AND qty < 30
```

使用 `$or` 操作符表示后边数组中的条件是 OR 的关系

```js
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```

等价于下面 sql 语句

```js
SELECT * FROM inventory WHERE status = "A" OR qty < 30
```

联合使用 `AND` 和 `OR` 的查询语句

```js
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

在 terminal 中查看结果可能不是很方便, 所以我们可以用 `pretty()` 来帮助阅读

```js
db.inventory.find().pretty()
```

匹配内容

```js
db.posts.find({
  comments: {
    $elemMatch: {
      user: 'Harry Potter'
    }
  }
}).pretty()

// 正则表达式
db.<collection_name>.find({ content : /once/ })
```

创建索引

```js
db.posts.createIndex({
  { title : 'text' }
})

// 文本搜索
// will return document with title "Post One"
// if there is no more posts created
db.posts.find({
  $text : {
    $search : "\"Post O\""
  }
}).pretty()
```

#### 2.2.3 更新 update

- 使用 `db.<collection_name>.updateOne(<filter>, <update>, <options>)` 方法修改一个匹配 `<filter>` 条件的文档
- 使用 `db.<collection_name>.updateMany(<filter>, <update>, <options>)` 方法修改所有匹配 `<filter>` 条件的文档
- 使用 `db.<collection_name>.replaceOne(<filter>, <update>, <options>)` 方法**替换**一个匹配 `<filter>` 条件的文档
- `db.<collection_name>.update(查询对象, 新对象)` 默认情况下会使用新对象替换旧对象

其中 `<filter>` 参数与查询方法中的条件参数用法一致.

如果需要修改指定的属性, 而不是替换需要用“修改操作符”来进行修改

- `$set` 修改文档中的制定属性

其中最常用的修改操作符即为`$set`和`$unset`,分别表示**赋值**和**取消赋值**.

```js
db.inventory.updateOne(
    { item: "paper" },
    {
        $set: { "size.uom": "cm", status: "P" },
        $currentDate: { lastModified: true }
    }
)

db.inventory.updateMany(
    { qty: { $lt: 50 } },
    {
        $set: { "size.uom": "in", status: "P" },
        $currentDate: { lastModified: true }
    }
)
```

> - uses the **`$set`** operator to update the value of the `size.uom` field to `"cm"` and the value of the `status` field to `"P"`,
> - uses the **`$currentDate`** operator to update the value of the `lastModified` field to the current date. If `lastModified` field does not exist, **`$currentDate`** will create the field. See **`$currentDate`** for details.

`db.<collection_name>.replaceOne()` 方法替换除 `_id` 属性外的**所有属性**, 其`<update>`参数应为一个**全新的文档**.

```js
db.inventory.replaceOne(
    { item: "paper" },
    { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

**批量修改**

```js
// 默认会修改第一条
db.document.update({ userid: "30", { $set {username: "guest"} } })

// 修改所有符合条件的数据
db.document.update( { userid: "30", { $set {username: "guest"} } }, {multi: true} )
```

**列值增长的修改**

如果我们想实现对某列值在原有值的基础上进行增加或减少, 可以使用 `$inc` 运算符来实现

```js
db.document.update({ _id: "3", {$inc: {likeNum: NumberInt(1)}} })
```

##### 修改操作符

| 姓名         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| $currentDate | 将字段的值设置为当前日期，可以是日期或时间戳。               |
| $inc         | 按指定的量增加字段的值。                                     |
| $min         | 仅当指定值小于现有字段值时才更新该字段。                     |
| $max         | 仅当指定值大于现有字段值时才更新该字段。                     |
| $mul         | 将字段的值乘以指定的量。                                     |
| $rename      | 重命名字段。                                                 |
| $set         | 设置文档中字段的值。                                         |
| $setonInsert | 如果更新导致文档插入，则设置字段的值。对修改现有文档的更新操作没有影响。(一般与 upsert 配合使用) |
| $unset       | 从文档中删除指定字段。                                       |

#### 2.2.4 删除 remove

- 使用 `db.collection.deleteMany()` 方法删除所有匹配的文档.
- 使用 `db.collection.deleteOne()` 方法删除单个匹配的文档.

```js
db.inventory.deleteMany( { qty : { $lt : 50 } } )
```

> Delete operations **do not drop indexes**, even if deleting all documents from a collection.
>
> 一般数据库中的数据都不会真正意义上的删除, 会添加一个字段, 用来表示这个数据是否被删除

删除文档的语法结构： 

```js
db.集合名称.remove(条件) 
```

以下语句可以将数据全部删除，请慎用 

```js
db.comment.remove({}) 
```

如果删除_id=1 的记录，输入以下语句

```js
db.comment.remove({_id:"1"})
```

### 2.3 文档投影 (projection)

有些情况, 我们对文档进行查询并不是需要所有的字段, 比如只需要 id 或者 用户名, 我们可以对文档进行“投影”

- `1` - display
- `0` - dont display

```js
> db.users.find( {}, {username: 1} )> db.users.find( {}, {age: 1, _id: 0} )
```

### 2.4 forEach()

```js
> db.posts.find().forEach(fucntion(doc) { print('Blog Post: ' + doc.title) })
```

### 2.5 文档的更多查询

#### 2.5.1 统计查询

1、统计所有记录数

```js
> db.class.count()
```

2、按条件统计

```js
> db.class.count({"sex":"man"})
```

#### 2.5.2 分页列表查询

```js
/**
 * 基础语法
 * limit : 显示多少条数据
 * skip : 跳过的参数,如果数字是3,那么前3个就不要了,默认是0
 * limit和skip不分前后
 */
db.collectionName,find().limit(NUMBER).skip(NUMBER)

-------------------------------------------------------

> db.class.find().limit(2)
{ "_id" : ObjectId("5f56f552c68adc5a2105ef64"), "name" : "Bob", "age" : "21", "sex" : "man" }
{ "_id" : ObjectId("5f56f552c68adc5a2105ef65"), "name" : "Juen", "age" : "19", "sex" : "man" }
> db.class.find().limit(2).skip(1)
{ "_id" : ObjectId("5f56f552c68adc5a2105ef65"), "name" : "Juen", "age" : "19", "sex" : "man" }
{ "_id" : ObjectId("5f56f58ac68adc5a2105ef66"), "name" : "Jesson", "age" : "21", "sex" : "man" }

----------------------------------------------------

//需求:每页两个,从第二页开始跳过前两条,接着显示第三、四条
//因为_id的值太过邋遢,所以直接就用命令不显示了
> db.class.find({},{_id:false}).skip(0).limit(2)
{ "name" : "Bob", "age" : "21", "sex" : "man" }
{ "name" : "Juen", "age" : "19", "sex" : "man" }
> db.class.find().skip(2).limit(2)
{ "name" : "Jesson", "age" : "21", "sex" : "man" }
{ "name" : "NewLily", "age" : "17", "sex" : "woman" }
> db.class.find().skip(4).limit(2)
{ "name" : "Rainbow", "age" : "23", "sex" : "woman" }
```

#### 2.5.3 排序查询

注:skip(),limit(),sort()同时出现时,执行的顺序是 sort()>skip()>limit() 和命令编写的顺序无关

```js
/**
 * 语法:可多条件查询
 * 1为升序
 * 2为降序
 */
db.colletionName.find().sort(排序方式)

//根据age排序
> db.class.find({},{_id:false}).sort({"key":1,age:-1})
{ "name" : "Rainbow", "age" : "23", "sex" : "woman" }
{ "name" : "Bob", "age" : "21", "sex" : "man" }
{ "name" : "Jesson", "age" : "21", "sex" : "man" }
{ "name" : "Juen", "age" : "19", "sex" : "man" }
{ "name" : "NewLily", "age" : "17", "sex" : "woman" }
```

#### 2.5.4 正则的复杂条件查询

```js
/**
 * 支持js的正则写法
 */
db.collectionName.find({content:正则表达式})
```

#### 2.5.5 比较查询

```js
/**
 * $gt >; $lt <; $gte >=; $lte <=; $ne !=;
 */
db.collectionName,find({"field":{$gt:value}})

//查找age>20的数据,因为当初存数据的时候存成string类型了,所以数字需要加双引号"" '
> db.class.find({age : {$gt : "20"}})
{ "name" : "Bob", "age" : "21", "sex" : "man" }
{ "name" : "Jesson", "age" : "21", "sex" : "man" }
{ "name" : "Rainbow", "age" : "23", "sex" : "woman" }
```

#### 2.5.6 包含查询

1、$in 的使用

```js
/**
 * $in和sql中的in意思一样'
 */
> db.class.find({age:{$in:["19","17"]}},{_id:0})
{ "name" : "Juen", "age" : "19", "sex" : "man" }
{ "name" : "NewLily", "age" : "17", "sex" : "woman" }
```

#### 2.5.7 条件链接查询

1、and 的使用

```js
/**
 * 语法
 * and 和 or语法一样
 */
 db.collectionName.find({$and[{},{},{}....]})
 
 //需求：19 <= 年龄 <= 21并且按照年龄升序'
 > db.class.find({$and:[{age:{$gte:"19"}},{age:{$lte:"21"}}]},{_id:0}).sort({age:1})
{ "name" : "Juen", "age" : "19", "sex" : "man" }
{ "name" : "Bob", "age" : "21", "sex" : "man" }
{ "name" : "Jesson", "age" : "21", "sex" : "man" }
```

2、or 的使用

```js
//需求:年龄=17或者年龄=23的学生按照age排序'
> db.class.find({$or:[{age:"17"},{age:"23"}]},{_id:0}).sort({age:1})
{ "name" : "NewLily", "age" : "17", "sex" : "woman" }
{ "name" : "Rainbow", "age" : "23", "sex" : "woman" }
```

### 2.6 常用命令小结

```js
选择切换数据库：use articledb
插入数据：db.comment.insert({bson数据})
查询所有数据：db.comment.find();
条件查询数据：db.comment.find({条件})
查询符合条件的第一条记录：db.comment.findOne({条件})
查询符合条件的前几条记录：db.comment.find({条件}).limit(条数)
查询符合条件的跳过的记录：db.comment.find({条件}).skip(条数)
修改数据：db.comment.update({条件},{修改后的数据})
        或
        db.comment.update({条件},{$set:{要修改部分的字段:数据})
修改数据并自增某字段值：db.comment.update({条件},{$inc:{自增的字段:步进值}})
删除数据：db.comment.remove({条件})
统计查询：db.comment.count({条件})
模糊查询：db.comment.find({字段名:/正则表达式/})
条件比较运算：db.comment.find({字段名:{$gt:值}})
包含查询：db.comment.find({字段名:{$in:[值1, 值2]}})
        或
        db.comment.find({字段名:{$nin:[值1, 值2]}})
条件连接查询：db.comment.find({$and:[{条件1},{条件2}]})
           或
           db.comment.find({$or:[{条件1},{条件2}]})
```

## 3 文档间的对应关系

- 一对一 (One To One)
- 一对多 (One To Many)
- 多对多 (Many To Many)

举个例子, 比如“用户-订单”这个一对多的关系中, 我们想查询某一个用户的所有或者某个订单, 我们可以

```js
var user_id = db.users.findOne( {username: "username_here"} )._id
db.orders.find( {user_id: user_id} )
```

多对多就把 user_id 变为一个数组

## 4 MongoDB 的索引

### 4.1 概述

索引是特殊的数据结构, 它以易于遍历的形式存储集合数据集的一小部分.索引存储特定字段或一组字段的值, 按字段值排序.索引项的排 序支持有效的相等匹配和基于范围的查询操作.此外, MongoDB 还可以使用索引中的排序返回排序结果.

MongoDB 使用的是 **B Tree**, MySQL 使用的是 B+ Tree

```js
// create index
db.<collection_name>.createIndex({ userid : 1, username : -1 })

// retrieve indexes
db.<collection_name>.getIndexes()

// remove indexes
db.<collection_name>.dropIndex(index)

// there are 2 ways to remove indexes:
// 1. removed based on the index name
// 2. removed based on the fields

db.<collection_name>.dropIndex( "userid_1_username_-1" )
db.<collection_name>.dropIndex({ userid : 1, username : -1 })

// remove all the indexes, will only remove non_id indexes
db.<collection_name>.dropIndexes()
```

### 4.2 索引的类型

#### 4.2.1 单字段索引

MongoDB 支持在文档的单个字段上创建用户定义的**升序/降序索引**, 称为**单字段索引** Single Field Index

对于单个字段索引和排序操作, 索引键的排序顺序（即升序或降序）并不重要, 因为 MongoDB 可以在任何方向上遍历索引.

#### 4.2.2 复合索引

MongoDB 还支持多个字段的用户定义索引, 即复合索引 Compound Index

复合索引中列出的字段顺序具有重要意义.例如, 如果复合索引由 `{ userid: 1, score: -1 }` 组成, 则索引首先按 `userid` 正序排序, 然后 在每个 `userid` 的值内, 再在按 `score` 倒序排序.

#### 4.2.3 其他索引

- 地理空间索引 Geospatial Index

为了支持对地理空间坐标数据的有效查询, MongoDB 提供了两种特殊的索引: 返回结果时使用平面几何的二维索引和返回结果时使用球面几何的二维球面索引.

- 文本索引 Text Indexes

MongoDB 提供了一种文本索引类型, 支持在集合中搜索字符串内容.这些文本索引不存储特定于语言的停止词（例如 “the”, “a”, “or”）, 而将集合中的词作为词干, 只存储根词.

- 哈希索引 Hashed Indexes

为了支持基于散列的分片, MongoDB 提供了散列索引类型, 它对字段值的散列进行索引.这些索引在其范围内的值分布更加随机, 但只支持相等匹配, 不支持基于范围的查询.

### 4.3 索引的管理操作

#### 4.3.1 索引的查看

语法

```js
db.collection.getIndexes()
```

默认 `_id` 索引： MongoDB 在创建集合的过程中, 在 `_id` 字段上创建一个唯一的索引, 默认名字为 `_id_`, 该索引可防止客户端插入两个具有相同值的文 档, 不能在 `_id` 字段上删除此索引.

注意：该索引是**唯一索引**, 因此值不能重复, 即 `_id` 值不能重复的.

在分片集群中, 通常使用 `_id` 作为**片键**.

#### 4.3.2 索引的创建

语法

```js
db.collection.createIndex(keys, options)
```

> 索引名称未指定时，默认为 field1_(1/0)...，通过下划线拼接索引列及其排序方式
>
> 例如：username_1_age_-1

参数

| Parameter | Type     | Description                                                  |
| --------- | -------- | ------------------------------------------------------------ |
| keys      | document | 包含字段和值对的文档，其中字段是索引键，值描述该字段的索引类型。对于字段上的升序索引，请 指定值 1；对于降序索引，请指定值 -1。比如：`{字段：1或-1}`，其中 1 为指定按升序创建索引，如果你 想按降序来创建索引指定为 -1 即可。另外，MongoDB 支持几种不同的索引类型，包括文本、地理空间和哈希索引。 |
| options   | document | 可选。包含一组控制索引创建的选项的文档。有关详细信息，请参见选项详情列表。 |

options（更多选项）列表

| Parameter          | Type Description |                                                              |
| ------------------ | ---------------- | ------------------------------------------------------------ |
| background         | Boolean          | 建索引过程会阻塞其它数据库操作，background 可指定以后台方式创建索引，即增加 "background"可选参数。"background"默认值为 **false**。 |
| unique             | Boolean          | 建立的索引是否唯一。指定为 true 创建唯一索引。默认值为 **false**. |
| name               | string           | 索引的名称。如果未指定，MongoDB 的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups           | Boolean          | **3.0+版本已废弃**。在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse             | Boolean          | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为 true 的话，在索 引字段中不会查询出不包含对应字段的文档。默认值为 **false**. |
| expireAfterSeconds | integer          | 指定一个以秒为单位的数值，完成 TTL 设定，设定集合的生存时间。 |
| V                  | index version    | 索引的版本号。默认的索引版本取决于 mongod 创建索引时运行的版本。 |
| weights            | document         | 索引权重值，数值在 1 到 99,999 之间，表示该索引引相对于其他索引字段的得分权重。 |
| default_language   | string           | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。默认为英语 |
| language_override  | string language. | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的 language，默认值为 |

注意在 3.0.0 版本前创建索引方法为 `db.collection.ensureIndex()` , 之后的版本使用了 `db.collection.createIndex()` 方法, `ensureIndex()` 还能用, 但只是 `createIndex()` 的别名.

```js
$  db.comment.createIndex({userid:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}

$ db.comment.createIndex({userid:1,nickname:-1})
...
```

#### 4.3.3 索引的删除

语法

```js
# 删除某一个索引
$ db.collection.dropIndex(index)

# 删除全部索引
$ db.collection.dropIndexes()
```

提示:

`_id` 的字段的索引是无法删除的, 只能删除非 `_id` 字段的索引

示例

```js
# 删除 comment 集合中 userid 字段上的升序索引
$ db.comment.dropIndex({userid:1})

# 根据 索引名 删除
$ db.comment.dropIndex("userid_1")
```

### 4.4 索引使用

#### 4.4.1 执行计划

分析查询性能 (Analyze Query Performance) 通常使用执行计划 (解释计划 - Explain Plan) 来查看查询的情况

```js
$ db.<collection_name>.find( query, options ).explain(options)
```

比如: 查看根据 `user_id` 查询数据的情况

**未添加索引之前**

`"stage" : "COLLSCAN"`, 表示全集合扫描

```js
> db.c.find({userid:1001}).explain()
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'test.c',
    parsedQuery: { userid: { '$eq': 1001 } },
    indexFilterSet: false,
    queryHash: '56449150',
    planCacheKey: '0099E6F0',
    optimizationTimeMillis: 0,
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    prunedSimilarIndexes: false,
    winningPlan: {
      isCached: false,
      stage: 'COLLSCAN',
      filter: { userid: { '$eq': 1001 } },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  command: { find: 'c', filter: { userid: 1001 }, '$db': 'test' },
  ...
  ok: 1
}
```

**添加索引之后**

`"stage" : "IXSCAN"`, 基于索引的扫描

```js
> db.c.createIndex({userid:1})
userid_1
> db.c.getIndexes()
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { userid: 1 }, name: 'userid_1' }
]
test> db.c.find({userid:1001}).explain()
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'test.c',
    parsedQuery: { userid: { '$eq': 1001 } },
    indexFilterSet: false,
    queryHash: '56449150',
    planCacheKey: '718E1C3C',
    optimizationTimeMillis: 0,
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    prunedSimilarIndexes: false,
    winningPlan: {
      isCached: false,
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { userid: 1 },
        indexName: 'userid_1',
        isMultiKey: false,
        multiKeyPaths: { userid: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { userid: [ '[1001, 1001]' ] }
      }
    },
    rejectedPlans: []
  },
  command: { find: 'c', filter: { userid: 1001 }, '$db': 'test' },
  ...
  ok: 1
}
```

#### 4.4.2 涵盖的查询

当查询条件和查询的投影仅包含索引字段是, MongoDB 直接从索引返回结果, 而不扫描任何文档或将文档带入内存, 这些覆盖的查询十分有效

其实就是对应 MySQL 的覆盖索引

> https://docs.mongodb.com/manual/core/query-optimization/#covered-query

## 5 MongoDB 聚合操作

MongoDB 中聚合(aggregate)主要用于处理数据(诸如统计平均值，求和等)，并返回计算后的数据结果。

有点类似 **SQL** 语句中的 **count(\*)**。

### 5.1 单一聚合操作

像之前用到的所有操作函数都属于单一作用聚合，例如：find(), findOne(), sort(), skip(), sort(), distinct()等等

而下面在 aggregate() 中使用**多个**管道的操作才叫聚合管道，使用一个管道也叫做单一聚合

### 5.2 aggregate() 方法

aggregate() 方法的基本语法格式如下所示：

```js
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

// aggregate方法中可以放入多个聚合管道进行操作，前一个聚合管道的操作结果交给后一个聚合管道继续处理
db.COLLECTION_NAME.aggregate([
    {聚合管道操作1},
    {聚合管道操作2},
    ...
])
```

准备数据：

```js
[{
  "_id": {
    "$oid": "670d07916db71cc1c7ef9089"
  },
  "title": "MongoDB Overview",
  "description": "MongoDB is no sql database",
  "by_user": "runoob.com",
  "url": "http://www.runoob.com",
  "tags": [
    "mongodb",
    "database",
    "NoSQL"
  ],
  "likes": 100
},
{
  "_id": {
    "$oid": "670d07be6db71cc1c7ef908b"
  },
  "title": "NoSQL Overview",
  "description": "No sql database is very fast",
  "by_user": "runoob.com",
  "url": "http://www.runoob.com",
  "tags": [
    "mongodb",
    "database",
    "NoSQL"
  ],
  "likes": 10
},
{
  "_id": {
    "$oid": "670d07c66db71cc1c7ef908d"
  },
  "title": "Neo4j Overview",
  "description": "Neo4j is no sql database",
  "by_user": "Neo4j",
  "url": "http://www.neo4j.com",
  "tags": [
    "neo4j",
    "database",
    "NoSQL"
  ],
  "likes": 750
}]
```

实例：

```js
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "runoob.com",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
```

### 5.3 聚合操作符

[官方文档](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/aggregation/)

常用聚合操作符如下：

| 表达式 描述 |                                                              | 实例                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| $sum        | 计算总和。                                                   | db.mycol.aggregate([($group : {id : "$by_user", num_tutorial : ($sum : =$likes")]) |
| $avg        | 计算平均值                                                   | db.mycol.aggregate([($group : {id : "$by_user", num_tutorial : ($avg : "$likes")]) |
| $min        | 获取集合中所有文档对应值得最小值。                           | db.mycol.aggregate([$group : {id : "$by_user", num_tutorial : ($min : "$likes")] |
| $max        | 获取集合中所有文档对应值得最大值。                           | db.mycol.aggregate([($group : {id : "$by_user", num_tutorial : ($max : =$likes")]) |
| $push       | 将值加入一个数组中，不会判断是否有重复的值。                 | db.mycol.aggregate([($group : {id : "$by_user", url : ($push:“$url")) |
| $addToSet   | 将值加入一个数组中，会判断是否有重复的值，若相同的值在数 组中已经存在了，则不加入。 | db.mycol.aggregate([{$group : {id : "$by_user", ur] :($addToSet :"$url")) |
| $first      | 根据资源文档的排序获取第一个文档数据。                       | db.mycol.aggregate([($group : {id : "$by_user", first_ur : ($first : “$ur")]) |
| $last       | 根据资源文档的排序获取最后一个文档数据                       | db.mycol.aggregate([$group : {id : "$by_user", last_url : ($last : "$url"]) |

### 5.4 聚合管道

管道在 Unix 和 Linux 中一般用于将当前命令的输出结果作为下一个命令的参数。

MongoDB 的聚合管道将 MongoDB 文档**在一个管道处理完毕后将结果传递给下一个管道处理**。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

上一小节中使用的 `$group` 就是一个聚合管道，这里我们介绍一下聚合框架中常用的几个操作：

- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $group：将集合中的文档分组，可用于统计结果。
- $match：用于过滤数据，只输出符合条件的文档。match 使用 MongoDB 的标准查询操作。
- $count：统计个数。
- $sort：将输入文档排序后输出。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $limit：用来限制 MongoDB 聚合管道返回的文档数。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
   - path: 数组字段的字段路径。
   - includeArrayIndex: 可选。新增一个字段来存放 path 数组中该元素的索引位置。
   - preserveNullAndEmptyArrays: 可选。若为 `true`，如果 `path` 为 null、缺失或空数组，则 `$unwind` 会输出该文档。若为 `false`，如果 `path` 为 null、缺失或空数组，则 `$unwind` 不会输出文档。默认值为 `false`。
- $lookup：用于多表关联查询
- [更多操作符](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/aggregation-pipeline/)

### 5.5 管道操作符实例

1、$project 实例（有投影的作用）

```js
db.article.aggregate(
    { $project : {
        _id : 0 ,
        title : 1 ,
        anthor: "$by_user"
    }}
 );
```

这样的话结果中就只还有_id,tilte 和 author 三个字段了，默认情况下_id 字段是被包含的。

2.$match, $group 实例

```js
db.articles.aggregate( [
                        { $match : { likes : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
```

$match 用于获取分数大于 70 小于或等于 90 记录，然后将符合条件的记录送到下一阶段 $group 管道操作符进行处理。

3.$sort,$skip,$limit 实例：

```js
db.c.aggregate([
  {$sort: {likes: 1}},
  {$skip: 2}, 
  {$limit: 2}
])
```

4.$unwind 实例

```js
db.c.aggregate([
  {$match: {by_user: /runoob/}},
  {$unwind: "$tags"}
])
```

新增一个字段存储 tag 在 tags 数组中的索引下标：

```js
db.c.aggregate([
  {$match: {by_user: /runoob/}},
  {$unwind: {path: "$tags", includeArrayIndex: "arrayIndex"}}
])
```

不过滤 path 数组 为空的情况：

```js
db.c.aggregate([
  {$match: {by_user: /runoob/}},
  {$unwind: {path: "$tags", preserveNullAndEmptyArrays: true}}
])
```

5.$lookup 实例：

```js
{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}
```

该操作对应于如下伪 SQL 语句：

```js
SELECT *, (
   SELECT ARRAY_AGG(*)
   FROM <collection to join>
   WHERE <foreignField> = <collection.localField>
) AS <output array field>
FROM collection;
```

假设现在有 customer 和 order 两个 collection 集合：

```js
db.customer.aggregate([
  {$lookup: {
                from: "order", 
                localField: "customerCode", 
                foreignField: "customerCode", 
                as: "customerOrder"
        }}
])
```

## References

- https://www.mongodb.com/zh-cn/docs/
- https://www.runoob.com/mongodb/mongodb-aggregate.html
- https://www.bilibili.com/video/av59604756
- https://www.bilibili.com/video/BV1bJ411x7mq
- https://www.youtube.com/watch?v=-56x56UppqQ