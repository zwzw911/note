##npm:  
**--save**:运行程序，此安装包是必须的.  
**--save-dev**：此安装包是辅助的，例如，开发完毕后，用来压缩，优化css/js/html。这些包不需要部署，所以放在devDependcy下。

npm **-init**：创建packet.json文件

#####设置代理
npm config set proxy http://135.245.48.14:8000
npm config set https-proxy https://135.245.48.14:8000  
执行后，自动写入配置文件  

#####获得配置文件路径  
npm config get userconfig  
