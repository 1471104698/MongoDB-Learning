# MongoDB



## 1、MongoDB 基本概念

MongoDB 和 mysql 之间的概念映射关系：

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                                                    |
| ------------ | ---------------- | ------------------------------------------------------------ |
| database     | database         | 数据库                                                       |
| table        | collection       | 表/集合                                                      |
| row          | document         | 记录行/文档                                                  |
| column       | field            | 字段                                                         |
| index        | index            | 索引                                                         |
| table joins  |                  | 表连接,MongoDB不支持                                         |
| primary key  | primary key      | 主键,MongoDB自动将 "_id" 字段设置为主键，插入的时候会自动生成 |



MongoDB 是一个文档型数据库，所有的数据都以文档的形式存储的，这个所谓的文档就是 bson，可以理解为以 json 的格式存储

比如一个 collection 下的数据：

```json
{ 
    "_id" : ObjectId("60d1d35c361035b3c7699763"), 
    "id" : 1.0, 
    "name" : "自动触发插件3", 
    "desc" : "自动触发插件描述3", 
    "trigger_type" : 2.0, 
    "feedback_id" : "B", 
    "createAt" : "2016-10-13", 
    "project_id" : "wesee", 
    "user_id" : "milkouyang"
}
{ 
    "_id" : ObjectId("60d206b2361035b3c7699764"), 
    "id" : 2.0, 
    "addr" : "test", 
    "age" : "test", 
    "test" : "test"
}
```

其中 "_id" 是固定主键，在插入的时候自动生成，我们也可以自己在插入的时候指定 "_id"，不过一般不会这么做

同时我们可以看出，同一个 collection 下的数据它们之间的字段可以是不同的，即数据之间没有任何的关联性，它们都是 bson，简单讲就是存储的时候传入的是什么就直接存储什么，不会去理会它的字段结构，而 mysql 的话每个表的结构是固定的，插入的数据必须严格按照表的结构来。

上面这两个数据之间没有任何关联，可以理解为它们单单只是存储在同一个 collection 而已



## 2、语法

[MongoDB的常用语法](https://www.cnblogs.com/leskang/p/6000852.html)

[MongoDB 语法](https://juejin.cn/post/6875951740798107662#heading-9)



### collection 创建

```sql
//显示创建 collection
db.createCollection("collection_1")

// 隐式创建集合，插入一条数据，如果对应的 collection 不存在，那么会先创建该 collection
db.collection_1.insert(
	{name:"zhangsan", age:14}
)

// 获取 collection 的数据量
db.collection_1.count()

// 删除集合
db.collection_1.drop()
```



### 基本 eq 查询

```sql
-- 查询一条数据，即所有数据中的第一条数据
db.collection_1.findOne()
-- 查询所有数据
db.collection_1.find()

-- 查找 name = zhangsan and age = 12 的数据
-- 注意，数据之间的字段可以是不同的，不一定所有的数据都有 name 和 age 字段，因此它这里会先根据 name 查找所有具有该字段的数据
-- 然后再从这些数据中找 age 字段返回，对于有 name 字段而没有 age 字段的数据，那么它就不会返回 age，但是还是会返回，返回 _id，其他有 age 的返回 _id 和 age
db.collection_1.find(
	{name:"zhangsan"， age:12}
)

-- 查询 name = zhangsan 的数据， select 的字段为 age，第一个 {} 表示查询条件，第二个 {} 表示返回字段
db.collection_1.find(
	{name:"zhangsan"}, {age: 1}
)

-- 查询 name = zhangsan 的数据， select 返回除了 age 和 addr 之外的所有字段
-- 第二个 {} 中，key 是返回字段的处理，value 如果是 0，表示不返回该字段，其他任意数字都表示返回该字段
db.collection_1.find(
	{name:"zhangsan"}, {age: 0, addr: 0}
)

-- 查询 name = zhangsan and age = 12 的数据， select 返回 age 和 addr
db.collection_1.find(
	{name:"zhangsan", age: 12}, {age: 1, addr: 1}
)

-- 报错，第二个 {} 中 0 和 非 0 不能同时存在
db.collection_1.find(
	{name:"zhangsan"}, {age: 0, addr: 1}
)

-- 查询所有数据，select 返回 age 和 addr
db.collection_1.find(
	{}, {age: 1, addr: 1}
)

-- 查询 collection 的第一条数据
db.collection_1.findOne()

-- 查询 name = zhangsan 的第一条数据，select 返回 age
db.collection_1.findOne(
	{name:"zhangsan"}, {age:1}
)

-- findOne() 其他操作跟 find() 一致，这里不赘述
```



### 特殊查询

```sql
-- 查询 name 中 contains 同时包含 "zhang" 和 "a" 的数据，比如 zhangsan，就同时包含了这两个字符串
-- 这里涉及到 {} 嵌套，外层 {name: {}}，这个 name 是外层的 key，指定的是查询的字段， value 是内层 {$all, []string{"zhang", "a"}}
-- 而在内层 {} 中 key 是 "$all" 表示对应执行的操作，value 是操作的值，比如这里 “$all” 是操作，[]string{"zhang", "a"} 是操作值，内层 {} 作用于外层 {} 指定的字段，即 name
-- 可以转换为 where name.contains(“zhang") and name.contains(”a")
db.collection_1.find(
	{name:{$all, ["zhang", "a"]}}
)

-- in 操作，即存在其中一个即可
db.collection_1.find(
	{name:{$in, ["zhang", "a"]}}
)

-- not in 查找不包含其中任何一个的数据
db.collection_1.find(
	{name:{$nin, ["zhang", "a"]}}
)

-- or 操作，查询 name = zhangsan 或者 age in (12, 14) 的数据，$or 是 {} 中的 key，value 对应的是一个数组，每个元素都是 {}
db.collection_1.find(
	{
    	$or:[
            {name:"zhangsan"}, {age: {$in: [12, 14]}}
        ]
    }
)

-- 查询 name != zhangsan and age not in (12, 14) 的数据
db.collection_1.find(
	{
    	$nor:[
            {name:"zhangsan"}, {age: {$in: [12, 14]}}
        ]
    }
)

-- 分页操作，skip = offset
db.collection_1.find().skip(10).limit(20)

-- 排序，先 find() 查询数据，再 sort() 排序， 这里是根据 id 进行降序排序，1 表示升序、-1 表示降序，其他的值会报错
db.collection_1.find().sort({id:-1})

-- 查找所有存在 name 字段的数据，这里其中 exists 后面跟 非0 表示查找存在的数据，0 表示查找不存在该字段的数据
db.collection_1.find({name:{$exists:1}})
```





### 更新

```sql
-- 更新一条数据
db.collection_1.updateOne() 
-- 更新多条数据
db.collection_1.updateMany() 
-- 替换 collection 中的某条数据，这里不做讨论，因为不知道具体的使用场景
db.collection_1.replaceOne() 

-- 将 name = "zhangsan" and age = 12 的一条数据更新 name = "lisi"
-- 如果存在多条满足条件的数据，那么更新的数据可以理解为 db.collection_1.findOne({name:"zhangsan", age:12}) 查询出来的那一条数据
db.collection_1.updateOne(
	{name:"zhangsan", age:12},
    {$set:{name:"lisi"}}
) 

-- 将所有 name = "zhangsan" and age = 12 的数据更新 name = "lisi"
db.collection_1.updateMany(
	{name:"zhangsan", age:12},
    {$set:{name:"lisi"}}
) 

-- 实际上这里也可以存在各种的操作，比如更新存在 test 字段的数据、更新 age not in (12, 14) 的数据等，都是一样的逻辑，这里就不赘述了
```



### 删除

```sql
-- 删除一条数据
db.collection.deleteOne()
-- 删除多条数据
db.collection.deleteMany()


-- 删除 name = "zhangsan" and age = 12 的数据，只删除一条
-- 原理同 updateOne()
db.collection_1.deleteOne({name:"zhangsan", age:12})

-- 删除 name = "zhangsan" and age = 12 的所有数据
db.collection_1.deleteMany({name:"zhangsan", age:12})

-- 实际上这里也可以存在各种的操作，比如删除存在 test 字段的数据、删除 age not in (12, 14) 的数据等，都是一样的逻辑，这里就不赘述了
```



### 插入

```sql
-- 插入一条数据
db.collection_1.insertOne()
-- 插入多条数据
db.collection_1.insertMany()

-- 插入一条 id = 1, name = "zhangsan", age = 12 的数据
db.collection_1.insertOne(
	{id:1, name:"zhangsan",age:12}
)

-- 插入一条 id = 1, name = "zhangsan", age = 12 的数据 和 一条 id = 1，test = "test", addr = "beijing" 的数据
-- 数据需要使用 [] 括起来，表示这个是一个数组，每个元素就是 {}，表示一条数据，它们之间没有任何关联，不需要具有相同的字段
db.collection_1.insertMany(
	[
        {id:1, name:"zhangsan",age:12},
    	{id:2, test:"test",addr:"beijing"}
    ]
)
```



### 创建索引

```sql
-- keys 表示创建的索引，options 表示索引创建可选项
db.collection_1.createIndex(keys, options)
-- 获取 collection 所有的索引
db.collection_1.getIndexes()
-- 删除 collection 所有的索引，但不会删除 _id 主键索引
db.collection_1.dropIndexes()
-- 删除 collection 指定的索引，这里只需要传入索引名的字符串
db.collection_1.dropIndexe("索引名")


-- 为 name 创建索引， 1 表示升序存储，-1 表示降序存储
db.collection_1.createIndex({name:1})

-- 为 name 和 age 创建联合索引，其中 name 升序，age 降序（注意按照字段区分度的顺序来建立）
db.collection_1.createIndex({name:1, age:-1})

-- 为 name 创建索引，同时定义为唯一索引，即 name 每个值在 collection 数据中唯一
db.collection_1.createIndex({name:1}, {unique:true})
```

createIndex() 接收 options 可选参数，常用 options 如下：

| Parameter          | Type          | Description                                                  |
| :----------------- | :------------ | :----------------------------------------------------------- |
| background         | Boolean       | 告知 mongodb 创建索引时是否阻塞其它操作，background 可指定以后台方式创建索引， "background" 默认值为**false**，即默认阻塞。 |
| unique             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| name               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| sparse             | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
| v                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights            | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language   | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override  | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |



## 3、MongoDB Driver 使用

[mongdo driver API 使用](https://mongoing.com/archives/27257)

### bson 类型

> #### bson.E：

```go
等同于原生的 {}，它是以 key-value 的形式来表示 {} 的

结构为：
type E struct {
	Key   string
	Value interface{}
}

使用例子：
	bson.E{Key:"name", Value:"zhangsan"}
== 
	{name:"zhangsan"}

	bson.E{Key:"name", Value:bson.E{Key:"$all", Value: []string{"zhang", "a"}}} 
== 
	{name:{$all:["zhang","a"]}}
```

> #### bson.D

```go
用于存储 bson.E 的切片，[]bson.E，内部的 E 是有序的，根据 bson.E 的顺序就是插入的顺序，当需要执行一些有先后顺序的命令的使用就需要使用 bson.D

结构为：
type D []E

例子：
bson.D{{"foo", "bar"}, {"hello", "world"}, {"pi", 3.14159}}
```

> #### bson.M

```go
它是一个 map，可以存储任意 bson 类型，内部的元素是无序的，每次插入都是经过 hash 运算然后插到对应的索引位置上的，跟插入顺序无关

结构为：
type M map[string]interface{}

例子：
bson.M{"foo": "bar", "hello": "world", "pi": 3.14159}	// 每个 key:value 都可以当作一个 bson.E，当然内部也可以存储 bson.D
```

> #### bson.A

```go
它是一个 slice，可以存储任意 bson 类型

结构为：
type A []interface{}

例子：
bson.A{"bar", "world", 3.14159, bson.D{{"qux", 12345}}}
```



### 连接数据库

建立并获取数据库连接：

```go
// Set client options
clientOptions := options.Client().ApplyURI("MongoDB://localhost:27017")

// Connect to MongoDB
client, err := mongo.Connect(context.TODO(), clientOptions)

// 获取 MongoDB 数据库 "test" 中的 collection 集合 "user" 的一条连接
collection := client.Database("test").Collection("user")
```



### 插入

存在以下结构体：

```go
type User struct {
    ID int32 `bson:"id, omitempty"`
    Age int32 `bson:"age, omitempty"`
    Name string `bson:"name, omitempty"`
    Addr string `bson:"addr, omitempty`
}
```



```go
// InsertOne() 插入单个数据
user1 := User{ID:1, Name:"zhangsan", Age:12}
insertResult, _ := collection.InsertOne(context.TODO(), user1)
// insertResult.InsertedID 返回的是 MongoDB 自动生成的 _id 的值，它是唯一主键，我们自己定的 id 不是唯一主键
fmt.Println("新插入的数据的 id 为：", insertResult.InsertedID)


// InsertMany() 插入多个数据
user1 := User{ID:1, Name:"zhangsan", Age:12}
user2 := User{ID:2, Name:"zhangsan", Addr:"beijing"}
// 注意这里设置的切片类型必须是 Interface{}，因为接口接收的就是 []interface{}，在 golang 中，[]interface{} 作用并不等同于 interface{}，它不能去接收 []string 之类的
users := []interface{}{user1, user2}
insertManyResult, _ := collection.InsertMany(context.TODO(), users)
fmt.Println("新插入的数据的 id 集合为: ", insertManyResult.InsertedIDs)
```



### 更新

```go
// 构造需要 update 的数据，这里是将 set age=age+1, count=count+2, name=zhangsan
update := bson.D{
    {"$inc", bson.D{
        {"age", 1},
        {"count", 2},
    }},
    {"$set",
     bson.E{"name", "zhangsan"},
    },
}
// 构造查询条件，这里是 where name = "list" and age = 12
filter := bson.D{
    {"name", "lisi"},
    {"age", 12},
}

// UpdateOne 只会更新一条数据，如果有多条那么更新的是 findOne() 的那条
updateResult, _ := collection.UpdateOne(context.TODO(), filter, update)
fmt.Printf("匹配到的数据个数：%v， 实际修改的数据个数：%v\n", updateResult.MatchedCount, updateResult.ModifiedCount)

// UpdateMany 更新所有满足条件的数据
updateResult, _ := collection.UpdateMany(context.TODO(), filter, update)
fmt.Printf("匹配到的数据个数：%v， 实际修改的数据个数：%v\n", updateResult.MatchedCount, updateResult.ModifiedCount)
```



### 查询

```go
// 查询条件的构造跟上面 更新 中的 filter 是一致的，这里不做赘述


// 查询一条数据，并将查询结果解码到 user 中（可以理解为反序列化）
user = new(User)
_ = collection.FindOne(context.TODO(), filter).Decode(user)


// 查询多条数据
// filter 是查询条件，这里传入 bson.D{{}} 表示不存在 where 条件
// filterOptions 设置一些数据操作，比如 limit、skip 等，这个看框架具体实现，一般会封装成结构体，然后对外提供方法直接调用，比如 opts.setLimit(10)
filter := bson.D{{}}
// Find() 会返回一个游标（类似于迭代器）cur
cur, _ = collection.Find(context.TODO(), filter, filterOptions).Decode(user)
// 从 cur 中获取数据的方式一：使用 for 来通过 Next() 来让它指向下一条数据，然后调用它的 Decode() 进行反序列化
var users []*User
for cur.Next(context.TODO()) {
    user = new(User)
    _ := cur.Decode(user)
    users = append(users, user)
}
// 从 cur 中获取数据的方式二：直接调用 All()
var users []*User
cur.All(context.TODO(), &users)
```



### 删除

```go
// 删除一条数据，只传 filter 作为删除条件
collection.DeleteOne(filter)
// 删除多条数据，只传 filter 作为删除条件
collection.DeleteMany(filter)

// DeleteMany() 删除多条数据，删除条件为 bson.D{{}}，表示删除所有数据
filter := bson.D{{}}
deleteResult, err := collection.DeleteMany(context.TODO(), filter)
fmt.Printf("删除的数据个数：%v\n", deleteResult.DeletedCount)

// 基本这个删除操作根据经验就知道怎么做，这里就不赘述
```

