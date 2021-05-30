<a name="index">**Index**</a>

<a href="#0">短连接设计</a>  
&emsp;<a href="#1">1. 资料</a>  
&emsp;<a href="#2">2. 确定问题及业务需求</a>  
&emsp;<a href="#3">3. 短链接设计要点</a>  
&emsp;&emsp;<a href="#4">3.1. 使是否使用数据库</a>  
&emsp;&emsp;<a href="#5">3.2. 确定短连接长度</a>  
&emsp;&emsp;<a href="#6">3.3. 方案一: 摘要hash算法</a>  
&emsp;&emsp;&emsp;<a href="#7">3.3.1. 使用hash算法 + bash 62位转换</a>  
&emsp;&emsp;&emsp;&emsp;<a href="#8">3.3.1.1. hash冲突</a>  
&emsp;&emsp;<a href="#9">3.4. 方案二:使用UUID的发号器服务方案</a>  
&emsp;&emsp;<a href="#10">3.5. 数据库映射</a>  
&emsp;&emsp;&emsp;<a href="#11">3.5.1. 短连接永久有效</a>  
&emsp;&emsp;&emsp;<a href="#12">3.5.2. 短连接存在过期时间</a>  
&emsp;&emsp;&emsp;<a href="#13">3.5.3. 数据库表设计</a>  
&emsp;&emsp;&emsp;<a href="#14">3.5.4. 分库分表</a>  
&emsp;<a href="#15">4. 短链接跳转，301还是302重定向</a>  
&emsp;<a href="#16">5. 预防攻击</a>  
&emsp;<a href="#17">6. 思考问题</a>  
# <a name="0">短连接设计</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

## <a name="1">资料</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

相关视频：
- [b站视频简单解说](https://www.bilibili.com/video/BV1Q541167ac?from=search&seid=14125928488165991852)
- [b站多方案完整解说（完整解说）](https://www.bilibili.com/video/BV1dy4y1E7A3?from=search&seid=15146001906771814541)

相关文章：
- [短连接设计](https://github.com/soulmachine/system-design/blob/master/cn/tinyurl.md)
- [国外设计宝典翻译：短连接设计](https://github.com/xitu/system-design-primer/blob/translation/solutions/system_design/pastebin/README-zh-Hans.md)
- [如何实现一个短链接服务](https://www.cnblogs.com/rickiyang/p/12178644.html)
- [知乎:短链接、短网址使用的是什么算法？](https://www.zhihu.com/question/20103344/answer/573638467)
- [TinyURL 设计短网址系统](https://segmentfault.com/a/1190000006140476)

## <a name="2">确定问题及业务需求</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
1. 数据量，增长数据量。
2. 永久短连接还是具有过期时间的短连接。
3. 使用数据库或不使用数据库保存。


## <a name="3">短链接设计要点</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
### <a name="4">使是否使用数据库</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
使用数据库，可以收集数据，进而再进行数据分析。而不适用数据库纯粹只是生成短连接服务。

### <a name="5">确定短连接长度</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
1. 使用大小写字母+数字的62进制方案，取6~8位便可以支持上亿的不同短连接

62^6 = 568,00235584
62^7 = 35216,14606208

2. 使用32进制小写字母+数字 36位
36^8 = 3w亿


### <a name="6">方案一: 摘要hash算法</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

#### <a name="7">使用hash算法 + bash 62位转换</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
hash算法使用MD5

1. 使用MD5 摘要算法进行长URL加密(分为16位和32位)，
    - MD5 是一个普遍用来生成一个 128-bit 长度的哈希值的一种哈希方法
2. 使用base 62位，转换md5
结果获取前8个字符或者后八位，当成短连接地址。

> 对于 urls，使用 Base 62 编码 [a-zA-Z0-9] 是比较合适的, 对于每一个原始输入只会有一个 hash 结果，Base 62 是确定的（不涉及随机性）
> Base 64 是另外一个流行的编码方案，但是对于 urls，会因为额外的 + 和 - 字符串而产生一些问题

##### <a name="8">hash冲突</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
1. 加密后，必定会出现hash冲突的情况，可以对原有的url进行加减字符重新加密(如在长连接前添加固定字符randomSalt)。或者对于加密后的md5再次md5加密。
2. 使用url+时间戳生成，生成失败后获取新的时间戳生成。


### <a name="9">方案二:使用UUID的发号器服务方案</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

1. 使用如雪花主键的UUID当成长链接对应的键，
2. 维护一个keyGenerate 服务
3. 随机生成数
> B站视频有完整解说

### <a name="10">数据库映射</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
区分短连接的用途进行设计，主要有以下两种：
1. 短连接永久有效，无需对短连接进行数据分析。
2. 短连接存在过期时间
    1. 无需对短连接进行数据分析。
    2. 需要根据生成链接的用户等信息，的对短连接进行数据分析，
    
#### <a name="11">短连接永久有效</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
短连接永久有效，正常就需要落数据库表。

表设计：使用短连接作为主键，长连接作为表字段。
    
#### <a name="12">短连接存在过期时间</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
若短链接无需进行数据分析，那么可以直接选用NoSql的数据库，如Redis。
1. 长连接与短连接直接使用String数据结构，长连接K - 短连接V 加上过期时间
2. 如何判断短连接是否已经生成，使用Redis的布隆过滤器。


若短连接需要进行数据分析，那么数据需要存储到数据库中：
- 存储到关系型数据库中，由于短链接存在过期时间，那么可以使用自增主键，而短连接、长连接作为字段，另外还有用户信息等。
1. 若长链接需要区分分享的用户，地点，时间等信息，长链接与短链接以1对多的方式存储。可用于后序的数据分析。
2. 若长连接无需区分用户的类型，可直接公用相同的短链接。如何建立映射关系可见下方：
3. 结合Redis，使用布隆过滤器进行短连接的判空。使用String的数据结构判断，短连接是否存在。



#### <a name="13">数据库表设计</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

使用shortlink 当成组件
```
shortlink char(7) NOT NULL
expiration_length_in_minutes int NOT NULL
created_at datetime NOT NULL
paste_path varchar(255) NOT NULL
PRIMARY KEY(shortlink)
```

#### <a name="14">分库分表</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
由于短链接可直接做进制转换，并具有唯一性

使用短连接作为sharding key，进行水平分库拆分。

另外可以基于一季度做数据归档。

## <a name="15">短链接跳转，301还是302重定向</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
这个问题主要是考察你对301和302的理解，以及**浏览器缓存**机制的理解。

301是永久重定向，302是临时重定向。短地址一经生成就不会变化，所以用301是符合http语义的。但是如果用了301， Google，百度等搜索引擎，搜索的时候会直接展示**真实地址**(即短链接对应的长连接地址，浏览器缓存)，那我们就无法统计到短地址被点击的次数了，也无法收集用户的Cookie, User Agent 等信息，这些信息可以用来做很多有意思的大数据分析，也是短网址服务商的主要盈利来源。

所以，正确答案是302重定向。

## <a name="16">预防攻击</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
1. 频繁生成短链接的 使用ip黑名单屏蔽。
2. 使用redis建立长连接->ID 的缓存。

## <a name="17">思考问题</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
短连接永久有效，使用Mysql关系型存储数据，如何判断该长链接已经生成过短链接？
- 长链接建立索引？
 错，字段随机性过大，且索引树节点存储的数据少。
 
- 长链接生成短连接后，通过短连接查询？ 
    - 若短连接使用发号器生成，该方法失效。
    - 若长链接使用Hash算法生成，需保证同一个Key处理Hash冲突的流程及结果一致，才能查到是否已经转换过短链接。如冲突时加上同样的字段再hash
上述方法问题点：短连接过于随机，数据插入时候，可能导致短连接索引树调整的效率问题。