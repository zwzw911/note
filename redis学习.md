0. **redis常用组件**    
redis-server  
redis-cli    
redis-benchmark  性能测试工具  
redis-check-aof  AOF文件修复工具  
redis-check-dump  RDB文件检查工具  

db是实例中的命名空间，即db并不互相隔离，例如使用***FLUSHALL***会将所有db中的数据清空

1. redis支持16个数据库，默认使用0.可以是**select 1**更换数据库

#####公用命令
1.  **exists**  //判断key是否存在
2.  **keys**  //打印所有key
3.  **type**    //显示key的类型

#####string
1.  **set/get**     //设置/读取key-value    
2.  **mget/mset**    // mget key1 val1 key2 val2 
3.  **del**     //删除所有key
4.  **incr**    //如果存储的字符是数字，value+1并返回  
5.  **incrby**  //和incr类似，但是可以指定对应的增长值  
6.  **decr/decrby** //和incr/decrby相反，减少对应的值  
7.  **INCRBYFLOAT**    //增加浮点数
8.  **append**       //向尾部增加字符，如果字符不存在，创建新的。返回字符长度 *foo=6.1; append foo 2; foo=6.1**2**,并返回4（字符的长度，包含小数点）*
9.  **strlen**       //**字符**长度，如果key不存在，返回0；否则返回字符长度   
10.  **getbit**   //获得字符的指定bit的值（0/1），idx从0开始，超出范围默认0。getbit k idx  
11.  **setbit**   //设定字符的指定bit的值（0/1）。如果key不存在，设置idx前的bit为0，idx的bit为设定值。*setbit notExist 5 1====>idx 5对应的bit为1，idx 0～4对应的bit为0*  
12.  **bitcount**   //统计1的个数  
13.  **bitop**      //bit运算。OR/AND/XOR/NOT    *bitop op result para1 para2*  
  
#####**hash**  
hash的值只能存储字符串  
1.  **hset/hget**  //hset/hget key field value    设置/读取key中单个field的value    
2.  **hmset/hmget** //key field1 val1 field2 vale  设置/读取key中多个field的value
4.  **hgetall** key  //返回所有key和value
7.  **hkeys/hvals** key   //获得一个key所有字段名/字段值
5.  **hexists** key f1    //判断**key下的field**是否存在，存在，返回1，不存在，返回0  
6.  **hSetNX**  key f3 3  //如果f3不存在，那么设置值，并返回1；如果field已经存在，不做任何操作，并返回0.*可以代替**hexists和hset***      
8.  **hdel** key f1   //删除hash下的一个字段
9.  **hlen**  key   //key下有几个字段  
10. **hIncrBy**  key field increament //对指定key的某个field，增加increament。*hash 没有hincr，使用hincrby key field 1代替*

#####list
1.  **lpush/rpush** key val1 [val2]   //把val1、val2从左边（L）或右（R）边推入list
2.  **lpop/rpop** key    //从左边（L）或右（R）边删除**一个**值，并返回这个值
3.  **llen**  key     //list的长度（key不存在时为0）
4.  **lrange** key start stop   //start、stop可选正数和负数；正数：从左边开始选取，负数：从右边选取。lrange key 0 -1：选取所有。
5.  **lrem** key count val    //从key中删除conut个值为val的元素。count>0，从左开始删除count个；count<0，从右开始删除count个；count=0，删除所有val；
6.  **lindex** key idx  //当成一个数组，使用index显示值
7.  **lset** key idx val  //把list中idx对应的值设成val
8.  **ltrim** key startIdx endIdx   //把list中，除了startIdx至endIdx外，所有的元素都删除
9.  **linsert** key **after/before** checkval insertval //在**第一个**查找到的checkval之前（before）或者（之后），插入insertval
10.  **RpopLPush**  key1 key2 //从key1中rpop出**一个元素**，然后lpush到key2中

#####set(集合)
1.  **sADD**  key member1 member2 //集合添加成员，如果成员值已经存在，忽略当前添加。返回值为成功添加的成员数。  
2.  **sREM**  key member1 member2 //集合删除成员。返回值为成功删除的成员数。  
3.  **sMEMBERS**  key   //获得集合中所有元素。  
4.  **sISMEMBER** key member1 //判断member1是不是已经在key中存在了。存在：1，不存在：0
5.  **sDIFF** key1 key2 //key1－key2（key1中有，而key2中没有的部分）
6.  **sINTER**  key1 key2 //key1，key2共有部分
7.  **sUNION**  key1 key2 //key1+key2
8.  **sCARD** key   //获得集合中所有成员的**数量**
9.  **sDIFF/INTER/UNION/STORE** dest key1 key2  //和diff/inter/union类似，只是会把结果存储在dest这个新的键值中
10.  **sRANDMEMBER**  key [count]   //从集合中随机获取count个成员。如果count为正，随机获得conut个不同的成员，如果count>集合数，返回集合中所有的值；如果count为负，获得的count个值可能相同  
11.  **sPOP** key   //从集合中返回一个成员，并从集合中删除

#####有序集合
1. **zADD** key score val //scroe可以是整数，小数，+inf，－inf
2. **zSCORE** key val   //val对应的score
3. **z(REV)RANGE** key startIdx endIdx **[withscores]**  //按照score从小到大排序，取index[start,end]的元素。withscores：同时显示score
4.  **z(REV)RANGEBYSCORE** key scoreMIN scoreMAX **[withscores]** **[LIMIT]** start total//根据scoreMIN/MAX范围取值, limit同mysql
5.  **zINCRBY** key scorenum val  //为某个val增加score值，可以为正数，负数
6.  **zCARD** key   //有序集合的成员数
7.  **zCount**  key scoremin scoremax   //统计key中min/max范围内的成员数量
8.  **zREM**  key member1 [member2]   //删除成员
9.  **zREMRANGEBYRANK** key start stop    //按照score排序，删除index [start,stop]的元素，并返回（显示）
10.  **zRANK**  key val     //获得val对应的排名（idx）


#####事务
1.  **multi/exec**      //开始/结束事务，如果**事务命令语法出错，事务不执行**；如果事务**执行中（运行时）出错，跳过出错语句，继续向下执行，需要手动回滚（redis不支持自动rollback）**
2.  **(p)expire** key  time(s)  //设置key超时时间（秒），redis自动删除; pEXPIRE是以ms为单位
3.  **ttl** key   //key还剩多久过期
4.  **persist** key   //设成expire的key再次变成永久key


#####排序
**sort** key [alpha] [DESC] [LIMIT offset count]   //用作对列表（list）和集合（set/zset，有序集合的话，据略分数，直接对value）进行排序。alpha对字符进行排序。DESC：反向排序。LIMIT:跳过offset，总共显示count个。
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
RDB：snapshot。 save 900 1/save 300 10   可以有多个条件，是或的关系。900秒内有一个数据或者300秒内有10个数据被更改，则保存。原理：执行的时候fork一个子进程，将内存数据写入临时文件，此期间，如果有数据变化，父进程copy一份，保证子进程的内存不受影响。也可以通过SAVE（父进程备份，期间阻塞操作）或者BGSAVE（子进程）手工持久化。    

AOF：append only file。 默认不开启，设置appendonly **no=>yes**开启。    
appendfilenam默认为appendonly.aof，内容是文本文件，记录了操作的命令。有些命令可以优化，例如，连续对一个key进行了若干次set操作，那么只要记录最后一个set即可：**auto-aof-rewrite-percentage**：如果AOF超过上次重写时文件的大小（百分比），**auto-aof-rewrite-min-size** 64mb：AOF超过64MB，就重写。**BGREWRITEAO**，    
理论是，每次更改会被记录到硬盘上，但是由于硬盘缓存的存在，可能会有延时，此时，可以设置**appendfsync** always（每条命令都写入硬盘，最安全但是最慢）/everysec（每秒执行，适中）/no（follow OS，一般是30秒写一次，效率最高，但是有丢失数据的风险）从磁盘缓存写入磁盘的策略。
  
#####实际操作
1. redis-cli：进入redis
2. redis-cli下运行脚本：eval "script.lua" 2 key1 val1 key2 val2。 **直接使用eval，可能会报错误信息，使用evalsha就不会有错**。  
3. 获得脚本SHA：通过fs读取脚本内容，然后通过ioclient的load方法把内容转换成sha  
4. Lua 脚本中，return之后最好不要跟redis.call这样的语句，否则无法编译。
