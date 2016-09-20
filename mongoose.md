###概念  
1. schema：定义collection的结构。  var personSchema=new mongoos.Schema({name:String,age:number})  
2. model：由schema生成。  var personModel=db.model('personModel',personSchema)    
3. entity：由model生成的实际document。  var entity=personModel({name:'zw',age:18})  

###数据类型  
Mongoose包含了nodejs中的数据类型，并添加了自己的类型。  
String/Buffer/Boolean/Date/Number/Schema.Types.Mixed/Schema.Types.ObjectId/Array。**js中，array不是数组，而是集合，所以可以容纳不同的数据类型**。  
Schema.Types.Mixed类似nested，区别在，mixed没有定义具体成员，而nested定义了。所以，mixed可以接受不同的数据成员（但是save之前需要调用markModified()）在定义Mixed数据类型的时候，以下相等。  
    var AnySchema = new Schema({any:{}});
    var AnySchema = new Schema({any:Schema.Types.Mixed});

###schema  
#####方法
静态方法：在**model**上即可使用。 Schema.**statics**.func=function(param1,cb){}  
动态方法：在**entity**上使用。Schema.**methods**.func=function(param1,cb){}  
虚拟方法：在**entity**上使用。get和set方法的区别：get需要**return**，返回指定path的value；set，对传入的参数处理，设置给entity的path。 *path就是field的意思*  
schema.virtual('name.full').**get**(function(){**return** this.name.full=this.name.firstname+this.name.lastname})。 schema.virtual('name.full').**set**(function(**name**){ this.name.firstname=this.name.full.firstname})  
#####option  
1. safe：默认true。也可以设成{j:1,w:2,timeout:1000}    
2. strict: 默认true：如果entity中的path没有在schema中定义，则此path无法保存（entity可以保存）；false，则可以保存。**而不是path的值没有定义，就无法保存**。     
3. shardKey：mongodb为分片时使用。  
4. capped：批量操作的上限。  {capped:1000}:每次最多操作1000个document；还可以{capped:{size:1000,max:100,autoIndexId:true}}   
5. versionKey:版本所，默认true。    
6. autoIndex:  

###CRUD
1. 查询  
   1.1 直接查询：带回调  
   1.2 链式查询：初始返回**query**对象。**query对象是尚未执行的预编译查询语句**。之后可以任意添加查询条件，最后exec带回调。 
   1.3 方法：在model上执行find/findOne/findById  
    ####Model.find(conditions, [projection], [options], [callback])  
    `MyModel.find({ name: /john/i, age:{$gte:18}}}, 'name', { skip: 10 }, function (err, docs) {});` 
    `MyModel.find({ name: /john/i, age:{$gte:18}}}).select(name).skip(10).exec();`   
    projection: 可选，返回哪些字段。  
    ####Model.findOne(conditions, [projection], [options], [callback])     
    和find类似。conditions可以为空，此时返回任一document。  
    ####Model.findById(id, [projection], [options], [callback])  
   `model.findById(id) = model.find({_id:id})`      
    `model.findById(id,'field1 field2', {lean:true}, function(err,doc){})`  or `model.findById(id,'field1 field2').lean().exec(cb)`  
    lean: doc为js object而不是mongoose document   

    ####Document#get(path, [type])  
    获得字段的值（可以无视doc是js obj还是mongoose document）。 
    path：字段。  
    type：数据类型。string等。将获得的值转化成type类型。  
    
    ####Document#equals(doc)  
    根据_id判断当前doc是否和doc相等。如果没有_id，使用deepEqual()比较。  
    
    
2. update  
   2.1 传统：读取document，修改，保存
    *PersonModel.findById(id,function(err,person){  
      person.name = 'MDragon';  
      person.save(function(err){});  
    });*  
    2.2 update：读取，删除，update。比较麻烦     
    2.3 **update+$set**：更新少量字段时方便  
    *PersonModel.update({_id:_id},{$set:{name:'MDragon'}},function(err,person){});*  
    2.4 方法：  
    ####Model.findByIdAndUpdate(id, [update], [options], [callback])  
    根据id找到一个doc，并触发mongodb的findAndModify命令，根据[update]进行更新。  
    *options*:  
        new: bool。如果true，返回更新过的document。默认false。    
        upsert:bool。如果true，更新文档不存在则创建新文档。默认false。  
        runValidators：bool。如果true，在update的时候，执行schema上的validator（因为某些caverts，默认false）  
        setDefaultsOnInsert：bool。当和upsert同时为true时，在插入新文档时，使用默认值。  
        sort：如果找到多个文档（应该不太可能），按照什么顺序选择第一个文档进行update。  
        select: 返回哪些字段。
    `Model.findByIdAndUpdate(id, { name: 'jason borne' }, options, callback)`  
    ####Document#markModified(path)   
    对于mixed(nested)文档有用：标记此文档的此field已经修改，save的时候必须记录。  
3. insert  
    3.1 entity的save  
    3.2 model的create：create的对象只能是JSON（即要保存的数据本身），而不是entity，因为entity虽然只是打印数据，但实际上包含了schema和model的行为（例如，动态静态方法）等其他属性，不是纯粹的数据  
    ####Model.create(doc(s), [callback])  
    保存一个或者多个文档。等同于为每个文档调用new Model(doc).save()  

    ####Model.insertMany(doc(s), [callback])  
    一次插入多个文档。比create快。  
    `var arr = [{ name: 'Star Wars' }, { name: 'The Empire Strikes Back' }];`  
    `Movies.insertMany(arr, function(error, docs) {});`  

4. remove: entity和model都有remove  
    ####Model.findByIdAndRemove(id, [options], [callback])   
    *等于Model.findOneAndRemove({_id:id}, [options], [callback])*  

###子文档
子文档的操作都在副文档上执行。  
    var ChildSchema1 = new Schema({name:String});  
    var ChildSchema2 = new Schema({name:String});  
    var ParentSchema = new Schema({  
      children1:ChildSchema1,   //嵌套Document  
      children2:[ChildSchema2]  //嵌套Documents  
    });  
    
###验证器
1. **验证器是中间件**。  
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
   自定义：**可以在schema上定义（推荐），或者在model.schema上定义**。validate带2个参数，**验证函数和错误消息**  
   *Toy.schema.path('color').validate(function (value) {  
    return /blue|green|white|red|orange|periwinkel/i.test(value);  
   }, 'Invalid color');*  
2. 验证失败返回err
   如果验证失败，则会返回err信息，err是一个对象该对象属性如下  
    err.errors                //错误集合（对象）  
    err.errors.color          //错误属性(Schema的color属性)  
    err.errors.color.message  //错误属性信息  
    err.errors.path             //错误属性路径  
    err.errors.type             //错误类型  
    err.name                //错误名称  
    err.message                 //错误消息  

**Validation错误后，entity也将有相同的errors属性**  
toy.errors.color.message === err.errors.color.message  

###中间件
中间件在**schema**定义，是一种控制函数，类似插件。  
分成2大类：**document和query**中间件
document中间件适用document的**init、validate、save、remove**方法。  
query中间件适用**Model和query**的**count/find/findOne/findOneAndRemove/findOneAndUpdate/update**
中间件分成pre和post2种。  
pre**会在方法（init/vlidate/save/remove）执行前操作**。pre分成串行和并行2中，带有流控制（通过next和done）  
post**活在pre和方法执行之后操作**。post无流控（因为此时方法已经完成）。  
**全部中间件执行完后执行其他操作**，使用中间件可以雾化模型，避免异步操作的层层迭代嵌套  

使用范畴：
复杂的验证  
删除有主外关联的doc  
异步默认  
某个特定动作触发异步任务，例如触发自定义事件和通知  

**pre中间件**分成2类：串行和并行。  
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
    
**POST** 
schema.post('init', function (doc) {  
  console.log('%s has been initialized from the db', doc._id);  
})  


###population  
0. 语法  
####Model.populate(docs, options, [callback(err,doc)])  
doc(s)：可以是单个doc，也可以是doc**数组**。**doc可以是js object，也可以是mongoose doc**。  
options：  
    path：空格区分的外键（此键关联到其他doc）。  
    select：  
    match：  
    model：  
    options： 查询选项，如sort，limit等。  

####Document#execPopulate()  
返回一个promise，使用于ES6/7.  

####Document#depopulate(path)  
对一个已经populate的path，返回unpopulate状态（即获得字段的原始值）  

1. 定义:  
var storySchema = Schema({  
  _creator : { type: Number, **ref: 'Person'** } //PersonModel  
})  

2. 使用：**可以通过model.find到entity后执行，或者直接在entity上执行(在entity执行还可以populate多个document？？？)**  
*Story  
.find(...)  
.populate({  
  path: 'fans',  
  match: { age: { $gte: 21 }},  
  select: 'name _id',//多个字段，用空格分隔    
  options: { limit: 5 }  
})*  
.exec()


###connection options  
1. db/server/replset/user/pass/auth/mongos  
var options = {  
  db: { native_parser: true },  
  server: { poolSize: 5 },  
  replset: { rs_name: 'myReplicaSetName' },  
  user: 'myUserName',  
  pass: 'myPassword'  
}  
2. keepAlive  
options.**server.socketOptions** = options.**replset.socketOptions** = { **keepAlive**: 1 };  
mongoose.connect(uri, options);  
3. 连接副本集（一般用不到，因为都直接连接mongos，通过mongos来决定连接到哪个副本）  
mongoose.connect('mongodb://username:password[@host](/user/host):port/database,mongodb://username:password[@host](/user/host):port,mongodb://username:password[@host](/user/host):port?options...' [, options]);  
4. 连接**mongos副本集**  
mongoose.connect('mongodb://mongosA:27501,mongosB:27501', { mongos: true }, cb);  
5. 连接多个uri（**连接不同的db**）
`var conn = mongoose.createConnection('uri,uri,uri...', options);`  
6. 连接池：mongoose内部维护一个连接池，默认大小5，可以更改：  
// single server  
var uri = 'mongodb://localhost/test';  
mongoose.createConnection(uri, { server: { poolSize: 4 }});  

// for a replica set  
mongoose.createConnection(uri, { replset: { poolSize: 4 }});  

// passing the option in the URI works with single or replica sets  
var uri = 'mongodb://localhost/test?poolSize=4';  
mongoose.createConnection(uri);  
