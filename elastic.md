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
4. 执行service（不需要bat） **manager**，检查elastic已经安装成service 。或者使用浏览器或者curl，get本地地址http://localhost:9200/?pretty，如果有信息返回，说明service已经启动。或者**service.msc**打开windows服务，直接诶查看。

#####安装plugin
1. https://www.elastic.co/downloads/marvel
2. npm install elasticsearch --save

#####配置
1.  elasticsearch.yml: node.name=>节点名称。所有同名节点属于同一个集群。
2.  

#####连接
**curl -X\<VERB> "\<PROTOCOL>://\<HOST>/\<PATH>?\<QUERY_STRING>" -d "\<BODY>"**  
**注意，是双引号隔开，而不是单引号**  
**body中的双引号，需要用\进行转义**  
    VERB HTTP方法：GET, POST, PUT, HEAD（文档是否存在）, DELETE  
    PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）  
    HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost  
    PORT Elasticsearch HTTP服务所在的端口，默认为9200  
    QUERY_STRING 一些可选的查询请求参数，例如?pretty参数将使请求返回更加美观易读的JSON数据  
    BODY 一个JSON格式的请求主体（如果请求需要的话）  

**pretty**：美化输出
  
#####文档
1. 文档的元数据:_index(存储和索引数据)/_type（文档代表的对象的类）/_id（文档的唯一标识）。_version/_source
2. 所应（存储？）：PUT /type/index/id: 把文档放到id对应的空间。POST：把文档添加到type下（_id自动增加）。
3. 检索：GET  /index/type/id**/_source**，只显示检索到的内容。  /index/type/id**？_source＝title**（可能有用，只显示某个字段）
4. 是否存在：HEAD: 通过-i参数获得resopnse的header判断，404：不存在；200：存在。  
5. 
#####
1. _search: type下所有数据
2. _search?q=lastname:smith：q=设置条件
3. {query:{match:{lastname:smith}}}: query is q=;
4. match/match_phrase: match：包含匹配；match_phrase：全部匹配
    
#####分词
1. curl -XGET "http://localhost:9200/_analyze?analyzer=standard" -d "text to text"
2. 分词类型：string/whole number/floating number/bollean/date
    string=>string  
    whole number=>byte/short/integer/long  
    floating number=>float/double  
    boolean=>true/false  
    date=>date  

