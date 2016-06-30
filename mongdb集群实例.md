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

#####目录结构
1. ss_db==>mongo==>config/shard1/shard2/shard3==>**repair**。 repair only for shard。  
2. ss_log===>mongo===>config==>config.log/shard1===>shard1.log/shard2===>shard2.log/shard3===>shard3.log。**config server其实不需要log文件，但是为了install as window service，添加上的**   
3. ss_conf==>mongo==>config.conf/shard1.conf/shard2.conf/shard3.conf  

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

#####数据（复制集）命令
1. 作为service安装：
**mongod -f D:\ss_conf\mongo\shard1.conf --serviceName MongoDBShard1 --serviceDisplayName MongoDBShard1 --install**。必须设定serviceDisplayName，否则每个shard会重名，无法安装。
配置分片复制集：
1. 初始化复制集（不带任何参数）：**rs.initiate()**  
2. 添加成员，如果是P，设置priority为2（大于默认的1），以便对应的成员是P。
    PC1：rs.**add**({host:**"**135.252.254.80:27020",priporty:1})/rs.add({host:"135.252.254.87:27020",priority:1}) 。  
    PC1：rs.add({host:"135.252.254.80:27021",priporty:**2**})/rs.add({host:"135.252.254.87:27021",priority:1}) 。
    PC1：rs.add({host:"135.252.254.80:27022",priporty:1})/rs.add({host:"135.252.254.87:27022",priority:2})。  
**注意：以上命令都在PC1上执行，通过设置不同priority，确定P落在哪个member。host的value要用双引号括起**  
3. 设置成员priority(rs.initiate()添加当前PC的复制集member，并且priority默认为1)
    cfg=rs.conf()
    cfg.members[0].priority=2
    rs.**reconfig**(cfg)

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
4. **mongod -f D:\ss_conf\mongo\cfgsvr.conf --serviceName MongoDBCfgsvr --serviceDisplayName MongoDBCfgsvr --install**。 
5. 在P上，执行> rs.initiate( {_id: "configReplSet",configsvr: true,members: [{ _id: 0, host: "
135.252.254.77:27019" },{ _id: 1, host: "135.252.254.80:27019" },{ _id: 2, host: "135.252.254.87:27019" }]} )。**和普通repl略有不同，initiate时候必须含有如上filed：_id/configsvr/members**。普通repl不需这些参数，可以在initiate后，通过命令方式添加member。  

#####启动mongos
**mongos --configdb configReplSet/135.252.254.77:27019,135.252.254.80:27019,135,252,254,87:21091 --port 27018**。可以指定port。  
连接mongos：mongo --host 135.252.254.77(运行mongos的PC地址) --port 27018


#####shard操作
1. **添加shard**  
每个shard只需添加一个member（如果分片使用复制集）。注意：**addShard只需添加复制集中任意一个member（P or S都行）即可，mongs会自动找到其他member**  
sh.addShard("shard1/135.252.254.77:27020")  
sh.addShard("shard2/135.252.254.77:27021")  
sh.addShard("shard3/135.252.254.77:27022")  
2. *删除shard**  
**删除只能通过runCommand** 
db.runCommand(removeShard:"shard1")  
3. **db上启动shard**
sh.enableSharding("ss")  
