###安装windows版本git客户端
https://git-for-windows.github.io  

###创建key
为了能和github创建联系，需要在本机上创建ssh密钥，并把**公钥**添加到github中 
在git客户端，执行`ssh-keygen -t rsa -C "youremail@example.com"`，无需passphrase，一路回车，结果会显示公钥和私钥创建的位置，打开公钥文件（.pub文件），并把内容粘帖到github中即可（github->setting->ssh and GPG keys->New ssh Key）  
`git clone git@github.com:zwzw911/ss-express.git`来下载代码  

###设置git  
`git config --global user.email "zwzw911@hotmail.com"`  
`git config --global user.name "zwzw911"`   


###使用git  
check in
1. `git add -A`
2. `git commit -m 'test'`
3. `git push origin master`

###初始化
1. 在对应的项目先执行`git init`  
2. 在github上创建一个仓库，例如finance.git（.git是自动添加的）
3. 在项目下执行`git add -A`和`git commit -m "any"`（将要上传的内容commit）
4. `git add remote origin git@github.com:zwzw911/finance`，将本地仓库和远程仓库的origin分支联系起来
5. `git push -u origin master`。将本地的master分支推送到远程的origin分支。
