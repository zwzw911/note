#####配置
1. 最简配置  
单台PC： app/图片/redis/mongo都在单台PC    
2. 简单配置  
3台PC：  app/图片/redis/shard1在第一台PC，shar2在2nd PC，shard3在3rd PC。每个shard由复制集组成，1P+2S，分布在3台机器上，只能保证任意一个P OOS，新选举出的P还能IS，但是和其他shard的P在同一个PC上；极端情况，2台PC OOS，所有shard都落在单台PC。
3. 中等配置  
7台PC：  app/图片/redis占用一台，shard1/2/3的P各自占用一台，剩下的2S分布在剩下的3台。如此，可以保证，任意一个P OOS，新选举出来的P还是能单独占用一台PC。  
4. 满配  
11台PC： app/redis一台，图片一台; 每个shard*3台PC（复制集最少需要奇数个有选举权的PC？？）  

手头有3台机器：  
1. shard设置3个，以便每个shard配在不同的PC上，实现负荷分担。每个shard配置成复制集，1P+2S，每个shard的P在不同PC上。3个shard复制集，分别使用27020/27021/27022端口  
2. config server也配置3个，在不同的PC，端口为27019.  
3. mongos和app在同一个server，port：27017  

##### conf file in shard1
systemLog:
    traceAllExceptions: true
    #log存储的方式，如果定义file，必须定义path；还可以定义为syslog
    destination: file
    # 必须指定log文件名
    path: **"D:/ss_log/mongo/shard1/shard1.log"**
    #window中，对新log重命名；unix/linux中，rename，以便新建文件
    logRotate: rename
    #window中，创建一个新log文件；unix/Linux，true，以便在新打开的文件中存入log
    logAppend: false
    timeStampFormat: iso8601-local
net:
    bindIp: **135.252.254.77**
    port: **27020**
replication:
    oplogSizeMB: 1000
    replSetName: **shard1**
    secondaryIndexPrefetch: all
    #默认false
    enableMajorityReadConcern: false
sharding:
    #在shard cluster中，mongod的作用：configsvr/shardsvr
    clusterRole: shardsvr
    archiveMovedChunks: false
operationProfiling:
    slowOpThresholdMs: 100
    #off/slowOp/all
    mode: slowOp
storage:
    dbPath: **"D:/ss_db/mongo/shard1"**
    #如果在build index过程中停止mongodb，那么下次启动mongodb是否需要rebuild index
    indexBuildRetry: true
    #必须是dbPath的子目录，并且已经存在（不存在需手工创建）
    repairPath: **D:/ss_db/mongo/shard1/repair**
    journal: 
      #mongod only；只有定义了dbPath后，才能使用；64bit OS默认打开，32bit默认关闭
      enabled: true
      #mongod only； MMAPv1，如果日志文件不是data file（而是其他的block device，默认30）； wiredTirger，默认100，并且write带j:true,立刻同步
      commitIntervalMs: 100
      #mongod only； 每个db用不同的目录（dbPath之下的子目录）
    directoryPerDB: true
    #3.2开始，3个：MMAPv1/wiredTiger（默认）/inMemory
    engine: wiredTiger
    wiredTiger:
        engineConfig:
           # 60%的RAM（最小1GB） 1GB，两者取大值； 定义的只是存储引擎使用的cache size，并且是针对单个mongod，如果运行多个mongod，需要减小
           cacheSizeGB: 1
           # none/zlib/snappy(默认)
           journalCompressor: snappy
           # index和data分存2目录下，默认false
           directoryForIndexes: false
        collectionConfig:
           #
           blockCompressor: snappy
        indexConfig:
           # 对index使用prefix压缩，默认true。只对设置true之后的doc生效。
           prefixCompression: true
...  

#####命令
1. 作为service安装：
**C:\Users\lte>mongod -f D:\ss_conf\mongo\shard1.conf --serviceName MongoDBShard1--serviceDisplayName MongoDBShard1 --install**。必须设定serviceDisplayName，否则每个shard会重名，无法安装。
1. 复制集：  


#####配置服务器
1. 从3.2起，config server可以使用复制集方式，前提是必须使用wiredTiger。**mongo会把localhost和127.0.0.1辨认为不同的host**  
2. 使用复制集来配置config server，必须满足：没有仲裁节点，没有延迟节点，必须建立索引（所有成员的buildIndexes？？必须设为true）
3. # 为了安装成window service，需要提供log path；否则config server启动无需此option
systemLog:
    traceAllExceptions: true
    #log存储的方式，如果定义file，必须定义path；还可以定义为syslog
    destination: file
    # 必须指定log文件名
    path: "D:/ss_log/mongo/config/config.log"
    #window中，对新log重命名；unix/linux中，rename，以便新建文件
    logRotate: rename
    #window中，创建一个新log文件；unix/Linux，true，以便在新打开的文件中存入log
    logAppend: false
    timeStampFormat: iso8601-local
sharding:
   clusterRole: configsvr
replication:
   replSetName: configReplSet
net:
   bindIp: 135.252.254.77
   port: 27019
storage:
   dbPath: D:\ss_db\mongo\config  
4. **C:\Users\lte>mongod -f D:\ss_conf\mongo\cfgsvr.conf --serviceName MongoDBCfgsvr--serviceDisplayName MongoDBCfgsvr --install**。 
5. 
