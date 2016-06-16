#####replSet
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


#####配置命令
1. 首先**分别在所有的节点上各自**启动一个mongod进程，出了bing_ip不同，其他尽量一致
C:\Users\lte>mongod --bind_ip SH-JQ-254-77 --logpath D:/ss_log/mongo/mongodb.log --logRotate reopen --logappend --timeStampFormat iso8601-local --oplogSize 1000 --dbpath D:/ss_db/mongo/ --replSet "ss"
没有采用mongod -f xxx.conf的方式，因为window的mongodb版本是3.0，不支持某些选项。而直接采用命令行的方式。  
--bind_ip：必须是主机名，而不是IP地址，例如127.0.0.1（否则无法区分primary/secondary）
--replSet：复制集合名称  
--oplogSize: primary的oplog大小，默认是磁盘剩余空间的5%。如果设置，单位是MB。 
2. 联入此mongod，在**单独一个节点**上执行initiate命令，完成replication需要的oplog文件的创建。单个文件最大2GB，如果oplogSize小于2GB，只创建一个文件local.1。    
**rs.initiate()**  
{  
        "info2" : "no configuration explicitly specified -- making one",  
        "me" : "SH-JQ-254-77:27017",  
        "ok" : 1  
}  
3. 命令  
   rs.conf():
   rs.isMaster():
