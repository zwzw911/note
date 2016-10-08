### connection  
2种方式：**connect**或者**createConnection**，2者参数一样，但是前者创建默认的connection，后者显示的创建一个connection（每个connection是mongoose的一个实例，对应一个db）。  
建议采用**createConnection()**。  

### Promise 
mongoose默认使用mpromise，但是将会在5.0废除。建议使用原生或者第三方Promise。  
使用ES6原生Promise。
`var mongoose=require('mongoose); mongoose.Promise=Promise`

### C(RUD)  
####新建记录，使用create或者insertMulti  
**Model.create(doc(s), [callback])**：docs中每个记率转换成new Model(doc).save()执行。返回Promise。  
**Model.insertMany(doc(s), [callback])**: 参数docs为**数组**。比create快，因为将所有记录一次发送给mongodb（而不是每个记录执行save())。返回Promise。  
####更新记录：findByIdAndUpdate  
**Model.findByIdAndUpdate(id, { name: 'jason borne' }, options, callback)**
根据id找到一个doc，并触发mongodb的findAndModify命令，根据[update]进行更新。
options:
new: bool。如果true，返回更新过的document。默认false。
upsert:bool。如果true，更新文档不存在则创建新文档。默认false。
runValidators：bool。如果true，在update的时候，执行schema上的validator（因为某些caverts，默认false）
setDefaultsOnInsert：bool。当和upsert同时为true时，在插入新文档时，使用默认值。
sort：如果找到多个文档（应该不太可能），按照什么顺序选择第一个文档进行update。
select: 返回哪些字段。 


