1. 运行mongodb  
 mongod --logpath /home/db/log/mongodb.log --logRotate rename --timeStampFormat iso8601-local   --dbpath /home/db/  --cpu  
 --logpath /home/db/log/mongodb.log： log信息输入指定文件，而不是打印在屏幕上  
 --logRotate rename： 每次启动mongodb，将之前的log文件改名  
 --timeStampFormat iso8601-local/ctime：log中显示的时间格式，默认是iso8601-local，ctime和iso8601-local类似，只是Date和time间的T没有  
 --dbpath /home/db/： db文件路径
 --cpu： 定时显示cpu和iowait的统计
