#####配置replSet
1. 检测对端：mongo --host 135.252.254.80 --port 27017，能够连接说明本机和对端的连接OK。  
2. mongo的rep set最多支持50个节点，其中最多7个voting节点。voting节点必须是奇数，如果不是，添加一个arbiter节点。  
3. arbiter节点没有数据，同时不能被转换成primary。arbiter必须运行在没有primary或者secondary的节点上。  
4. replication是含有同样数据的mongo实例的集合。一个replication包含若干个数据节点和一个可选的arbiter节点。  
