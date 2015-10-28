#####1. install  
`npm install -g grunt-cli`:是的，首先全局安装grunt-cli，它可以用来启动不同的grunt  

#####2. install grunt
`npm install grunt --save-dev`: 安装grunt，保存信息到packa.json的devDependcy目录，因为grunt只是用来对代码进行处理，而非依赖代码的。  

######3，install grunt-init  
`npm install grunt-init --save-dev`：可选，用来创建Gruntfile.js。如果已经有现成的，直接copy过来修改就可以，不用使用grunt-init产生这个文件了。  
