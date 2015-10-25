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

#####3. 设置captcha
   创建路径,`mkdir /home/ss-express/captcha_Img`  
   修改ss-express/routes/assist/general.js: **captchaImg_path:['/home/ss-express/captcha_Img']**  
   
#####4. 更改cookie的domian
   修改ss-express/routes/express_component/cookieSession.js， 把domain改成server的ip或者域名（如果有）  
   
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
  general.js/ueUploadPath设置路径，udeitor.config/imagePathFormat设置目录  
  general.js/ueUploadPath:/home/ss-express. **为了使用node来提供静态资源，此目录必须设在ss-express下**，但是可以为软连接，到实际放置文件的地方(windows不可以用快捷方式)
  `mkdir -p /home/inner_image`

#####8. 设置附件上传路径  
  upload_define.js/saveDir:/home/attachment/  
  `mkdir -p /home/attachment`  

#####9. 安装nginx  
首先安装3个包，pcre（必须），openssl和zlib。可以通过源代码安装，也可以使用yum install pcre/openssl/zlib安装。  
但是pcre和zlib的源代码必须下载，安装nginx的时候，需要指明源代码的目录（而不是安装的目录）。  
安装完毕后，`rpm -qa | grep -i pcre`获得包名字，`rpm -ql packagename`获得包安装的地址，一般就是/usr/lib64.  
然后进入nginx源码目录，执行` ./configure --prefix=/usr/local --with-pcre=pcre源代码目录 --with-zlib=zlib源代码目录 --with-openssl=/usr/lib64 && make && make install`.   **注意，--with-pcre和--with-zlib的参数是pcre代码的目录，而不是安装目录**  
	upstream localhost {  
		server localhost:3000;=====>这是nodejs的地址和端口  
		ip_hash;  
	}    
	listen       80;========》这是服务器的端口  
 server_name  192.168.116.131;=====>这是服务器的地址 
 charset utf-8;========>服务器的编码    
 		location = / {  
			proxy_pass http://localhost;=======>接收到进入服务器的request，转发到localhost的地址上  
		}  
		
