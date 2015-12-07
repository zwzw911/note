1. redis支持16个数据库，默认使用0.可以是**select 1**更换数据库

1.  **set**     //设置key-value
2.  **exists**  //判断key是否存在
3.  **keys**  //打印所有key
4.  **del**     //删除所有key
5.  **type**    //显示key的类型
6.  **get**     //获得key对应的value
7.  **incr**    //增加key对应的value
8.  **incrby**  //增加key对应的value
9.  **decr** **decrby** //和incr/decrby相反
10.  **INCRBYFLOAT**    //增加浮点数
11.  **append**       //尾部增加字符。foo=6.1; append foo 2; foo-6.1**2**
12.  **strlen**       //**字符**长度
13.  **mget/mset**    // mget key1 val1 key2 val2
  
#####hash  
1.  **hset** key field value
2.  **hmset** key field1 val1 field2 vale
3.  **hmget** key f1 f2
4.  **hgetall** key
5.  **hexists** key f1
6.  **hsetNX**  key f3 3  //如果f3已经存在，那么不设置
7.  **hkeys/hvals** key   //获得一个key所有字段名/字段值  

#####list
1.  **lpush/rpush** key val1 [val2]   //把val1、val2从左边（L）或右（R）边推入list
2.  **lpush/rpush** key    //从左边（L）或右（R）边删除一个值，并返回这个值
3.  **llen**  key     //list的长度（key不存在时为0）
4.  **lrange** key start stop   //start、stop可选正数和负数；正数：从左边开始选取，负数：从右边选取。lrange key 0 -1：选取所有。
5.  **lrem** key count val    //从key中删除conut个值为val的元素。count>0，从左开始删除count个；count<0，从右开始删除count个；count=0，删除所有val；
6.  **lindex** key idx  //当成一个数组，使用index显示值
7.  **lset** key idx val  //把list中idx对应的值设成val
8.  **ltrim** key startIdx endIdx   //把list中，除了startIdx至endIdx外，所有的元素都删除
9.  **linsert** key **after/before** checkval insertval //在**第一个**查找到的checkval之前（before）或者（之后），插入insertval
10.  **RpopLPush**  key1 key2 //从key1中rpop出**一个元素**，然后lpush到key2中

#####set(集合)
1.  **sADD**  key member1 member2 //集合添加成员，如果成员值已经存在，忽略当前添加
2.  **sREM**  key member1 member2 //集合删除成员
3.  **sMEMBERS**  key   //获得集合中所有元素
4.  **sISMEMBER** key member1 //判断member1是不是已经在key中存在了。存在：1，不存在：0
