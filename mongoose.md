#####概念  
1. schema：定义collection的结构。  var personSchema=new mongoos>schema({name:String,age:number})  
2. model：由schema生成。  var personModel=db.model('personModel',personSchema)    
3. entity：由model生成的实际document。  var entity=personModel({name:'zw',age:18})  

#####数据类型  
String/Buffer/Boolean/Date/Number/Mixed/ObjectId/Array。**js中，array不是数组，而是集合，所以可以容纳不同的数据类型**。  

#####动态静态方法
都定义在schema上。  
静态方法：在**model**上即可使用。 Schema.**statics**.func=function(param1,cb){}  
动态方法：在**entity**上使用。Schema.**methods**.func=function(param1,cb){}  
