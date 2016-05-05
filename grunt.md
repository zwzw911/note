#####1. install  
`npm install -g grunt-cli`:首先**全局**安装grunt-cli，它会根据进入的不同grunt目录，自动启动对应的grunt。如此，就可以安装多个grunt版本，并各自独立的运行了。  

#####2. install grunt
`npm install grunt --save-dev`: 安装grunt，保存信息到packa.json的**devDependcy**目录，因为grunt只是用来对代码进行处理，而非依赖项目的（即部署时，grunt无需部署）。   

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
"grunt-ng-annotate": 重构angular依赖，以便可以进行压缩  

#####5. 如果要对当前目录下的子目录和文件进行操作。  
必需使用pwd/\*\*/\*.ejs，而不是pwd/\*\*.ejs

#####6. 采用cwd  
首先从a目录处理文件到目录dist_tmp，结构变成dist_tmp/a；然后从dist_tmp/a处理，变成dist/a。  
在第二步中，需要使用**cwd**。如此，会忽略cwd指定的目录，直接处理cwd下的目录到src中
