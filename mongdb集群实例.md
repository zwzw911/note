手头有3台机器：  
1. shard设置3个，以便每个shard配在不同的PC上，实现负荷分担。每个shard配置成复制集，1P+2S，每个shard的P在不同PC上。3个shard复制集，分别使用27020/27021/27022端口  
2. config server也配置3个，在不同的PC，端口为27019.  
3. mongos和app在同一个server，port：27017  

#####命令
1. 复制集：  
