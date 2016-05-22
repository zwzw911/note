1. redis支持16个数据库，默认使用0.可以是**select 1**更换数据库

#####公用命令
1.  **exists**  //判断key是否存在
2.  **keys**  //打印所有key
3.  **type**    //显示key的类型

#####string
1.  **set**     //设置key-value
2.  **del**     //删除所有key
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
3.  **hget** key f1
3.  **hmget** key f1 f2
4.  **hgetall** key
5.  **hexists** key f1    //判断的是**key下的field**
6.  **hsetNX**  key f3 3  //如果f3已经存在，那么不设置
7.  **hkeys/hvals** key   //获得一个key所有字段名/字段值  
8.  **hdel** key f1   //删除hash下的一个字段
9.  **hlen**  key   //key下有几个字段

#####list
1.  **lpush/rpush** key val1 [val2]   //把val1、val2从左边（L）或右（R）边推入list
2.  **lpop/rpop** key    //从左边（L）或右（R）边删除一个值，并返回这个值
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
5.  **sDIFF** key1 key2 //key1－key2（key1中有，而key2中没有的部分）
6.  **sINTER**  key1 key2 //key1，key2共有部分
7.  **sUNION**  key1 key2 //key1+key2
8.  **sCARD** key   //获得集合中成员的数量
9.  **sDIFF/INTER/UNION/STORE** dest key1 key2  //和diff/inter/union类似，只是会把结果存储在dest这个新的键值中
10.  **sRANDMEMBER**  key [count]   //从集合中随机获取count个成员
11.  **sPOP** key   //从集合中返回一个成员，并从集合中删除

#####有序集合
1. **zADD** key score val //scroe可以是整数，小数，+inf，－inf
2. **zSCORE** key val   //val对应的score
3. **z(REV)RANGE** key startIdx endIdx **[withscores]**  //按照score从小达到排序，去index[start,end]的元素。withscores：同时显示score
4.  **z(REV)RANGEBYSCORE** key scoreMIN scoreMAX **[withscores]** **[LIMIT]** start total//根据scoreMIN/MAX范围取值, limit同mysql
5.  **zINCRBY** key scorenum val  //为某个val增加score值，可以为正数，负数
6.  **zCARD** key   //有序集合的成员数
7.  **zCount**  key scoremin scoremax   //统计key中min/max范围内的成员数量
8.  **zREM**  key member1 [member2]   //删除成员
9.  **zREMRANGEBYRANK** key start stop    //按照score排序，删除index [start,stop]的元素，并返回（显示）
10.  **zRANK**  key val     //获得val对应的排名（idx）


#####事务
1.  **multi/exec**      //开始结束事务，如果**事务命令出错，事务不执行**；如果事务**执行中出错，跳过出错语句，继续向下执行，需要手动回滚（redis不支持自动rollback）**
2.  **(p)expire** key  time(s)  //设置key超时时间（秒），redis自动删除; pEXPIRE是以ms为单位
3.  **ttl** key   //key还剩多久过期
4.  **persist** key   //设成expire的key再次变成永久key


#####排序
**sort** key [alpha] [DESC] [limit offset count]   //用作对列表（list）和集合（set/zset）进行排序。alpha对无法字符进行排序。DESC：反向排序。limit:跳过offset，总共显示count个。
**by**
**get**   //#返回本身
**store**   //sort/by/get返回值存储在**列表**中

#####任务
**生产者**  //产生任务
**消费者**  //执行任务
任务的概念使用列表（list）比较合适，lpush加入任务，rpop取出任务。
**bRPOP** list1 list2 timeout   //阻塞，直到有新元素加入list才读取。list1,list2，如果都有新元素，按照list1，list2的顺序执行。timeout，检测超时

#####订阅发布
**publish** channel val  //向channel发布数据val
**(un)subscribe** channel   //（退订）订阅channel的数据
**p(un)subscribe**  channel.*   //和subscribe互不影响。可以使用通配符

#####管道
将多个不互相依赖的命令，通过管道一次发送，并且一次获得结果。可以节省client/server之间的往返时间

#####持久化
RDB：snapshot。 save 900 1/save 300 10
AOF：append only file。 appendonly **no=>yes**。appendfilenam默认为appendonly.aof，**auto-aof-rewrite-percentage** 100，**auto-aof-rewrite-min-size** 64mb，**BGREWRITEAO**，**appendfsync** always/everysec/no9从磁盘缓存写入磁盘的策略）
  
#####实际操作
1. redis-cli：进入redis
2. redis-cli下运行脚本：eval "script.lua" 2 key1 val1 key2 val2。 **直接使用eval，可能会报错误信息，使用evalsha就不会有错**。  
3. 获得脚本SHA：通过fs读取脚本内容，然后通过ioclient的load方法把内容转换成sha  
4. Lua 脚本中，return之后最好不要跟redis.call这样的语句，否则无法编译。
