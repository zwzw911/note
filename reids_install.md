#####window
1. **Redis-x64**-3.0.500-rc1.msi(**redis默认不支持windows，如要下载windows版的redis：https://github.com/MSOpenTech/redis/releases**)  
2. 将C:\Program Files\Redis加入到PATH，以便可以直接使用redis-server。  
3. 安装目录下的文档Windows Service Documentation.docx有详细描述。
4. **redis-server --service-uninstall** 删除window service  
5. **redis-server --service-install**   安装window service  
