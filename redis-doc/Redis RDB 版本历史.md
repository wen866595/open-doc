

**
译者： coderbee (wen866595@163.com)   
转载请注明出处
**



#  Redis RDB 版本历史
这个文档跟踪了转储文件格式在过去的改变。

Redis 转储文件是100%向后兼容的。旧的转储文件总是可以在新版的Redis里工作。


##  版本6
在上一个版本，ziplist为整数使用一个可变长度的编码模式。
整数存储在 16, 32, 或 64位。在这个版本里，这种可变长度编码系统已被扩展。

做出的添加如下：

1.  整数0 到12，都包含（[0-12]），被编码为条目头部的一部分
2.  数字 -128 到 127，都包含（[-128 - 127]），存储在1个字节
3.  数字 -2^32 到 2^32 - 1，都包含（[-2^32  -  2^32 - 1]），存储在3个字节

问题ID： <https://github.com/antirez/redis/issues/469>

为为迁移到版本5：

1.  在redis.conf，设置 `list-max-ziplist-entries`为0
2.  重启Redis服务器，发送`SAVE`命令
3.  编辑dump.rdb文件，把头部的rdb版本改为`REDIS0005`


##  版本5
这个版本在文件尾部引入一个8字节CRC-32校验和，如果redis.conf里禁用校验和，最后8字节是0。

问题ID： <https://github.com/antirez/redis/issues/366>

为迁移到版本4：

1.  删除文件的最后8个字节（在字节`0xFF`后）
2.  把头部的rdb版本改为`REDIS0004`


##  版本4
这个版本为hashmap引入一种新的编码--“编码为Ziplist的Hashmap”。这个版本也废弃了之前版本使用的Zipmap编码。

“编码为Ziplist的Hashmap”的编码类型=13。值像ziplist那样解析，list里邻接的条目被认为是hashmap的键值对。

问题ID：<https://github.com/antirez/redis/pull/285>

为迁移到版本3：

1.  在redis.conf，设置`hash-max-ziplist-entries`为0。
2.  重启Redis服务器，发送`SAVE`命令
3.  编辑dump.rdb文件，把头部的rdb版本改为`REDIS0003`


##  版本3
这个版本引入精确到毫秒的键有效期限。

之前版本存储键有效期限的格式为`0xFD <4字节时间戳>`。在版本3，键有效期限存储为`0xFC <8字节时间戳>`。
在这里，0xFD和0xFC是分别指出键有效期限是以秒和毫秒的操作码。

问题ID： <https://github.com/antirez/redis/issues/169>

为迁移到版本2：

1.  如果你不使用键有效期限，简单地把头部的版本改为`REDIS0002`
2.  如果你使用键有效期限，仍然可以迁移，但那将导致有效期限精度的损失。而且，迁移将有点复杂
3.  对于转储文件里的每个键值对你将需要转换`0xFC <8字节时间戳>` 到 `0xFD <4字节时间戳>`
4.  转换完时间戳后，把头部的rdb版本改为`REDIS0002`


##  版本2
这个版本为小的hashmap，list和set引入特殊的编码。

特别是，它引入下面的编码类型 -   
REDIS_RDB_TYPE_HASH_ZIPMAP = 9   
REDIS_RDB_TYPE_LIST_ZIPLIST = 10   
REDIS_RDB_TYPE_SET_INTSET = 11   
REDIS_RDB_TYPE_ZSET_ZIPLIST = 12   

提交： <https://github.com/antirez/redis/commit/6b52ad87c05ca2162a2d21f1f5b5329bf52a7678>

为迁移到版本1：

1.  在redis.conf，设置属性`hash-max-zipmap-entries, list-max-ziplist-entries, set-max-intset-entries, zset-max-ziplist-entries` 为0
2.  重启Redis服务器，发送`SAVE`命令
3.  编辑dump.rdb文件，把头部的rdb版本改为`REDIS0001`

