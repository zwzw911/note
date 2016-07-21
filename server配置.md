#####0. 安装必须文件  
######windows  
1. 拷贝**mongodb**-win32-x86_**64**-3.0.1-signed.msi, **Redis-x64**-3.0.500-rc1.msi, **nginx**-1.8.0.tar.gz，  
2. msi直接点击安装。nginx直接解压
3. 配置nginx的http  
3.1 设置上传最大不超过2M：client_max_body_size 2M;  
3.2 设置gzip
    	gzip  on;==========>打开gzip  
	gzip_min_length  1000;=======>当内容能够超过1000byte才使用gzip压缩  
	gzip_buffers 4 16k;========>以4*16K为单位，申请内存用于缓存gzip的结果  
	\#gzip_http_version 1.0;===========>默认1.1，只对http1.1的request的结果进行gzip；如果nginx用作反向代理服务器，此时nginx和其他server用http1.0通信  
	gzip_comp_level 3;==========>1~9 压缩比例越大，速度越慢  
	gzip_proxied     expired no-cache no-store private auth;  
	gzip_types       text/plain text/css application/x-javascript application/javascript application/xml image/jpeg   image/gif image/png;===============>对header中mime那些类型进行压缩  
	gzip_disable "MSIE [1-6]\.";==========>不支持IE6(对image进行gzip会造成IE6假死)  
3.3 设置负载均衡（暂时不用，需要从网址映射到IP）
	upstream 172.24.251.205 {
		server 127.0.0.1:3000;**这是nodejs提供的访问地址**  
		ip_hash;
	}  
3.4 配置server：  
    server_name 172.24.251.205;==============>外部IP（可以是网址），以便访问  
    charset **utf-8**  
4. 安装mongodb和nginx为window服务  
   4.1 在windows的PATH，添加mongodb安装路径（C:\Program Files\MongoDB\Server\3.0\bin）  
       然后再cmd中，执行**mongod --logpath D:/ss_db/mongo/mongodb.log --logRotate rename --timeStampFormat iso8601-local --dbpath D:/ss_db/mongo/ --replSet ss/135.252.254.80:27017 --serviceName MongoDB --install**  
	可以通过命令net start/stop MongoDB来启动/停止mongo服务  
	--repSet ss/135.252.254.80:27017: 副本集中其它任意一个副本

#####1. 运行mongodb  
 mongod --logpath /home/db/log/mongodb.log --logRotate rename --timeStampFormat iso8601-local   --dbpath /home/db/  --cpu  
 --logpath /home/db/log/mongodb.log： log信息输入指定文件，而不是打印在屏幕上  
 --logRotate rename： 每次启动mongodb，将之前的log文件改名  
 --timeStampFormat iso8601-local/ctime：log中显示的时间格式，默认是iso8601-local，ctime和iso8601-local类似，只是Date和time间的T没有  
 --dbpath /home/db/： db文件路径  
 --cpu： 定时显示cpu和iowait的统计  
--dbpath /home/db/：不同的db使用不同的文件。不能使用，编译好的文件已经disable这个功能，除非重新从源代码编译  

#####2. 生成pem密钥  
首先`yum update openssl`,升级到最新版本  1.0.1e-42.el7.9
然后在一个指定目录下，执行`openssl genrsa -out key.pem 1024`, 产生私钥  
继续执行`openssl req -key key.pem -new -x509 -out cert.pem`,一路回车，生成公钥  
修改ss-express/routes/assist/general.js，将pemPath改成key.pem的路径

#####3. 修改当前server匹配的参数
3.1 更改cookie的domian  
   ~~修改ss-express/routes/express_component/cookieSession.js， 把domain改成server的ip或者域名（如果有）~~。直接从assist/general.js中读取  
   
3.2 修改app.js  
app.set("env","dev")====>app.set("env","pro")  

3.3 assist/upload_define.js    
saveDir: attachment目录    

3.4 修改assist/general.js（大量的参数实际还是采用此文件中的参数，而不是redis中的参数）    
reqHostname: 用在cookieSession  127.0.0.1->135.252.254.77**此处IP地址为浏览器访问的IP**  
port: 同上**此处IP地址为浏览器访问的PORT，默认80**  
ueUploadPath: ue上传的图片放置的目录，和**ueditor_config.js**中的imagePathFormat共同组成 文件上传目录。例如D:/ss_file+inner_image。  
searchResultPageSize：每页显示搜索结果的数量  
articleFolderPageSize： 用户文档中每页显示的文档数量  
pemPath：  

3.5 修改routes/inputDefine/adminLogin/defaultGlobalSetting.js  
reqHostName: 127.0.0.1->135.252.254.77**此处IP地址为浏览器访问的IP**  
port: 同上**此处IP地址为浏览器访问的PORT，默认80**  
pemPath   
globalSettingBackupPath: 备份全局设置的文件位置  
以下为defaultSetting下的item
inner_image/default:   
user_icon/fileName:  
user_icon/uploadDir:  
attachment/saveDir:  
Lua/scriptPath:  

#####4. 载入defaultGlobalSetting（**第一次部署才运行**）  
运行脚本maintaince/setDefaultGlobalSetting.bat 或者 直接 `node setDefaultGlobalSetting.js`，将默认设置写入redis。  

#####5. 安装Lua SHA为service
直接运行运行脚本maintaince/before_launch/preCheckBeforeLaunch.bat 或者 `node preCheckBeforeLaunch.js`  
~~C:\Users\lte>sc create ss_pre binPath= "C:\Program Files\nodejs\node.exe D:\ss_dist\maintaince\before_launch\preCheckBeforeLaunch.js"~~  
~~[SC] CreateService SUCCESS~~  
~~**注意，binPath=后是个空格**~~

#####6. 数据迁移
1. 数据导出  
**monogdump** --host 127.0.0.1 --port 27017 -d ss -o c:/  
-d: 数据库名，不写是**所有**db  
-o: 导出数据存放的目录  
2. 数据导入   
mongorestore --host 135.252.254.77 --port 27018 -d ss --drop c:/
--drop：删除db，无参数删除所有？？
c:/  : 要导入的数据  
  

#####5. 安装font
   生成captcha需要用到某种字体。  
   首先`fc-cache -vfs`查看当前是否有字体，如果没有，安装基本字体`yum install liberation-sans-fonts.noarch liberation-mono-fonts.noarch liberation-serif-fonts.noarch`, 然后查看当前可用字体`fc-match`，查看所有字体`fc-list :lang=zh/en`  

#####6. 更改时区  
  1. 查看:`timedatectl`   
  2. 列出时区:`timedatectl list-timezones`  
  3. 设置时区:`tiemdatectrl set-timezone Asia/Shanghai`  
  4. 设置时间:  `timedatectrl set-time "YYYY-MM-DD hh:mm:ss"`  
 
#####7. 设置ueditor上传路径  
  ueditor上传路径由2部分组成：  
  assist/general.js/ueUploadPath设置路径，assist/udeitor.config/imagePathFormat设置目录  
  general.js/ueUploadPath:/home/ss-express. ~~为了使用node来提供静态资源，此目录必须设在ss-express下，但是可以为软连接，到实际放置文件的地方(windows不可以用快捷方式)~~  
  **部署时候使用ngnix，可以自由配置静态资源**  
  `mkdir -p /home/inner_image`  

#####10. 安装nginx  
首先安装3个包，pcre（必须），openssl和zlib。可以通过源代码安装，也可以使用yum install pcre/openssl/zlib安装。  
但是pcre和zlib的源代码必须下载，安装nginx的时候，需要指明源代码的目录（而不是安装的目录）。  
安装完毕后，`rpm -qa | grep -i pcre`获得包名字，`rpm -ql packagename`获得包安装的地址，一般就是/usr/lib64.  
然后进入nginx源码目录，执行` ./configure --prefix=/usr/local --with-pcre=pcre源代码目录 --with-zlib=zlib源代码目录 --with-openssl=/usr/lib64 && make && make install`.   **注意，--with-pcre和--with-zlib的参数是pcre代码的目录，而不是安装目录** 
  
gzip  on;	*启用gzip*
gzip_min_length  1000	*内容长度大于1000byte才用gzip压缩(小于1000bytes,压缩之后可能会更大)*
gzip_buffers 4 16k;	*压缩使用的buffer大小, 4*16k*  
\#gzip_http_version 1.0;  
gzip_comp_level 3;	*压缩比率,1-9, 1:不压缩,9:压缩最大(cpu消耗最多)*  
gzip_proxied     expired no-cache no-store private auth;  
gzip_types       text/plain text/css application/x-javascript application/javascript application/xml image/jpeg image/gif image/png;	*压缩类型*   
gzip_disable "MSIE [1-6]\.";	*IE6以及更老版本不压缩*  
curl -I -H "Accept-Encoding:gzip,deflate" 127.0.0.1:	*检测是否启动了gzip*  
curl -I -H "Accept-Encoding:gzip,deflate" 127.0.0.1/image/logo.png:	*检测是否启动了gzip*  
  
upstream localhost 
{  
server localhost:3000;	*这是nodejs的地址和端口*  
ip_hash;  
}    
listen       80;	*这是服务器的端口*  
 server_name  192.168.116.131;	*这是服务器的地址*  
 charset utf-8;	*服务器的编码*    
location = / {  
proxy_pass http://localhost:3000;	*ngnix服务器的接收到的request，转发到nodejs的地址上*
}  
		
