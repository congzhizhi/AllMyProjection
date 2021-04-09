1、建议git命令行

 

首先是尽量全部在git命令行模式下完成，命令行模式玩儿熟了，才能触及git的本质

 

到了后面，当然在企业中开发的时候，那是要用eclipse+git整合，git gui可视化工具，gitlab开源的git服务器去使用

 

git命令行是最核心的工具，可以执行git所有的功能，而且可以触及git的本质

 

而大多数的git gui工具，只能执行一部分的git命令和功能，而且学会了git命令行之后，对于git gui工具的使用就是轻而易举，因为各种功能背后的命令和原理你都熟悉了

 

因此建议学习从git命令行开始，然后实际工作中根据自己的个人喜好选择一种git gui工具使用即可，或者在工作中也可纯碎使用git命令行，感觉很酷

 

2、安装git

 

直接下载git的windows安装包，一步一步傻瓜式安装即可

 

git官网：https://git-scm.com/

 

安装完了之后，就可以得到两个工具，git gui和git bash，git gui就是给你图形化的方式来使用git，但是不推荐现在使用；git bash就是用模拟linux的方式让你在命令行使用git

 

打开git-bash，进入一个模拟了linux环境的命令行的界面，然后通过这里就可以去操作git，同时也可以认为git-bash是一个windows上的linux命令行模拟器，因为linux上的大部分命令都可以执行，包括ssh

 

git --version，检查一下git的版本号

 

3、配置git

 

git有一个命令，git config，专门用来对你的git环境进行各种配置。git有三个地方放环境配置：

 

之前正好都学习过了maven，在本地安装了maven之后，maven是有自己的一个配置文件，settings.xml，然后每个maven管理的项目中又有一个这个项目的特定的maven配置文件，pom.xml

 

git是一样的，全局情况下，也是有git自己的配置文件的

 

（1）/etc/gitconfig，对当前机器上所有的用户和git项目都生效，使用git config --system，即可对这个配置文件进行操作，也不是很重要，因为我们一般很少去配置这个文件，因为git的配置一般都是对某个用户和项目的范围去生效的

 

（2）~/.gitconfig，当前用户主目录的.gitconfig文件，对当前用户有效，使用git config --global，即可对这个配置文件进行操作，windows的话，是在这里：/c/Documents and Settings/All Users/Application Data/Git

 

（3）项目的.git目录下的config文件，仅仅对当前git管理的这个项目有效，直接git config操作的就是这个文件

 

因为我们其实一般都不会直接修改git的配置文件来进行配置，一般都是执行git config命令，来修改我们的各个不同的范围的配置，比如全局的配置，或者是项目特殊的配置

 

配置用户名和邮箱，安装好git之后第一个事情，就是配置自己的用户名和邮箱，后面每次你提交代码的时候，都会带上你的个人信息

 

git config --global user.name "zhss"

git config --global user.email "zhss@163.com"

 

git config --global的意思，就是设置当前用户范围内的配置，对机器上的其他用户是无效的

git config --system，就是对当前机器上所有用户都生效

git config，就是对当前所在的git项目本身生效

 

接着可以配置默认的文本编辑器，比如输入提交代码时的备注信息时，可能会打开一个文本编辑器让你输入，默认就是用操作系统自带的编辑器。不过这个说实话，一般用操作系统自带的就好了，除非你自己喜欢用别的。

 

git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -nosession"

 

查看所有的配置项：git config --list

查看某个配置项：git config user.name

 

4、帮助文档

如果对某个git命令不太熟悉，可以用git的帮助文档

 git add --help

-----





配置用户名和邮箱，安装好git之后第一个事情，就是配置自己的用户名和邮箱，后面每次你提交代码的时候，都会带上你的个人信息

git config --global user.name "zhss"

git config --global user.email "zhss@163.com"

git config --global的意思，就是设置当前用户范围内的配置，对机器上的其他用户是无效的

git config --system，就是对当前机器上所有用户都生效

git config，就是对当前所在的git项目本身生效

