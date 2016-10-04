#### POST  
发送{values:{name:{value:aa}}}  
C:\Users\wzhan039>curl -l -H "'Content-type': 'application/json,charset=utf-8','Accept': 'text/plain'" -X POST -d "values"**=**{\"name\":{\"value\":\"af\"}} http://127.0.0.1:3000/billType  
等号表示数据为JSON。  
key用双引号括起（否则无法解析），value如果为字符，也用双引号括起  
