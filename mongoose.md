###概念  
1. schema：定义collection的结构。  var personSchema=new mongoos>schema({name:String,age:number})  
2. model：由schema生成。  var personModel=db.model('personModel',personSchema)    
3. entity：由model生成的实际document。  var entity=personModel({name:'zw',age:18})  

###数据类型  
String/Buffer/Boolean/Date/Number/Mixed/ObjectId/Array。**js中，array不是数组，而是集合，所以可以容纳不同的数据类型**。  

###schema  
#####方法
静态方法：在**model**上即可使用。 Schema.**statics**.func=function(param1,cb){}  
动态方法：在**entity**上使用。Schema.**methods**.func=function(param1,cb){}  
虚拟方法：在**entity**上使用。get和set方法的区别：get需要**return**，返回指定path的value；set，对传入的参数处理，设置给entity的path。 *path就是field的意思*  
schema.virtual('name.full').**get**(function(){**return** this.name.full=this.name.firstname+this.name.lastname})。 schema.virtual('name.full').**set**(function(**name**){ this.name.firstname=this.name.full.firstname})  
#####option  
1. safe：默认true。也可以设成{j:1,w:2,timeout:1000}    
2. strict: 默认true：如果entity中的path没有在schema中定义，则此entity无法保存；false，则可以保存。**而不是path的值没有定义，就无法保存**。     
3. shardKey：mongodb为分片时使用。  
4. capped：批量操作的上限。  {capped:1000}:每次最多操作1000个document；还可以{capped:{size:1000,max:100,autoIndexId:true}}   
5. versionKey:版本所，默认true。    
6. autoIndex:  

