1. 设置淘宝镜像    
`npm install -g cnpm --registry=https://registry.npm.taobao.org`    
2. webpack不要安装4.29.6，有问题，暂时使用稍旧的版本4.29.0
   webpack和webpack-cli全局安装，这样可以直接运行webpack。否则运行` npx webpack `    
 `npm install -g webpack@4.29.0 webpack-cli`  
3. 安装babel-loader    
`npm i -D babel-loader @babel/core @babel/preset-env`  
   



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
2. **filename**：entry编译后的文件名。 如果有多个entry，可以使用**[name],[hash],[chunkhash]**来定义  
  output: {
    filename: '[name].js',
    path: __dirname + '/build'
  }
除此还有其他属性：  
1. **chunkFilename**: **非entry**的*output文件相对于path定义的路径*
1.1 **[id]**:chunk id  
1.2 **[name]**: chunk name  
1.3 **[hahs]**: 编译后的hash  
1.4 **[chunkhash]**: chunk的hash  
2. **devtoolLineToLine**: 源文件和编译文件的sourcemap。默认true，对所有文件生效，可以采用module.loader类似的格式{test:,include,exclude}  
3. **hotUpdateChunkFilename**：**[id], [hash]**  
4. **hotUpdateFunction**：异步加载hot update的JSONP函数  
5. ****  
  
### 4. loader  
loader是作用在源文件上的一个函数，返回另外一个resource。  
module:{
 rules:[
   {test:/\.css/, **use**:'css-loader'} == {test:/\.css/, **loader**:'css-loader'} =={test: /\.css$/, **use**: {**loader**: 'css-loader',options: {}}
 ]
}  
loader可以通过3中方式定义：  
1. webpack.config.js（**推荐使用**）  
rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader'},
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
2. require('style-loader!css-loader?modules!./styles.css');通过!作为分隔符，通过？带入option  
3. CLI
loader遵循module resolution，通常是node_modules


### plugin  
plugin是webpack的骨干，是一个带有apply属性的对象。  因为使用plugin要带入option，所以一般都要使用new来实例化。  

### 配置文件  
配置文件是一个export object的JS文件，所以可以使用各种Node.js的方法、函数等  

resolver是一个lib，用来定位到module的绝对路径  
1. 绝对路径: *import "/home/me/file";*  因为已经知道文件路径，无需resolve  
2. 相对路径：require或者import的文件路径，需要和context的路径结合，获得绝对路径  
3. 模块路径： 在**resolve.modules定义的路径**中搜索文件（可以通过resolve.alias更改此路径）。如果resolve后，import或者require的是一个文件，那么：如果有**扩展名**，直接大包；否则，通过resolve.extensions来判断哪种扩展名是webpack可以处理的。  如果是目录，检查有无package.json
