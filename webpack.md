##### 1. 4个组成部分：  
1.1 **entry**：入口点，告知需要打包的文件，以及这些文件之间的依赖关系  
1.2 **output**：打包的文件放置的位置  
1.3 **loader**：webpack默认只能处理js，为了能处理其他类型的文件，使用loader进行转换  
1.4 **plugin**: loader只是对单个文件进行处理，为了对多个文件进行处理（例如合并，打包），用plugin  

##### 2. entry  
entry: string|Array\<string\>  
*entry: './path/to/my/entry/file.js'* 实际被转换成 *entry: { **main**: './path/to/my/entry/file.js' }* ，如果输入的是array，会被转换成multi main entry，这对加入多个依赖文件，并把它们打成一个chunk是很有用。  
