#####1. install  
`npm install -g grunt-cli`:是的，首先全局安装grunt-cli，它可以用来启动不同的grunt  

#####2. install grunt
`npm install grunt --save-dev`: 安装grunt，保存信息到packa.json的devDependcy目录，因为grunt只是用来对代码进行处理，而非依赖代码的。  

#####3. install grunt-init  
`npm install grunt-init --save-dev`：可选，用来创建Gruntfile.js。如果已经有现成的，直接copy过来修改就可以，不用使用grunt-init产生这个文件了。  

#####4. install各种grunt插件  
"grunt-contrib-clean": 删除文件  
"grunt-contrib-concat": 合并文件  
"grunt-contrib-copy": 复制文件  
"grunt-contrib-cssmin": 缩小css文件  
"grunt-contrib-htmlmin": 缩小html文件  
"grunt-contrib-jshint": 检查js语法  
"grunt-contrib-uglify": 压缩js文件  
"ngmin": 设置angular文件格式，以便可以用uglify压缩  
