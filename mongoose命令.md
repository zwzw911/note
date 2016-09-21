### connection  
2种方式：**connect**或者**createConnection**，2者参数一样，但是前者创建默认的connection，后者显示的创建一个connection（每个connection是mongoose的一个实例，对应一个db）。  
建议采用**createConnection()**。  

### Promise 
mongoose默认使用mpromise，但是将会在5.0废除。建议使用原生或者第三方Promise。  
使用ES6原生Promise。
`var mongoose=require('mongoose); mongoose.Promise=Promise`

### C(RUD)  
新建记录，使用create或者insertMulti  
**Model.create(doc(s), [callback])**：docs中每个记率转换成new Model(doc).save()执行。返回Promise。  
**Model.insertMany(doc(s), [callback])**: 比create快，因为将所有记录一次发送给mongodb（而不是每个记录执行save())。返回Promise。  

