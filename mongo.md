#####配置replSet
1. 检测对端：mongo --host 135.252.254.80 --port 27017，能够连接说明本机和对端的连接OK。  
2. mongo的rep set最多支持50个节点，其中最多7个voting节点。voting节点必须是奇数，如果不是，添加一个arbiter节点。  
3. arbiter节点没有数据，同时不能被转换成primary。arbiter必须运行在没有primary或者secondary的节点上。  
4. replication是含有同样数据的mongo实例的集合。一个replication包含若干个数据节点和一个可选的arbiter节点。数据节点中，只有一个primary，剩下的都是secondary和voting节点。只有primary能处理write操作；secondary通过复制primary的oplog，（执行后）复制数据。
5. secondary可以通过一些参数配置，得到特殊的功能：  
  5.1 优先级为0：能接受read，也能进行vote，但是不会触发选举，永远是secondary（cold standby）  
  5.2 隐藏节点：不接受read，能进行vote，防止application从中读取数据  
  5.3 延时节点：通过一个运行中的“历史”snapshot，防止意外操作被记录下来。延时节点必须被设置成优先级0，且为隐藏节点。     
5. 当primary停止响应超过10秒后，合适的secondary发起选举，选自己为primary，成员通过则变成primary。3.2采用protocol 1来节省故障回滚时间。
