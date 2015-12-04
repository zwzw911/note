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
6.  **hsetNX**  key f3 3
