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

###CRUD
1. 查询  
   1.1 直接查询：带回调  
   1.2 链式查询：初始返回**query**对象。**query对象是尚未执行的预编译查询语句**。之后可以任意添加查询条件，最后exec带回调。  
2. update  
   2.1 传统：读取document，修改，保存
    *PersonModel.findById(id,function(err,person){  
      person.name = 'MDragon';  
      person.save(function(err){});  
    });*  
    2.2 update：读取，删除，update。比较麻烦     
    2.3 **update+$set**：更新少量字段时方便  
    *PersonModel.update({_id:_id},{$set:{name:'MDragon'}},function(err,person){});*  
3. insert  
    3.1 entity的save  
    3.2 model的create：create的对象只能是JSON（即要保存的数据本身），而不是entity，因为entity虽然只是打印数据，但实际上包含了schema和model的行为（例如，动态静态方法）等其他属性，不是纯粹的数据  
4. remove: entity和model都有remove  

###子文档
子文档的操作都在副文档上执行。  
    var ChildSchema1 = new Schema({name:String});  
    var ChildSchema2 = new Schema({name:String});  
    var ParentSchema = new Schema({  
      children1:ChildSchema1,   //嵌套Document  
      children2:[ChildSchema2]  //嵌套Documents  
    });  
    
###验证器
1. 验证器是中间件。  
2. validate应用在使用默认值（如果定义了default），并准备保存之前。  
1. 分为2种：内部和自定义。  
   内部：  
      **require**:非空验证。  
      **min/max**： 边界验证。  
      **enum/match**：枚举验证/匹配验证  
      **validate**：自定义验证规则  
      *var PersonSchema = new Schema({  
      name:{  
        type:'String',  
        required:true //姓名非空  
      },  
      age:{  
        type:'Nunmer',  
        min:18,       //年龄最小18  
        max:120     //年龄最大120  
      },  
      city:{  
        type:'String',  
        enum:['北京','上海']  //只能是北京、上海人  
      },  
      other:{  
        type:'String',  
        validate:[validator,err]  //validator是一个验证函数，err是验证失败的错误信息  
      }  
    });*  
2. 验证失败返回err
   如果验证失败，则会返回err信息，err是一个对象该对象属性如下  
    err.errors                //错误集合（对象）  
    err.errors.color          //错误属性(Schema的color属性)  
    err.errors.color.message  //错误属性信息  
    err.errors.path             //错误属性路径  
    err.errors.type             //错误类型  
    err.name                //错误名称  
    err.message                 //错误消息  


###中间件
中间件在**schema**定义，是一种控制函数，类似插件，能控制流程中的**init、validate、save、remove**方法。**一旦定义了中间件，就会在全部中间件执行完后执行其他操作**，使用中间件可以雾化模型，避免异步操作的层层迭代嵌套  

使用范畴：
复杂的验证  
删除有主外关联的doc  
异步默认  
某个特定动作触发异步任务，例如触发自定义事件和通知  

中间件分成2类：串行和并行。  
    var schema = new Schema(...);  
    schema.pre('save',function(next){  
      //做点什么  
      **next()**;  
    });  
    var schema = new Schema(...);  
    schema.pre('save',function(next,done){  
      //下一个要执行的中间件并行执行  
      **next();**  
      **doAsync(done);**  
    });      
    
