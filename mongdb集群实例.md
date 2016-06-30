手头有3台机器：  
1. shard设置3个，以便每个shard配在不同的PC上，实现负荷分担。每个shard配置成复制集，1P+2S，每个shard的P在不同PC上。3个shard复制集，分别使用27020/27021/27022端口  
2. config server也配置3个，在不同的PC，端口为27019.  
3. mongos和app在同一个server，port：27017  

##### conf file
systemLog:
    traceAllExceptions: true
    #log存储的方式，如果定义file，必须定义path；还可以定义为syslog
    destination: file
    # 必须指定log文件名
    path: "D:/ss_log/mongo/shard1/shard1.log"
    #window中，对新log重命名；unix/linux中，rename，以便新建文件
    logRotate: rename
    #window中，创建一个新log文件；unix/Linux，true，以便在新打开的文件中存入log
    logAppend: false
    timeStampFormat: iso8601-local
net:
    bindIp: 135.252.254.77
    port: 27020
replication:
    oplogSizeMB: 1000
    replSetName: shard1
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
    dbPath: "D:/ss_db/mongo/shard1"
    #如果在build index过程中停止mongodb，那么下次启动mongodb是否需要rebuild index
    indexBuildRetry: true
    #必须是dbPath的子目录，并且已经存在（不存在需手工创建）
    repairPath: D:/ss_db/mongo/shard1/repair
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
