### 1. 4个组成部分：  
1.1 **entry**：入口点，告知需要打包的文件，以及这些文件之间的依赖关系  
1.2 **output**：打包的文件放置的位置  
1.3 **loader**：webpack默认只能处理js，为了能处理其他类型的文件，使用loader进行转换  
1.4 **plugin**: loader只是对单个文件进行处理，为了对多个文件进行处理（例如合并，打包），用plugin  

### 2. entry  
1. **single entry**：entry: string|Array\<string\>    
*entry: './path/to/my/entry/file.js'* 实际被转换成 *entry: { **main**: './path/to/my/entry/file.js' }* ，如果输入的是array，会被转换成multi main entry，这对加入多个依赖文件，并把它们打成一个chunk是很有用。  
2. **object entry**:  entry: {[entryChunkName: string]: string|Array\<string\>}  
 entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }  
  这是最灵活的配置方式：一般用于单页app，告诉webpack，**同时**在app和vendor中创建依赖图，且这些依赖图是完全独立且不互不关联的（每个bundle中都有独立的webpack bootstrap）。这种方式可以利用CommonsChunkPlugin，将第三方的库打包在一起，然后在浏览器端实现永久缓存。  
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }  
  一般用于多页app  
  
### 3. output  
output告诉webpack如何将编译后的文件输出到磁盘上。**entry可以定义多个，但是putput只能定义一个**，如果使用[hash]或者[chunk_hash]来定义output，使用 **OccurrenceOrderPlugin or recordsPath**定义模块的顺序。  
output最起码要定义2个属性：  
1. **path**：绝对路径  
2. **filename**：编译后的文件名。  
除此还有其他属性：  
1. **chunkFilename**: 
1.1 **[id]**:
  
