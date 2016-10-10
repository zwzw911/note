### connection  
2种方式：**connect**或者**createConnection**，2者参数一样，但是前者创建默认的connection，后者显示的创建一个connection（每个connection是mongoose的一个实例，对应一个db）。  
建议采用**createConnection()**。  

### Promise 
mongoose默认使用mpromise，但是将会在5.0废除。建议使用原生或者第三方Promise。  
使用ES6原生Promise。
`var mongoose=require('mongoose); mongoose.Promise=Promise`

### Populate  
1. 如果要对多个document进行populate，如下格式：  
`Model.find(condition,selectField,option).populate(opt).exec(function(err,result){})`  
**condition**: 查询条件。例如{name:/w/}  
**selectField**: null,忽略 ; 'field1 field2': 显示field1 field2， '-field1 -field2': 显示所有，除了field1/field2；   
**option**；sort/limit等。  
**opt**：populate的选项  
path:'department',//需要populate的字段  
select:'name',//populate后，需要显示的字段  
match:{},//populate后，过滤字段(不符合这显示null)。一般不用  
options:{},//{sort:{name:-1}}  

### C(RUD)  
####新建记录，使用create或者insertMulti  
**Model.create(doc(s), [callback])**：docs中每个记率转换成new Model(doc).save()执行。返回Promise。  
**Model.insertMany(doc(s), [callback])**: 参数docs为**数组**。比create快，因为将所有记录一次发送给mongodb（而不是每个记录执行save())。返回Promise。  
####更新记录：findByIdAndUpdate  
传统方式：查找文档，修改并保存。  
改进方式：update+$set，适用于少量字段。`PersonModel.update({_id:_id},{$set:{name:'MDragon'}},function(err,person){});`。  
推荐方式：**Model.findByIdAndUpdate(id, { name: 'jason borne' }, options, callback)**  
根据id找到一个doc，并触发mongodb的findAndModify命令，根据[update]进行更新。  
options:  
new: bool。如果true，返回更新过的document。默认false。  
upsert:bool。如果true，更新文档不存在则创建新文档。默认false。  
runValidators：bool。如果true，在update的时候，执行schema上的validator（因为某些caverts，默认false）  
setDefaultsOnInsert：bool。当和upsert同时为true时，在插入新文档时，使用默认值。  
sort：如果找到多个文档（应该不太可能），按照什么顺序选择第一个文档进行update。  
select: 返回哪些字段。 
####删除记录：findByIdAndRemove  
传统方式：查找文档，修改并保存。find/remove。  
推荐方式：**Model.findByIdAndRemove(id, [options], [callback])**。等同于`findOneAndRemove({ _id: id }, ...)`。成功返回被删除的doc。  
options:  
sort: if multiple docs are found by the conditions, sets the sort order to choose which doc to update??  
select: sets the document fields to return??  



