###分片
1. 分片集群的配置信息保存在一个/组专门的mongod服务器上，供包括mongos在内的其他程序使用。mongo移动chunk（数据块）时，**所有**的配置服务器都必须on service，否则操作无法进行/正在进行的chunk迁移回退。**任何一台**配置服务器OOS（或者primary的config server OOS，同时无法选出新的primary），config server变成**可读**，再此期间，对data仍可以读写，但是chunk的分裂和迁移都无法执行，直到所有config server都恢复正常。    
2. 集群由3部分组成：存储数据的分片，负责路由的mongos，跟踪（保存）集群状态的配置服务器。  
3. mongos和配置服务器都是轻量级：mongos可以和app server发在一起，配置服务器可以放在任意机器上。  
4. 片键不要选择:  
a) 值有限的字段，否则只能分成有限个分片；若是为了因为查询的原因，选择了小基数片键，采用组合片键（添加第二个字段）。  
b) 升/降序值：这样，所有数据总会被写入第一个/最后一个片。
3. 配置服务器的数量为1或3：1适用测试，3适用生产。**配置服务器之间有同步，但是和复制集机制不一样，所以只需启动mongod即可，无需参数**。**PS:3.2之后，配置服务器可以以复制集的方式部署，如此，可以最多配置50个config server（但是加大了管理难度）**    
4. **mongos --configdb hostname1:port1,hostname2:port2,hostname3:port3。** hostname可以用IP代替。mongos缓存config server的metadata，所以即使所有的config server都OOS，仍可以对cluster进行读写；但是一旦重启且config server仍旧未恢复，cluster不可读写。  
5. 当cluster中某个shard的chunk比其他shard多时，mongos会自动迁移数据。这是自动且透明的。3.2，开始均衡时，mongos会从config server获得一个lock，且此lock不受时钟不一致的影响（镜像config server对时钟同步有严格要求）。均衡器一次迁移一个chunk。可以**禁用均衡器**或者**设置均衡器时间窗口**。**迁移阀值**：cluster中chunk最多的shard和chunk最少的chunk之间的差值，如果达到迁移阀值，才开始迁移操作，防止过于频繁的操作，影响性能。2：少于20个；4：20～79；8：80个以上。均衡开始后，知道chunk差值少于2个或者迁移失败，才会停止。  
6. 数据迁移过程：均衡器发送**moveChunk**命令到**源分片**，源分片开始迁移数据，目标分片为要迁移的数据建立index，然后开始接收数据。接收完毕，开始同步（迁移过程中产生的新数据）。同步完成，更新config server。  
7. 2.6开始，迁移时，源分片上被迁移的chunk会被存成文件（--dbpath），使用**sharding.archiveMovedChunks**（默认enable）；**Jumbo chunk**：在迁移是，如果chunk大小超过chunk size或者chunk中的doc数量超过Maximum Number of Documents Per Chunk to Migrate，先分裂，后迁移。  
8. 

###replSet
1. 检测对端：mongo --host 135.252.254.80 --port 27017，能够连接说明本机和对端的连接OK。  
2. mongo的rep set最多支持50个节点，其中最多7个voting节点。voting节点必须是奇数，如果不是，添加一个arbiter节点。  
3. arbiter节点没有数据，同时不能被转换成primary。arbiter必须运行在没有primary或者secondary的节点上。  
4. replication是含有同样数据的mongo实例的集合。一个replication包含若干个数据节点和一个可选的arbiter节点。数据节点中，只有一个primary，剩下的都是secondary和voting节点。只有primary能处理write操作；secondary通过复制primary的oplog，（执行后）复制数据。
5. secondary可以通过一些参数配置，得到特殊的功能：  
  5.1 优先级为0：能接受read，也能进行vote，但是不会触发选举，永远是secondary（cold standby）  
  5.2 隐藏节点：不接受read，能进行vote，防止application从中读取数据  
  5.3 延时节点：通过一个运行中的“历史”snapshot，防止意外操作被记录下来。延时节点必须被设置成优先级0，且为隐藏节点。     
  5.4 投票节点：
5. 当primary停止响应超过10秒后，合适的secondary发起选举，选自己为primary，成员通过则变成primary。3.2采用protocol 1来节省故障回滚时间。  


###配置命令
1. 首先**分别在所有的节点上各自**启动一个mongod进程，出了bing_ip不同，其他尽量一致  
C:\Users\lte>**mongod --bind_ip 135.252.254.77 --logpath D:/ss_log/mongo/mongodb.log --logRotate rename ~~reopen~~ ~~--logappend~~ --timeStampFormat iso8601-local --oplogSize 1000 --dbpath D:/ss_db/mongo/ --replSet "ss"**  
**C:\Users\lte>mongod -f D:\ss_conf\mongo\shard1.conf --serviceName MongoDBShard1--install**  
**删除service：**sc delete MongoDBShard1
没有采用mongod -f xxx.conf的方式，因为window的mongodb版本是3.0，不支持某些选项。而直接采用命令行的方式。  
**--bind_ip**：可以是主机名或IP地址，建议使用IP，如果server配置多个IP，hostname可能会绑定内部IP，导致客户端无法连接。  
**--logRotate**:采用rename，这样每次启动生成一个新文件，防止单一文件过大？？  
**--replSet**：复制集合名称  
**--oplogSize**: primary的oplog大小，默认是磁盘剩余空间的5%。如果设置，单位是MB。 
2. 联入此mongod，在**单独一个节点**上执行initiate命令，完成replication需要的oplog文件的创建。单个文件最大2GB，如果oplogSize小于2GB，只创建一个文件local.1。    
**rs.initiate()**  
{  
        "info2" : "no configuration explicitly specified -- making one",  
        "me" : "SH-JQ-254-77:27017",  
        "ok" : 1  
}  
3. 命令  
   **rs.add()**:无论是IP还是hostname，必须使用""括起，作为字符  
   **rs.conf()**:  
   **rs.isMaster()**:  
   **rs.sterDown()**:对于primary，需要强制primary变成secondary，并且触发一次选举。之后可以进行维护操作。    
   **rs.reconf()**::传入配置信息，重新配置replication。  


###安全
######接入控制和身份验证（Authentication ）
集群中，要启用身份验证。MongoDB采用role的方式进行验证。  
######通信加密  
采用TTS/SSL在mongo通信中
######防止网络暴露
######审查网络活动  
企业版有工具可以记录分析各种操作  
######运行mongo  
使用特定的账号运行mongo，这个账号可以进入mongoDB，但是没有其他多余的权限。运行mongoDB with安全配置选项。  
mongoDB的某些操作（mapreduce，group，$where）需要server-side的javascript，如果不需要，使用**--noscriptig**选项关闭。  
######localhost Exception  
localhost Exception(通过localhost登录？？)只有在没有任何user的时候可以使用，用来在数据库admin中创建**第一个**用户。在**分片**中，LE不但应用在集群本身（mongos），也应用在分片。即使在mongos使用LE创建了user，还需要在各个分片上创建user（否则分片还是未使用验证机制）。  

#####用户  
1. 为了验证client，必须使用**db.createUser()**创建一个用户，同时授予合适的role。第一个创建的用户必须**管理**的role。  
2. 用户对不同的数据库有不同的权限。可能会出现同名用户，但是对不同的数据库有不同的操作。
3. 验证用户可以：a) **在命令行带上-u -p --db**; b) 连接进去后，执行**db.auth()**  
4. 创建用户：db.**createUser**({**user**:'zw',**password**:'1111', **roles**:[{role:'read',db:'ss'},{role:'write',db:'test'}]})    
#####验证机制
**client和mongodb间**，默认采用SDRAM-SHA-1。通过比较用户凭证against用户名+密码+用户db。  
**x.509**: 使用单独的CA（自己维护的或者有资质）颁发的证书。  
1. client和server都要有证书。  
2. client端的证书必须包含：keyUsage=digitalSignature，extendedKeyUsage = **clientAuth**  
3. 每个user有自己的证书。  
4. DN必须和成员证书（member）不一致，否则，会把client认为是复制集或者分片的成员，而赋予全部权限。
5. 对于集群，必须使用相同的**--clusterAuthMode**

#####内部验证
对复制集或者分片集群的**成员之间**进行验证。**keyfiles或者X.509**。**启用内部验证也意味着启用了用户验证**  
keyfiles适用于dev环境，x.509适用于prod环境（**使用证书，而不是用户名密码来进行验证**）。  
x.509使用TLS/SSL连接。
使用x.509的要求：  
1. 所有的成员使用同一个CA颁发的证书；  
2. 证书中的DN，**至少包含**O（orgnization）/OU（orgnization unit）/DC（Domain component）**之一，且内容不为空**。  
3. O/OU/DC必须一致（如果设置，内容一样；或者就都没有设置）  
4. CN（common name）或者SAN（subject altenative name）之一必须和hostname（也许可以使用IP）一致，以便其他成员使用？？  
配置**cilent验证**，mongod或者mongos需要配置** net.ssl.PEMKeyFile***或者使用**--sslPEMKeyFile**指定Pemfile。对于成员证书，如果没有配置**clusterFile **，则使用Pemfile。  
对于复制集或者分片，使用相同的** --clusterAuthMode **。  
为了使用X.509，需要：
1. CLI中添加option：mongod --replSet <name> --sslMode requireSSL --clusterAuthMode x509 --sslClusterFile <path to membership certificate and key PEM file> --sslPEMKeyFile <path to SSL certificate and key PEM file> --sslCAFile <path to root CA PEM file>。 **如果sslCAFile没有指定或者指定的文件不存在，x.509不工作**  
2. 
对于keyfile，其key的长度在6～1024，并只能包含base64的字符。同时在所有的成员（分片包括mongos）上，内容都是一样的。为了使用keyfile，使用**security.keyFile**或者CLI的**--keyFile**选项  
  
#####diagnostic commands
db.runCommand({**collStats**:'articles'，scale:1024})=====>获得collection的统计信息。scale：默认是是byte，要使用KB，设成1024  
db.runCommand({**dbHash**:1, [collectios]})========>获得db中指定的（如不制定，返回所有）collection的hash，以及这些collection的MD5.  
db.runCommnad({**listDatabases**:1})==========>必须在**admin数据库**中执行，返回db一些基本信息。  
**listCommands/buildInfo(当前mongoDB的build信息)/availableQueryOptions**  
**connPoolStats/shardConnPoolStats**：返回当前实例/当前集群的pool。**集群中的连接池用于连接集群间的成员**  
**{dbStats:1,scale:1024}**:db统计信息  
{**dataSize**:"db.coll",**keyPattern**:{field:1}, **min**:{filed2:1}, **max**:{field:10} }  
**netstat**: for mongo**s**  
**profile**: -1:查询当前设置；0：disable；1：enable，记录slow（**slowms**，**执行后返回的结果是上次的设置结果，要查看当前结果，使用-1**）；2：记录所有。**db.getPrifilingStatus(), db.setProfilinigLevel()**  
**validate**: **full:true**:显示详细信息；**scandata：ture**：只验证数据，不验证index。等同**db.coll.validate**  
**top**: 只能运行在admin上，返回一些统计信息。  
**whatsmyuri**: client连接mongo的uri。  
**getLog**:运行在**admin**中，返回RAM中的部分log。**global/rs/startupWarnings**  
**hostInfo**:返回server的cpu信息。  
**serverStatus**：返回服务器状态。**rangeDelete/repl**默认不包含在输出，为了包含，设为1。 {serverStatus:1, rangeDelete:1, repl:1}  


#####role
1. 使用**db.createUser({})**创建用户之后，在**admin**中产生**system.roles**，带有**_id/role/db/privileges/roles**。  
   对于**privileges**，格式为[{resource:[],actions:[]}]，resource格式为:{db:"",collection:""}或者{cluster:true}，如果collection为空，**说明适用在db下所有的collection，但是不包括system.xx**，如果要对系统级的collection设置权限，需要显式设置privilege。  
   对于**roles**，格式如roles: [{ role: "appUser", db: "myApp" }| “myRole”]，指定role(**此role必须定义在同一个db中**)的privilege会赋给当前用户。  
   定义在admin中的role，适用admin和其他db，以及cluster；同时，也能继承定义在admin/其他db/cluster中定义的role。  
2. **db.updateRole(roleName, update, writeConcern)**:  
   必须在role所在的db上执行。  
   完全替换对应field（privilege或者roles）的值，使用**db.grantPrivilegesToRole()/grantRolesToRole()/revokePrivilegesFromRole()/revokeRolesFromRole()**替换。
3. **db.dropAllRoles()**:删除当前db中，所有**user-defined** role



#####选项
--repair: 在mongodump时，只**写入valid的数据**，而排除因为mongo crash活不正常shutdown产生的错误数据。此选项只对mmapv1引擎的mongod生效。**由于采用复杂算法，repair过程中会产生大量的重复数据**
storage.syncPeriodSecs：默认60秒。mongod会即使把数据写入日志，但是每隔一段时间从日志写入data file。
