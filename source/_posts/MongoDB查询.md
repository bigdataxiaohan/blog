---
title: MongoDB查询
date: 2018-12-09 16:21:04
tags: MongoDB
categories: 数据库
---

## find查询

MongoDB中使用find来进行查询。査询就是返回一个集合中文档的子集，子集合的范围从0个文档到整个集合。find的第一个参数决定了要返回哪些文档，其形式也是一个文档，说明要执行的査询细节。

### 查询所有数据

空的査询文档{}会匹配集合的全部内容。要是不指定査询文档，默认就是{}。	

```json
> db.user.find().pretty()
{
        "_id" : ObjectId("5c0cd66df792e1bdd43e96f6"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cd67ef792e1bdd43e96f7"),
        "username" : "hello",
        "nickname" : "Hello",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cd68bf792e1bdd43e96f8"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}

```

当我们查询文档中加入键值对的时候,就意味着限定了查询的条件,对于绝大部分的数据类型来说,这种方式简单明了,整数匹配整数,布尔匹配布尔类型,字符串匹配字符串,查询简单类型,只要制定好要查找的值就行,如果你想查找所"username"为“foo”的文档,直接可以把键值对写入查询文档.

```json
> db.user.find({username:"foo"}).pretty()
{
        "_id" : ObjectId("5c0cd66df792e1bdd43e96f6"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cd68bf792e1bdd43e96f8"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
```

可以向查询文档加入多个键值对来将多个查询条件组合到一起,如查询username为hello  password为"123" 的用户

```json
> db.user.find({ "username" : "hello",password:"123"}).pretty()
{
        "_id" : ObjectId("5c0cd67ef792e1bdd43e96f7"),
        "username" : "hello",
        "nickname" : "Hello",
        "password" : "123"
}
```

### 指定返回值

通常情况下我们查询数据的时候,并不需要将文档中所有的键值都返回.这种情况下我们可以通过find的第二个参数来指定想要返回的键.这样做机会节省传输数据量,又能节省客户端解码文档的时间和内存消耗。如果我们只想要username的相关数据,不要_id、nickname、password 可以这么写

db.文档.find({条件},{要查询的键})

```json
> db.user.find({},{_id:0,username:1})
第一个参数是一个空对象,表示从全部的数据中查询,我们可以看到这里没有写nickname=o和password=0但是也没有显示出来,如果_id:0不屑会显示出来_id的信息的
{ "username" : "foo" }
{ "username" : "hello" }
{ "username" : "foo" }
```

### 查询条件

| 运算符 | 对应到mysql的运算符          |
| ------ | ---------------------------- |
| $gt    | >                            |
| $gte   | >=                           |
| $in    | in                           |
| $lt    | <                            |
| $lte   | <=                           |
| $ne    | !=                           |
| $nin   | not   in                     |
| $all   | 无对应项,指数组所有单元匹配. |

### 元数据

```json
[{"goods_id":1,"cat_id":4,"goods_name":"KD876","goods_number":1,"click_count":7,"shop_price":1388.00,"add_time":1240902890},{"goods_id":4,"cat_id":8,"goods_name":"\u8bfa\u57fa\u4e9aN85\u539f\u88c5\u5145\u7535\u5668","goods_number":17,"click_count":0,"shop_price":58.00,"add_time":1241422402},{"goods_id":3,"cat_id":8,"goods_name":"\u8bfa\u57fa\u4e9a\u539f\u88c55800\u8033\u673a","goods_number":24,"click_count":3,"shop_price":68.00,"add_time":1241422082},{"goods_id":5,"cat_id":11,"goods_name":"\u7d22\u7231\u539f\u88c5M2\u5361\u8bfb\u5361\u5668","goods_number":8,"click_count":3,"shop_price":20.00,"add_time":1241422518},{"goods_id":6,"cat_id":11,"goods_name":"\u80dc\u521bKINGMAX\u5185\u5b58\u5361","goods_number":15,"click_count":0,"shop_price":42.00,"add_time":1241422573},{"goods_id":7,"cat_id":8,"goods_name":"\u8bfa\u57fa\u4e9aN85\u539f\u88c5\u7acb\u4f53\u58f0\u8033\u673aHS-82","goods_number":20,"click_count":0,"shop_price":100.00,"add_time":1241422785},{"goods_id":8,"cat_id":3,"goods_name":"\u98de\u5229\u6d669@9v","goods_number":1,"click_count":9,"shop_price":399.00,"add_time":1241425512},{"goods_id":9,"cat_id":3,"goods_name":"\u8bfa\u57fa\u4e9aE66","goods_number":4,"click_count":20,"shop_price":2298.00,"add_time":1241511871},{"goods_id":10,"cat_id":3,"goods_name":"\u7d22\u7231C702c","goods_number":7,"click_count":11,"shop_price":1328.00,"add_time":1241965622},{"goods_id":11,"cat_id":3,"goods_name":"\u7d22\u7231C702c","goods_number":1,"click_count":0,"shop_price":1300.00,"add_time":1241966951},{"goods_id":12,"cat_id":3,"goods_name":"\u6469\u6258\u7f57\u62c9A810","goods_number":8,"click_count":13,"shop_price":983.00,"add_time":1245297652}]

[{"goods_id":13,"cat_id":3,"goods_name":"\u8bfa\u57fa\u4e9a5320 XpressMusic","goods_number":8,"click_count":13,"shop_price":1311.00,"add_time":1241967762},{"goods_id":14,"cat_id":4,"goods_name":"\u8bfa\u57fa\u4e9a5800XM","goods_number":1,"click_count":6,"shop_price":2625.00,"add_time":1241968492},{"goods_id":15,"cat_id":3,"goods_name":"\u6469\u6258\u7f57\u62c9A810","goods_number":3,"click_count":8,"shop_price":788.00,"add_time":1241968703},{"goods_id":16,"cat_id":2,"goods_name":"\u6052\u57fa\u4f1f\u4e1aG101","goods_number":0,"click_count":3,"shop_price":823.33,"add_time":1241968949},{"goods_id":17,"cat_id":3,"goods_name":"\u590f\u65b0N7","goods_number":1,"click_count":2,"shop_price":2300.00,"add_time":1241969394},{"goods_id":18,"cat_id":4,"goods_name":"\u590f\u65b0T5","goods_number":1,"click_count":0,"shop_price":2878.00,"add_time":1241969533},{"goods_id":19,"cat_id":3,"goods_name":"\u4e09\u661fSGH-F258","goods_number":12,"click_count":7,"shop_price":858.00,"add_time":1241970139},{"goods_id":20,"cat_id":3,"goods_name":"\u4e09\u661fBC01","goods_number":12,"click_count":14,"shop_price":280.00,"add_time":1241970417},{"goods_id":21,"cat_id":3,"goods_name":"\u91d1\u7acb A30","goods_number":40,"click_count":4,"shop_price":2000.00,"add_time":1241970634},{"goods_id":22,"cat_id":3,"goods_name":"\u591a\u666e\u8fbeTouch HD","goods_number":1,"click_count":15,"shop_price":5999.00,"add_time":1241971076}]

[{"goods_id":23,"cat_id":5,"goods_name":"\u8bfa\u57fa\u4e9aN96","goods_number":8,"click_count":17,"shop_price":3700.00,"add_time":1241971488},{"goods_id":24,"cat_id":3,"goods_name":"P806","goods_number":100,"click_count":35,"shop_price":2000.00,"add_time":1241971981},{"goods_id":25,"cat_id":13,"goods_name":"\u5c0f\u7075\u901a\/\u56fa\u8bdd50\u5143\u5145\u503c\u5361","goods_number":2,"click_count":0,"shop_price":48.00,"add_time":1241972709},{"goods_id":26,"cat_id":13,"goods_name":"\u5c0f\u7075\u901a\/\u56fa\u8bdd20\u5143\u5145\u503c\u5361","goods_number":2,"click_count":0,"shop_price":19.00,"add_time":1241972789},{"goods_id":27,"cat_id":15,"goods_name":"\u8054\u901a100\u5143\u5145\u503c\u5361","goods_number":2,"click_count":0,"shop_price":95.00,"add_time":1241972894},{"goods_id":28,"cat_id":15,"goods_name":"\u8054\u901a50\u5143\u5145\u503c\u5361","goods_number":0,"click_count":0,"shop_price":45.00,"add_time":1241972976},{"goods_id":29,"cat_id":14,"goods_name":"\u79fb\u52a8100\u5143\u5145\u503c\u5361","goods_number":0,"click_count":0,"shop_price":90.00,"add_time":1241973022},{"goods_id":30,"cat_id":14,"goods_name":"\u79fb\u52a820\u5143\u5145\u503c\u5361","goods_number":9,"click_count":1,"shop_price":18.00,"add_time":1241973114},{"goods_id":31,"cat_id":3,"goods_name":"\u6469\u6258\u7f57\u62c9E8 ","goods_number":1,"click_count":5,"shop_price":1337.00,"add_time":1242110412},{"goods_id":32,"cat_id":3,"goods_name":"\u8bfa\u57fa\u4e9aN85","goods_number":4,"click_count":9,"shop_price":3010.00,"add_time":1242110760}]

```

```json
> db.goods.insert([{"goods_id":1,"cat_id":4,"goods_name":"KD876","goods_number":1,"click_count":7,"shop_price":1388.00,"add_time":1240902890},{"goods_id":4,"cat_id":8,"goods_name":"\u8bfa\u57fa\u4e9aN85\u539f\u88c5\u5145\u7535\u5668","goods_number":17,"click_count":0,"shop_price":58.00,"add_time":1241422402},{"goods_id":3,"cat_id":8,"goods_name":"\u8bfa\u57fa\u4e9a\u539f\u88c55800\u8033\u673a","goods_number":24,"click_count":3,"shop_price":68.00,"add_time":1241422082},{"goods_id":5,"cat_id":11,"goods_name":"\u7d22\u7231\u539f\u88c5M2\u5361\u8bfb\u5361\u5668","goods_number":8,"click_count":3,"shop_price":20.00,"add_time":1241422518},{"goods_id":6,"cat_id":11,"goods_name":"\u80dc\u521bKINGMAX\u5185\u5b58\u5361","goods_number":15,"click_count":0,"shop_price":42.00,"add_time":1241422573},{"goods_id":7,"cat_id":8,"goods_name":"\u8bfa\u57fa\u4e9aN85\u539f\u88c5\u7acb\u4f53\u58f0\u8033\u673aHS-82","goods_number":20,"click_count":0,"shop_price":100.00,"add_time":1241422785},{"goods_id":8,"cat_id":3,"goods_name":"\u98de\u5229\u6d669@9v","goods_number":1,"click_count":9,"shop_price":399.00,"add_time":1241425512},{"goods_id":9,"cat_id":3,"goods_name":"\u8bfa\u57fa\u4e9aE66","goods_number":4,"click_count":20,"shop_price":2298.00,"add_time":1241511871},{"goods_id":10,"cat_id":3,"goods_name":"\u7d22\u7231C702c","goods_number":7,"click_count":11,"shop_price":1328.00,"add_time":1241965622},{"goods_id":11,"cat_id":3,"goods_name":"\u7d22\u7231C702c","goods_number":1,"click_count":0,"shop_price":1300.00,"add_time":1241966951},{"goods_id":12,"cat_id":3,"goods_name":"\u6469\u6258\u7f57\u62c9A810","goods_number":8,"click_count":13,"shop_price":983.00,"add_time":1245297652}])
> db.goods.insert([{"goods_id":13,"cat_id":3,"goods_name":"\u8bfa\u57fa\u4e9a5320 XpressMusic","goods_number":8,"click_count":13,"shop_price":1311.00,"add_time":1241967762},{"goods_id":14,"cat_id":4,"goods_name":"\u8bfa\u57fa\u4e9a5800XM","goods_number":1,"click_count":6,"shop_price":2625.00,"add_time":1241968492},{"goods_id":15,"cat_id":3,"goods_name":"\u6469\u6258\u7f57\u62c9A810","goods_number":3,"click_count":8,"shop_price":788.00,"add_time":1241968703},{"goods_id":16,"cat_id":2,"goods_name":"\u6052\u57fa\u4f1f\u4e1aG101","goods_number":0,"click_count":3,"shop_price":823.33,"add_time":1241968949},{"goods_id":17,"cat_id":3,"goods_name":"\u590f\u65b0N7","goods_number":1,"click_count":2,"shop_price":2300.00,"add_time":1241969394},{"goods_id":18,"cat_id":4,"goods_name":"\u590f\u65b0T5","goods_number":1,"click_count":0,"shop_price":2878.00,"add_time":1241969533},{"goods_id":19,"cat_id":3,"goods_name":"\u4e09\u661fSGH-F258","goods_number":12,"click_count":7,"shop_price":858.00,"add_time":1241970139},{"goods_id":20,"cat_id":3,"goods_name":"\u4e09\u661fBC01","goods_number":12,"click_count":14,"shop_price":280.00,"add_time":1241970417},{"goods_id":21,"cat_id":3,"goods_name":"\u91d1\u7acb A30","goods_number":40,"click_count":4,"shop_price":2000.00,"add_time":1241970634},{"goods_id":22,"cat_id":3,"goods_name":"\u591a\u666e\u8fbeTouch HD","goods_number":1,"click_count":15,"shop_price":5999.00,"add_time":1241971076}])
> db.goods.insert([{"goods_id":23,"cat_id":5,"goods_name":"\u8bfa\u57fa\u4e9aN96","goods_number":8,"click_count":17,"shop_price":3700.00,"add_time":1241971488},{"goods_id":24,"cat_id":3,"goods_name":"P806","goods_number":100,"click_count":35,"shop_price":2000.00,"add_time":1241971981},{"goods_id":25,"cat_id":13,"goods_name":"\u5c0f\u7075\u901a\/\u56fa\u8bdd50\u5143\u5145\u503c\u5361","goods_number":2,"click_count":0,"shop_price":48.00,"add_time":1241972709},{"goods_id":26,"cat_id":13,"goods_name":"\u5c0f\u7075\u901a\/\u56fa\u8bdd20\u5143\u5145\u503c\u5361","goods_number":2,"click_count":0,"shop_price":19.00,"add_time":1241972789},{"goods_id":27,"cat_id":15,"goods_name":"\u8054\u901a100\u5143\u5145\u503c\u5361","goods_number":2,"click_count":0,"shop_price":95.00,"add_time":1241972894},{"goods_id":28,"cat_id":15,"goods_name":"\u8054\u901a50\u5143\u5145\u503c\u5361","goods_number":0,"click_count":0,"shop_price":45.00,"add_time":1241972976},{"goods_id":29,"cat_id":14,"goods_name":"\u79fb\u52a8100\u5143\u5145\u503c\u5361","goods_number":0,"click_count":0,"shop_price":90.00,"add_time":1241973022},{"goods_id":30,"cat_id":14,"goods_name":"\u79fb\u52a820\u5143\u5145\u503c\u5361","goods_number":9,"click_count":1,"shop_price":18.00,"add_time":1241973114},{"goods_id":31,"cat_id":3,"goods_name":"\u6469\u6258\u7f57\u62c9E8 ","goods_number":1,"click_count":5,"shop_price":1337.00,"add_time":1242110412},{"goods_id":32,"cat_id":3,"goods_name":"\u8bfa\u57fa\u4e9aN85","goods_number":4,"click_count":9,"shop_price":3010.00,"add_time":1242110760}])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 10,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})


> db.goods.find().count()
31
```

#### 查询区间

#####  $in

```json
查询商品价格3000-5000的
> db.goods.find({shop_price:{$gte:3000,$lte:5500}}).pretty()
{
        "_id" : ObjectId("5c0ce0c1bde03aeb84cf5a9c"),
        "goods_id" : 23,
        "cat_id" : 5,
        "goods_name" : "诺基亚N96",
        "goods_number" : 8,
        "click_count" : 17,
        "shop_price" : 3700,
        "add_time" : 1241971488
}
{
        "_id" : ObjectId("5c0ce0c1bde03aeb84cf5aa5"),
        "goods_id" : 32,
        "cat_id" : 3,
        "goods_name" : "诺基亚N85",
        "goods_number" : 4,
        "click_count" : 9,
        "shop_price" : 3010,
        "add_time" : 1242110760
}
>
```

\$in可以用来查询一个键的多个值,对于但意见要是由多个值与其匹配的话就要用\$in 

```json
# 查询商品是编号是4和3的
> db.goods.find({goods_number:{$in:[4,3]}},{_id:0,goods_number:1,goods_name:1,shop_price:1}).pretty()
{ "goods_name" : "诺基亚E66", "goods_number" : 4, "shop_price" : 2298 }
{ "goods_name" : "摩托罗拉A810", "goods_number" : 3, "shop_price" : 788 }
{ "goods_name" : "诺基亚N85", "goods_number" : 4, "shop_price" : 3010 }

### 查询商品编号不是4和3的

> db.goods.find({goods_number:{$nin:[4,3]}},{_id:0,goods_number:1,goods_name:1,shop_price:1})
{ "goods_name" : "KD876", "goods_number" : 1, "shop_price" : 1388 }
{ "goods_name" : "诺基亚N85原装充电器", "goods_number" : 17, "shop_price" : 58 }
{ "goods_name" : "诺基亚原装5800耳机", "goods_number" : 24, "shop_price" : 68 }
{ "goods_name" : "索爱原装M2卡读卡器", "goods_number" : 8, "shop_price" : 20 }
{ "goods_name" : "胜创KINGMAX内存卡", "goods_number" : 15, "shop_price" : 42 }
{ "goods_name" : "诺基亚N85原装立体声耳机HS-82", "goods_number" : 20, "shop_price" : 100 }
{ "goods_name" : "飞利浦9@9v", "goods_number" : 1, "shop_price" : 399 }
{ "goods_name" : "索爱C702c", "goods_number" : 7, "shop_price" : 1328 }
{ "goods_name" : "索爱C702c", "goods_number" : 1, "shop_price" : 1300 }
{ "goods_name" : "摩托罗拉A810", "goods_number" : 8, "shop_price" : 983 }
{ "goods_name" : "诺基亚5320 XpressMusic", "goods_number" : 8, "shop_price" : 1311 }
{ "goods_name" : "诺基亚5800XM", "goods_number" : 1, "shop_price" : 2625 }
{ "goods_name" : "恒基伟业G101", "goods_number" : 0, "shop_price" : 823.33 }
{ "goods_name" : "夏新N7", "goods_number" : 1, "shop_price" : 2300 }
{ "goods_name" : "夏新T5", "goods_number" : 1, "shop_price" : 2878 }
{ "goods_name" : "三星SGH-F258", "goods_number" : 12, "shop_price" : 858 }
{ "goods_name" : "三星BC01", "goods_number" : 12, "shop_price" : 280 }
{ "goods_name" : "金立 A30", "goods_number" : 40, "shop_price" : 2000 }
{ "goods_name" : "多普达Touch HD", "goods_number" : 1, "shop_price" : 5999 }
{ "goods_name" : "诺基亚N96", "goods_number" : 8, "shop_price" : 3700 }
Type "it" for more
> it
{ "goods_name" : "P806", "goods_number" : 100, "shop_price" : 2000 }
{ "goods_name" : "小灵通/固话50元充值卡", "goods_number" : 2, "shop_price" : 48 }
{ "goods_name" : "小灵通/固话20元充值卡", "goods_number" : 2, "shop_price" : 19 }
{ "goods_name" : "联通100元充值卡", "goods_number" : 2, "shop_price" : 95 }
{ "goods_name" : "联通50元充值卡", "goods_number" : 0, "shop_price" : 45 }
{ "goods_name" : "移动100元充值卡", "goods_number" : 0, "shop_price" : 90 }
{ "goods_name" : "移动20元充值卡", "goods_number" : 9, "shop_price" : 18 }
{ "goods_name" : "摩托罗拉E8 ", "goods_number" : 1, "shop_price" : 1337 }

```

#####  $OR

MongoDB中由两种方式进行OR查询.“\$sin”和(\$nin)可以用来查询多个键值\$or()更通用一些,用来完成多个任意给定的值,如查询商品价格大于5000 或者商品id为3的商品

```json
> db.goods.find({$or:[{shop_price:{$gte:5000}},{goods_number:3}]},{_id:0,goods_name:1,goods_number:1,shop_price:1})
{ "goods_name" : "摩托罗拉A810", "goods_number" : 3, "shop_price" : 788 }
{ "goods_name" : "多普达Touch HD", "goods_number" : 1, "shop_price" : 5999 }
```

##### $not

\$not是元条件句,即可以用在其他条件之上(正则和文档).\$not和\$nin 的区别是\$not可以用在任何地方\$nin只能用到几何上.例如查询商品名有没有诺基亚的

```json
> db.goods.find({goods_name:{$not:/诺基亚/}},{_id:0,goods_name:1})
{ "goods_name" : "KD876" }
{ "goods_name" : "索爱原装M2卡读卡器" }
{ "goods_name" : "胜创KINGMAX内存卡" }
{ "goods_name" : "飞利浦9@9v" }
{ "goods_name" : "索爱C702c" }
{ "goods_name" : "索爱C702c" }
{ "goods_name" : "摩托罗拉A810" }
{ "goods_name" : "摩托罗拉A810" }
{ "goods_name" : "恒基伟业G101" }
{ "goods_name" : "夏新N7" }
{ "goods_name" : "夏新T5" }
{ "goods_name" : "三星SGH-F258" }
{ "goods_name" : "三星BC01" }
{ "goods_name" : "金立 A30" }
{ "goods_name" : "多普达Touch HD" }
{ "goods_name" : "P806" }
{ "goods_name" : "小灵通/固话50元充值卡" }
{ "goods_name" : "小灵通/固话20元充值卡" }
{ "goods_name" : "联通100元充值卡" }
{ "goods_name" : "联通50元充值卡" }
Type "it" for more
> it
{ "goods_name" : "移动100元充值卡" }
{ "goods_name" : "移动20元充值卡" }
{ "goods_name" : "摩托罗拉E8 " }
```



### 特定类型的查询

####  null

null查询有点奇怪,null表示空,使用努力了查询不仅仅匹配自身,而且还会匹配"不存在多大",所以,这种匹配还会返回缺少这个键的文档,下面我们距离说明

元数据

```json
{
        "_id" : ObjectId("5c0cf5a2bde03aeb84cf5aaa"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5aabde03aeb84cf5aab"),
        "username" : "hello",
        "nickname" : "HELLO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5aebde03aeb84cf5aac"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5b8bde03aeb84cf5aad"),
        "username" : null,
        "nickname" : "JSON",
        "password" : "123"
}
```

第四个文档的"username"的值是null,若是我们查询集合user中"usrname"的值为null:

```json
> db.user.find({username:null}).pretty()
{
        "_id" : ObjectId("5c0cf5b8bde03aeb84cf5aad"),
        "username" : null,
        "nickname" : "JSON",
        "password" : "123"
}
```

集合中没有"age"这个键,如果我们将上面的查询语句username改成age结果会返回所有的文档。

```json
> db.user.find({age:null}).pretty()
{
        "_id" : ObjectId("5c0cf5a2bde03aeb84cf5aaa"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5aabde03aeb84cf5aab"),
        "username" : "hello",
        "nickname" : "HELLO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5aebde03aeb84cf5aac"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5b8bde03aeb84cf5aad"),
        "username" : null,
        "nickname" : "JSON",
        "password" : "123"
}
>
```

#### 正则表达式

```json
> db.user.find({username:/foo/i}).pretty()
{
        "_id" : ObjectId("5c0cf5a2bde03aeb84cf5aaa"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
{
        "_id" : ObjectId("5c0cf5aebde03aeb84cf5aac"),
        "username" : "foo",
        "nickname" : "FOO",
        "password" : "123"
}
```



#### 查询数组

查询数组中的元素也是很简单的，每个元素都是整个键（数组的键）的值。例如：如果数组是一个水果清单

```json
{
        "_id" : ObjectId("5c0cfe68bde03aeb84cf5aae"),
        "fruit" : [
                "apple",
                "banana",
                "peach"
        ]
}
```

查询语句

```json
> db.food.find({fruit:"banana"}).pretty()
{
        "_id" : ObjectId("5c0cfe68bde03aeb84cf5aae"),
        "fruit" : [
                "apple",
                "banana",
                "peach"
        ]
}
```

数组查询方法

#### $all

多个元素匹配数组，用$all匹配一组元素

```json
#元数据
> db.food.find()
{ "_id" : ObjectId("5c0cfe68bde03aeb84cf5aae"), "fruit" : [ "apple", "banana", "peach" ] }
{ "_id" : ObjectId("5c0d00b6bde03aeb84cf5aaf"), "fruit" : [ "apple", "kumquat", "orange" ] }
{ "_id" : ObjectId("5c0d00cbbde03aeb84cf5ab0"), "fruit" : [ "cherry", "banana", "apple" ] }

#查询由 apple和banana的文档
> db.food.find({fruit:{$all:["apple","banana"]}})
{ "_id" : ObjectId("5c0cfe68bde03aeb84cf5aae"), "fruit" : [ "apple", "banana", "peach" ] }
{ "_id" : ObjectId("5c0d00cbbde03aeb84cf5ab0"), "fruit" : [ "cherry", "banana", "apple" ] }
```

##### key.index

数组中的索引可以作为键使用，需要key.index语法指定数组的下标

db.food.find({})

```json
> db.food.find({"fruit.2":"orange"})
{ "_id" : ObjectId("5c0d00b6bde03aeb84cf5aaf"), "fruit" : [ "apple", "kumquat", "orange" ] }
```

数组的小标从0开始

#### \$size

size在查询语法中可以指定查询数组的大小（数组长度+1）

````js
> db.food.find({"fruit":{$size:3}})
{ "_id" : ObjectId("5c0cfe68bde03aeb84cf5aae"), "fruit" : [ "apple", "banana", "peach" ] }
{ "_id" : ObjectId("5c0d00b6bde03aeb84cf5aaf"), "fruit" : [ "apple", "kumquat", "orange" ] }
{ "_id" : ObjectId("5c0d00cbbde03aeb84cf5ab0"), "fruit" : [ "cherry", "banana", "apple" ] }
>
````



### 游标

#### 简介

MongoDB中的游标与关系型数据库中的游标在功能上大同小异,游标相当于C语言中的指针,可以定位到某一条记录,在MongoDB中,则是文档,因此在mongoDB中游标也有定义生命,打开,读取,关闭这几个流程.客户通过游标,能够实现对最终结构进行有效的控制,例如限制结果输了,跳过该部分结果或者根据.

游标不是查询结果,可以理解为数据在便利过程中内部指针,其返回值是一个资源,或者说数据读取接口,客户端通过对游标进行一些设置就能够对查询结果进行有效的控制如可以限制查询得到的结果数量,跳过部分结果,或者对结果集进行任意键进行排序等.

直接对一个集合进行调用find()方法是,我们会发现如果查询的结果超过二十条,指挥返回20条结果,这是因为MingoDB会自动递归find()返回的游标

#### 介绍

db.collection.find()方法返回一个游标,对于文档的访问,我们需要进行游标迭代,MongoDB的游标与关系型数据库SQL中的游标类似,可以通过对游标进行(如限制查询的结果数,跳过结果数设置来控制查询结果).游标会消耗内存和相关系统资源,游标使用完后应尽快释放资源.

在mongo shell 中,如果返回的游标结束为指定给某个var定义的变量,则游标自动迭代20次,输出前20个文档,超出20的情形则需要输入it来法爷.本文内容描述手动方式实现索引迭代过程如下

###### 声明游标

```js
var cusor = db.collectionName.find(query,projection)
```

###### 打开游标

````js
cusor.hasNext()
#判断游标是否已经取到头了
````

###### 读取数据

```js
cusor.Next()
#取出游标的下一个文档
```


###### 关闭游标


```js
cusor.Next()
#此步骤可以省略通常是自动关闭,也可以显示关闭.
```

我们用while来实现便利游标的示例:

```js
var mycursor =db.user.find({})
while(mycursor.hasNext()){
    printjson(mycursor.next())
}
```

#### 输出游标结果集



1. print
2. printjson



1. 使用print输出游标的结果集:

    ```js
    var cursor = db.user.find(){}
    	while(curwsor.hasNext()){
    	print(tojson(myCurson.next()))    
    }
    ```
2. 是用printjson输出游标的结果集
    ```js
    var cursor = db.user.find(){}
    	while(curwsor.hasNext()){
    	printjson(tojson(myCurson.next()))    
    }
    ```

#### 迭代

游标还有一个迭代函数,允许我们自定义回调函数来逐个处理每个单元.cursor.forEach(回调函数)步骤如下

1. 定义回调函数
2. 打开游标
3. 迭代



##### 元数据

```js
> db.food.find()
{ "_id" : ObjectId("5c0d23c7bb8749abe43ce55c"), "food" : [ "apple", "banana", "peach" ] }
{ "_id" : ObjectId("5c0d23dfbb8749abe43ce55d"), "food" : [ "apple", "kumquat", "orange" ] }
{ "_id" : ObjectId("5c0d23f3bb8749abe43ce55e"), "food" : [ "cherry", "banana", "apple" ] }
>
```

##### 定义函数

```json
> var getFuntion = function(obj){print(obj.food)}
```

##### 打开游标

```js
> var cursor = db.food.find()
```

##### 迭代

`````js
> cursor.forEach(getFuntion)
apple,kumquat,orange
cherry,banana,apple
cherry,banana,apple
`````

##### 数组迭代

````js
> var cursor = db.food.find()
> var documentArray=cursor.toArray()
> printjson(documentArray)
[
        {
                "_id" : ObjectId("5c0d250634103d4c5c927ac8"),
                "food" : [
                        "apple",
                        "kumquat",
                        "orange"
                ]
        },
        {
                "_id" : ObjectId("5c0d251734103d4c5c927ac9"),
                "food" : [
                        "cherry",
                        "banana",
                        "apple"
                ]
        },
        {
                "_id" : ObjectId("5c0d252534103d4c5c927aca"),
                "food" : [
                        "cherry",
                        "banana",
                        "apple"
                ]
        }
]
````

##### 注意 

 不要随意使用toArray()因为这样会把所有的行立即以对象的形式存到内存里,可以用于取出少数的几行.

####  游标解析

看待游标有两种角度：客户端的游标以及客户端游标表示的数据库游标。前面讨论的都是客户端的游标，接下来简要看看服务器端发生了什么。在服务器端，游标消耗内存和其他资源。游标遍历尽了结果以后，或者客户端发来消息要求终止，数据库将会释放这些资源。释放的资源可以被数据库换作他用，这是非常有益的，所以要尽量保证尽快释放游标（在合理的前提下)。还有一些情况导致游标终止（随后被清理）。首先，当游标完成匹配结果的迭代时，它会清除自身。另外，当游标在客户端已经不在作用域内了，驱动会向服务器发送专门的消息，让其销毁游标。最后，即便用户也没有迭代完所有结果，并且游标也还在作用域中，10分钟不使用，数据库游标也会自动销毁。这种“超时销毁”的行为是我们希望的：极少有应用程序希望用户花费数分钟坐在那里等待结果。然而，的确有些时候希望游标持续的时间长一些。若是如此的话，多数驱动程序都实现了一个叫immortal的函数，或者类似的机制，来告知数据库不要让游标超时。如果关闭了游标的超时时间，则一定要在迭代完结果后将其关闭，否则它会一直在数据库中消耗服务器资源。



