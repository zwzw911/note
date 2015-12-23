###安装
####windows
#####下载
1. 在**https://www.elastic.co/downloads/elasticsearch**下，点击**zip**，以便下载windows版本
2. 下载完毕，解压。
3. 检查是否安装了java。C:\Program Files\Java\jdk1.8.0_66。如果没有，那么从www.java.com下载最新版本安装。  

#####配置  
1. 打开**环境变量**，添加一个新的变量，"JAVA_HOME"，设成jdk所在目录。例如JAVA_HOME=C:\Program Files\Java\jdk1.8.0_66.
2. path中加入%JAVA_HOME%（就是step1中设置的值）。可能为optional。
3. path中加入elastic解压后所在的目录。例如：C:\Users\wzhan039\Downloads\elasticsearch-2.1.1\elasticsearch-2.1.1
3. 打开cmd，~~进入elastic解压目录，进入bin~~，执行service.bat **install**。将elastic安装成一个windows service。因为step3已经把elastic的目录加入path变量了
4. 执行service（不需要bat） **manager**，检查elastic已经安装成service 。
