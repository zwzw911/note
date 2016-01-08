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
 
#####集群
1. 集群中一个节点会被选举为**主节点**(master),它将临时管理集群级别的一些变更，例如**新建或删除索引、增加或移除节点**等。主节点不参与**文档**级别的**变更或搜索**，这意味着在流量增长的时候，该主节点不会成为集群的瓶颈。 
2. 我们能够发送请求给集群中任意一个节点。每个节点都有能力处理任意请求。每个节点都知道任意文档所在的节点，所以也可以将请求转发到需要的节点。这个节点我们将会称之为请求节点(requesting node)。
1. 集群健康： GET **_cluster/health**。至少有2个Node，才能保证状态是GREEN（一个node放主分片，一个node放复制分片）
2. 集群是由cluster.name相同的node组成。node是ES的一个实例。shard是自动分配到不同的shard。
3. 一个分片(shard)是一个最小级别“工作单元(worker unit)”,它只是保存了索引中所有数据的一部分，是一个Lucene实例，它本身就是一个完整的搜索引擎。
4. 分片可以是**主分片(primary shard)**或者是**复制分片(replica shard)**。
5. PUT /index {"settings":{"**number_of_shards**" : 3,"**number_of_replicas**" : 1}}。number_of_shards：index有几个主分片，每个主分片有几个复制分片。


#####文档
1. 文档的元数据:_index(存储和索引数据)/_type（文档代表的对象的类）/_id（文档的唯一标识）。_version/_source
2. 所应（存储？）：PUT /type/index/id: 把文档放到id对应的空间。POST：把文档添加到type下（_id自动增加）。
3. 检索：GET  /index/type/id**/_source**，只显示检索到的内容。  /index/type/id**？_source＝title**（可能有用，只显示某个字段）
4. 是否存在：HEAD: 通过-i参数获得resopnse的header判断，404：不存在；200：存在。  
5. 检查文档是否已经存在：PUT /index/type/id/**_create**。 201：created 409：conflict。 PS；必需是PUT（带ID）。POST的话会自动生成ID。
6. 删除：DELETE  
7. 更新冲突控制：悲观并发控制：将数据锁定，直到操作完毕；乐观并发控制：程序决定冲突后的操作。PUT /index/type/id?**version=1**  当version**等于**1才更改。PUT /index/type/id?**version=10&version_type=external** 当前的version**小于**10才执行update，**并且将version改成10**  
8. 更新: PUT /index/type/id/**_update**。body必需带**doc**，然后带字段：已经存在的字段更新，没有的字段添加。{"doc":{"title":"asdf"}}
9. 获得多个文档： **_mget**。GET /index/type/_mget  {"**ids**":**[**"123","124"**]**}
10. 批量（bulk）：？？  
  
#####搜索
1. 空搜索：不指定index/type/id，直接使用**_search**，获得集群中所有文档。**?timeout=10ms**：搜索超时时间，**不会停止搜索**，而是在达到定义的时间后返回当前搜索到的结果。took:搜索花费的时间。shards：节点告诉我们参与查询的分片数。
2. 多索引和多类别：/index1\*,index2\*,\_all/type1,type2/_search。**通配符\*只能用于index，而不能用于type**  
3. 分页：**_search**只返回10个文档，使用**size（个数）和from（跳过数）**。_search?size=5&from=5。**每个分片返回size个文档，然后请求节点从N\*size个文档中排序获得size个文档**。  
4. 建议搜索：**查询字符串** or **DSL（请求体）**。_search?**q=****+/-**title:test。**+：必需满足（类似AND）**；**-:必需不满足（类似NOT）**。没有+-，可有可无（类似OR）。**不建议使用查询字符串，虽然它功能强大，但是隐晦和调试困难，并且允许任意用户在索引中任何一个字段上运行潜在的慢查询语句，可能暴露私有信息甚至使你的集群瘫痪**  
5. _all：搜索是，ES把所有字段的字符组合成一个字符串，名字是_all。  

#####映射
**mapping**用于字段的数据类型确认（把字段值匹配到一种特定的数据类型：）。  
**analysis**机制用于Full Text的分词。  
1. 查看type的mapping： GET /index/**_mapping**/type?pretty。**不显示_all，因为是默认字段，且类型为string**  
2. 确切值（exact value）和全文文本（Full text）。**确切值：要么匹配，要么不匹配；全文文本：匹配程度。**  
3. 为了对Full text搜索，需要进行analysis，然后建立倒排索引。  
4. **分析器**完成分词，得到terms或者tokens数组的过程。3部分：**字符过滤器**(过滤html标签，或者转换成字符)；**分词器**（根据空格或,进行分词，不适用于中文）;**表征过滤**（修改词（例如将"Quick"转为小写），去掉词（例如停用词像"a"、"and"``"the"等等），或者增加词（例如同义词像"jump"和"leap"））  
5. 内置分析器：**标准分析器**：去掉大部分标点符号，转成小写。**简单分析器**：将非单个字母的文本切分，然后把每个词转为小写。**空格分析器**：依据空格切分文本。它**不转换小写**。**语言分析器**。  
6. 如果**查询字符串是Full text**而不是exact value，那么**查询字符串也要通过分析器进行分析**。  
7. 测试分析器： GET /**_analyze?analyzer=standard**。  
8. 分词类型：**string/whole number/floating number/bollean/date/null/数组/对象**  
    string=>string  
    whole number=>byte/short/integer/long  
    floating number=>float/double  
    boolean=>true/false  
    date=>date   
9. 当为文档添加一个新的字段，ES会用动态映射对字段类型进行推测。查看映射 GET /index/**_mapping**/type。出了string类型，其它类型很少需要映射。string有2个重要的映射参数：**index**和**analyzer**。**index:analyzed/not_anaylzed/no**。analyzed（full Text），not\_analyzed（exact value），no（不索引这个字段。这个字段不能为搜索到。）。对以非字符类型，也可以设置index，但是只能取not_analyzed或者no。对于analyzed类型的字符串字段，使用**analyzer参数来指定哪一种分析器将在搜索和索引的时候使用**。默认的，Elasticsearch使用**standard**分析器，但是你可以通过指定一个内建的分析器来更改它，例如**whitespace、simple或english**。  
10. 定义映射： **新建索引** PUT {"gb":{"**mappings**":{"tweet":{"**properties**":{"content":{"type":"string","**analyzer**":"english"}}}}}}。**添加字段**： **PUT** /gb/\_mapping/tweet {"**properties**":{"tag":{"type":"string","**index**":"**not_analyzed**"}}}。**测试映射**：GET /gb/_analyze?field=tweet -d "{Black-cats}"。
11. **多值对象（数组）**:对于数组**不需要特殊的映射**。任何一个字段可以包含**零个、一个或多个值**，同样对于全文字段将被分析并产生多个词。言外之意，这意味着**数组中所有值必须为同一类型**。你不能把日期和字符窜混合。如果你创建一个新字段，这个字段索引了一个数组，Elasticsearch将使用**第一个值**的类型来确定这个新字段的类型。
12. **空字段**: **""/null/[]/[null]**。
13. **内部对象**：文档字段包含对像。
  
#####结构化查询（搜索）
1. **大多数的参数以JSON格式所容纳而非查询字符串**
2. 使用结构化查询，你需要传递**query**参数。 
3. argument：**match/match_all**
4. 合并多子句：需要**must/must\_not/should**（类似SQL中的AND/NOT/OR）。**bool**。复合子句能合并 任意其他查询子句，包括其他的复合子句。 这就意味着复合子句可以相互嵌套，从而实现非常复杂的逻辑。
5. 查询和过滤：查询：查询语句会询问每个文档的字段值与特定值的匹配程度如何。
6. 查询语句不仅要查找相匹配的文档，还需要计算每个文档的相关性，所以**一般来说**查询语句要比 过滤语句更耗时，并且查询结果也不可缓存。有了倒排索引，一个**只匹配少量文档的简单查询语句**在百万级文档中的查询效率会与一条经过缓存 的过滤语句旗鼓相当，甚至略占上风。**原则上来说，使用查询语句做全文本搜索或其他需要进行相关性评分的时候，剩下的全部用过滤语句**。  
7. 过滤语句：**term**:用于**精确匹配**哪些值，比如数字，日期，布尔值或 not\_analyzed的字符串(未经分析的文本数据类型)。**terms**：terms 跟 term 有点类似，但 terms 允许指定多个匹配条件。 如果某个字段指定了多个值（**数组**），那么文档需要一起去做匹配。**range**：**gt/gte/lt/lte**。**exists/missing**:查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的IS_NULL条件。**bool**
8. 查询语句：**match**：查询是一个标准查询，不管你需要全文本查询还是精确查询（**精确查询最好用过滤，因为可以缓存**）基本上都要用到它。**march_all**：可以查询到所有文档，是没有查询条件下的默认语句。**multi_match**:查询允许你做match查询的基础上同时搜索多个字段。"**multi_match**":{"**query**":"full text search","**fields**":["title","body"]}。**bool查询**：bool 查询与 bool 过滤相似，用于**合并多个查询子句**。不同的是，bool 过滤可以直接给出是否匹配成功， 而bool查询要**计算每一个查询子句的 _score** （相关性分值）。must:: 查询指定文档**一定要被包含**。must_not:: 查询指定文档一定**不要**被包含。should::查询指定文档，有则可以为**文档相关性加分**（和bool过滤不同，should在此是加分的作用）。   
#####排序
1. 默认按照**相关性**排序。相关性指\_score。某些情况下，\_score是一样的（使用过滤来查询文档，\_score都是1）
2. GET \_search {"query":{"filtered":{"filter":{"term":{"user_id":1}}}},"**sort**":{"date":{"**order**":"**desc**"}}}。
3. sort添加一个sort字段，用来排序。如果sort的字段不是\_score，则\_score和max\_score为null，因为此时无需score（计算score耗费资源）。如要强迫开启：track\_scores设为true。**sort也可以使用\_score进行排序**
4. 如果要对多个字段排序，sort为数组。"sort":**[**{"date":{"order":"desc"}},{"\_score":{"order":"desc"}}**]**
5. 查询字符串：GET /\_search?sort=date:desc&sort=_score&q=search
6. 多值（数组）字段排序："sort":{"dates":{"order":"asc",**"mode":"min"**}}。模式可以是min/max/sum/avg。
#####分布式CRUD  
1. 确定shard的位置：** = hash(routing) % number_of_primary_shards**。routing值是一个任意字符串，它默认是_id但也可以自定义。自定义路由值可以确保所有相关文档——例如属于同一个人的文档——被保存在同一分片上。   
2.   
#####
1. _search: type下所有数据
2. _search?q=lastname:smith：q=设置条件
3. {query:{match:{lastname:smith}}}: query is q=;
4. match/match_phrase: match：包含匹配；match_phrase：全部匹配
    
#####分词
1. curl -XGET "http://localhost:9200/_analyze?analyzer=standard" -d "text to text"
2.  

