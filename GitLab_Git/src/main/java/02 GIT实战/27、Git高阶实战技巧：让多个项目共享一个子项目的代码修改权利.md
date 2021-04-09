
如果按照我自己的意思，我都不想讲，涉及到的知识点，是submodule和subtree，这两个东西，很多人，搞技术的，从技术出发，git里真正高深的东西是rebase和submodule773692218

 

实际上，我是很不认可这个东西，你要从实际工作场景触发，从实战出发，从需求出发

 

实际上我在这个Git高阶实战技巧这一大块里面讲解的内容，90%以上都是我工作中真正在企业场景中遇到过的，真正实用，有用的东西，结合工作场景给大家去讲解

 

简单来说，就是如果两个项目在两个仓库中，要共享一个子项目的代码，两个项目的人都可以去修改他，该怎么办呢？

 

但是实际上，在实际工作场景中，我坚决不建议出现这样的情况，大家就是如果有某一个子项目的话，单独交给某个人，然后单独拉一个仓库就可以了

 

1、Submodules

 

一般来说，submodule使用的场景，就是要在一个父项目下，再建立一个子项目。

 

（1）我们先建立一个DbConnector子项目

 

git clone git@192.168.31.80:OA/DbConnector.git

cd DbConnector

touch README.md

git add README.md

git commit -m "add README"

git push -u origin master

 

（2）将所有工程师的父项目中都注册DbConnector这个子项目

 

进入某一个父项目，然后在其下创建一个submodule：

 

git submodule add git@192.168.31.80:OA/DbConnector.git

 

这个时候，会出现一个新的目录，就是DbConnector。而且此时用git status命令，会发现说有.gitmodules和DbConnector两个东西等待提交。

 

.gitmodules里面保存了你的子模块和其对应的git url之间的映射关系

 

然后执行下面的命令，将子模块提交：

 

git add --all .

git commit -m ‘added DbConnector module’

git push origin master

 

接着别的研发人员，可以将这个包含了一个子模块的项目克隆到自己本地，但是此时默认是只有一个子模块的目录，里面是空的，没有文件的。

 

git clone git@192.168.31.80:OA/oa-parent.git

 

或者是

 

git pull

 

此时需要运行两个命令：

 

git submodule init

git submodule update

 

才能初始化子模块的目录，同时拉取子模块的文件

 

或者也可以简化一点，通过给下面的命令，一次性把上述工作都干了：

 

git clone --recursive git@192.168.31.80:OA/oa-parent.git

 

（3）对子项目进行代码修改

 

1）无论谁都可以跟平时一样，在DbConnector里面，当做是一个完全独立的项目，在里面可以修改源代码，包括说修改之后提交，分支操作，在DbConnector里面去执行就可以了

 

2）以后每次如果某个人要更新子模块中的数据，那么执行以下命令即可：

 

进入子模块的目录

git fetch

git merge origin/master

 

2、Subtree

 

git以前是推荐使用submodule来管理子项目，但是git 1.5.2之后就推荐使用subtree了。subtree的优点，就是子项目的维护对我们来说几乎就是透明的，不需要引入太多的维护成本。

 

cd P1项目的路径

git subtree add --prefix=用来放S项目的相对路径 S项目git地址 xxx分支

 

上面的代码，就会把s项目的代码（指定分支）下载到本地放入指定的目录中，同时提交到本地p1项目中

 

对于P2项目也做一样的事情

 

直接在p1中各种修改代码和提交，包括s项目的代码，都没有关系

 

接着某一天，p1项目负责人想把对s项目的代码更改提交到远程仓库，此时执行下面的命令：

 

git subtree push --prefix=S项目的路径 S项目git地址 xxx分支

这个时候，git会找到所有对s目录的commit，然后一起提交到其对应的远程版本库中

 

接着p2项目的负责人知道了这个事情，需要将s项目最新的代码更新到自己本地，此时就执行下面的命令即可：

 

git subtree pull --prefix=S项目的路径 S项目git地址 xxx分支

 

3、最后的一个总结

 

如果有多个仓库，多个项目，要共享对同一个子项目的修改权限的话，那么可以这么玩儿

 

子项目其实是有一个单独的仓库的

 

但是在父项目中可以为其注册子项目，接着不同仓库的人，就可以很方便的在自己的仓库代码中，修改子项目的代码，同时互相之间都可以push代码，也可以pull到别人push的代码

 

就算真的有类似的场景，我也不推荐使用，太复杂了

 

 

 

 

 

 

 