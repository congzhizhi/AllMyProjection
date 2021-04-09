centos

第一步，在centos上安装git

 

git --version

yum install -y git

 

1.7版本，还好

 

第二步，在centos上创建一个git用户来运行git服务：

 

groupadd git

adduser git -g git

 

第三步，每个工程师生成自己的ssh key公钥并放入git服务器

 

每个工程师都需要通过ssh-keygen -t rsa，然后在~/.ssh目录下可以找到id_rsa.pub文件，就是公钥，把公钥导入到git服务器上的/home/git/.ssh/authorized_keys文件里，一行一个。

 

第四步，初始化Git仓库：

 

使用/srv/oa-parent.git目录作为远程仓库，在/srv目录下输入命令：

 

git init --bare oa-parent.git

 

Git会创建一个裸仓库，裸仓库没有工作区。因为服务器上的Git仓库就是为了共享，不会让用户直接登录到服务器上去修改工作区的，而且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：

 

chown -R git:git oa-parent.git

 

第五步，禁用shell登录：

 

一般出于安全考虑，第二步创建的git用户是不允许使用shell的，我们可以编辑/etc/passwd文件。

 

找到类似下面的一行：

git:x:1001:1001:,,,:/home/git:/bin/bash

 

改为：

git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell

 

这样，git用户可以正常通过ssh使用git推送和下载代码，但是无法通过shell登录的，因为我们为git用户指定的git-shell是每次一登录就自动退出。

 

第六步，基于git服务器和远程仓库上传和下载代码：

 

git remote add origin ssh://git@192.168.31.244:/srv/oa-parent.git

 

将本地仓库和git服务器上的远程仓库关联起来

 

git push -u origin master

 

![img](https://tva1.sinaimg.cn/large/008eGmZEly1gop2gxf293j31n40igqhr.jpg) 

 

-u，将本地仓库的master分支和远程仓库的master分支关联起来

 

git clone ssh://git@192.168.31.244:/srv/oa-parent.git

 

（1）搭建起来了一个git服务器

（2）在git服务器上创建了一个远程仓库，bare裸仓库

（3）将本地的oa项目代码推送到了远程仓库

（4）在本地另外一个目录，模拟成另外一个研发人员，在自己本地将远程仓库中的oa项目代码克隆到了本地

773692218