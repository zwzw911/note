1. 运行mongodb  
 mongod --logpath /home/db/log/mongodb.log --logRotate rename --timeStampFormat iso8601-local   --dbpath /home/db/  --cpu  
 --logpath /home/db/log/mongodb.log： log信息输入指定文件，而不是打印在屏幕上  
 --logRotate rename： 每次启动mongodb，将之前的log文件改名  
 --timeStampFormat iso8601-local/ctime：log中显示的时间格式，默认是iso8601-local，ctime和iso8601-local类似，只是Date和time间的T没有  
 --dbpath /home/db/： db文件路径
 --cpu： 定时显示cpu和iowait的统计  
--dbpath /home/db/：不同的db使用不同的文件。不能使用，编译好的文件已经disable这个功能，除非重新从源代码编译  

2. 生成pem密钥  
首先`yum update openssl`,升级到最新版本  1.0.1e-42.el7.9
然后在一个指定目录下，执行`openssl genrsa -out key.pem 1024`, 产生私钥  
继续执行`openssl req -key key.pem -new -x509 -out cert.pem`,一路回车，生成公钥  
修改ss-express/routes/assist/general.js，将pemPath改成key.pem的路径

3. 修改ss-express/routes/assist/general.js  
   captchaImg_path:['/home/ss-express/captcha_Img'],
