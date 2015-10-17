###node安装可以通过两种方法:  
1. 源代码安装  
2. 二进制安装  

源代码安装可以定制：比如决定安装目录，但是需要编译；二进制安装直接解压即可，但是需要配置，来直接运行  

####源代码安装：  
1. 首先检查是否安装了python2.7/gcc/gcc-c++
    yum install python gcc gcc-c++ 
    
    (1/2): python-2.7.5-18.el7_1.1.x86_64.rpm                                                                                                                                                                             |  86 kB  00:00:00     
    (2/2): python-libs-2.7.5-18.el7_1.1.x86_64.rpm   
    
    Installing : mpfr-3.1.1-4.el7.x86_64                                                                                                                                                                                                   1/7 
    Installing : libmpc-1.0.1-3.el7.x86_64                                                                                                                                                                                                 2/7 
    Installing : cpp-4.8.3-9.el7.x86_64                                                                                                                                                                                                    3/7 
    Installing : kernel-headers-3.10.0-229.14.1.el7.x86_64                                                                                                                                                                                 4/7 
    Installing : glibc-headers-2.17-78.el7.x86_64                                                                                                                                                                                          5/7 
    Installing : glibc-devel-2.17-78.el7.x86_64                                                                                                                                                                                            6/7 
  
  
    Installing : libstdc++-devel-4.8.3-9.el7.x86_64                                                                                                                                                                                        1/2 
    Installing : gcc-c++-4.8.3-9.el7.x86_64  

2.  下载  
    `wget https://nodejs.org/dist/v4.2.1/node-v4.2.1.tar.gz`    
3. `./configure --prefix=/usr/local`, 这样，编译后的文件都会自动放入到/usr/local中对应的目录，例如，node/npm会放到/usr/local/bin下。如此，node和npm就可以在任意地方直接运行了  

####二进制安装：
1. 下载：  
    `wget https://nodejs.org/dist/v4.2.1/node-v4.2.1-linux-x64.tar.gz`
2. 解压：  
    `tar zxf node-v4.2.1-linux-x64.tar.gz`  
3. 生成一个目录，内含完整的结构：其下包括bin,share,lib等目录
4. 为了能在全局使用，可以：  
    4.1  **把node-v4.2.1下所有的目录copy到/usr/local对应的目录下**，例如：把node-v4.2.1/bin下的node/npm copy到/usr/local/bin，node-v4.2.1/lib=>/usr/local/lib，node-v4.2.1/share=>/usr/local/share。  
          因为node会依照原始结构来查找，例如，运行npm，会在../lib查找对应的文件    
    4.2   在PATH中添加node-v4.2.1/bin，这样，目录结构不用改变，可以直接运行bin下的文件了  
    

    

