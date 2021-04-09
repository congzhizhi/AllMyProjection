
这块针对的是在本地，如果你刚对工作区的代码进行了修改，或者将部分代码加入了暂存区，或者是提交了一个commit，此时你后悔了，那么如何吃后悔药

 

1、先回顾一下git中的三个概念

 

HEAD就是一个指向当前分支的一个指针

 

index，也就是暂存区，其实就是将会形成下一次提交的文件

 

工作区，其实就是我们的开发区

 

刚开始工作区中会有一些文件

 

然后我们执行git add命令可以将工作区中的文件加入到暂存区中

 

接着我们执行git commit命令，可以将暂存区中的文件一次性提交到git仓库中，同时创建一个commit object指向一个tree object，tree object指向那些文件对应的blob object。

 

然后master指针会指向那个commit object，HEAD指针指向master

 

我们同样可以对文件再次修改，让一个文件变成第二个版本，然后再次git add和git commit，此时暂存里的文件会变成第二个版本，同时也会提交到仓库里创建一个新的commit object

 

新的commit object会引用上一个commit object

 

假设我们又修改了第三次，同时提交了，那么此时会有此三个commit object，同时引用第二个commit object，工作区和暂存区也都是第三个版本

 

2、reset撤回操作

 

（1）执行git reset --soft HEAD^，会有下面的影响

 

1）在仓库中，将master分支指向上一个commit

2）暂存区和工作区还是停留在第三个版本，没有变化

 

因此这个命令，相当于是撤回了git commit操作，因为git commit操作，就是将暂存区中的第三个版本放入仓库，同时创建一个新的commit object指向第三个版本，master指针也指向最新的commit object

 

此时就相当于取消了刚执行完的git commit操作

 

我们可以再次对暂存区加入更多的内容，然后再次执行git commit操作，此时再看一下效果，你会发现跟git commit --amend类似

 

（2）执行git reset HEAD^，默认等同于git reset --mixed HEAD^

 

回到三个区域都是第三个版本的状态，执行git reset HEAD^

 

1）将master指针指向上一个commit

2）将master指针指向的commit对应的版本恢复到暂存区中

 

此时暂存区中的文件版本会变成第二个版本

 

这个命令，相当于是撤回了git commit和git add操作

 

这里我们可以想象一下，如果用git reset HEAD，就可以实现将刚git add到暂存区中的文件版本回退到仓库中的上一个版本，相当于就是回退了git add操作

 

（3）执行git reset --hard HEAD^

 

相当于是彻底撤回刚才的git commit，git add，以及工作区中的代码修改

 

因为这个命令，会将指针指向上一个commit，同时暂存区里的内容变成上一个commit的版本，同时将上一个commit的内容恢复到工作区

 

其实这个就是之前讲解过的，相当于是回退到历史上某一个版本了

 

（4）梳理一下

 

1）如果刚刚修改了工作区中的内容，但是没放入暂存区和仓库，此时要抛弃这些修改，怎么办？

 

这种情况是很常见的，一般见于什么呢？临时修改技术方案。。。。

 

比如说，你现在要开发一个工作流模块，然后之前定的技术方案是基于jBPM 4.x去开发，然后呢，你就开始吭哧吭哧吭哧开始写代码，引入了jBPM 4.x的jar包

 

写了几天，你的leader，架构师，反悔了，他说，我前两天没设计好，考虑不周到，要不换成用Activit去开发吧。。。

 

你做了几天，发现jBPM 4.x的功能不能满足你的需要，临时换框架

 

也可能是，做了几天以后，发现其他团队已经有一个现成的基础类库，可以直接基于他们的来封装

 

如果说此时你修改的代码比较少的话，那么就用git checkout -- 文件名，就可以将这个文件的内容恢复到上一次提交的状态，完全抹掉你最近的修改

 

但是如果你已经修改了很多代码了，此时你需要来个一次性的，强有力的恢复：git reset --hard HEAD

 

git reset --hard HEAD 或者是 git checkout -- 文件名

 

意思就是保持仓库和暂存区里的文件版本为HEAD最新版本，同时将上一次提交的HEAD版本写回工作区中，覆盖掉这次修改。

 

2）如果修改了工作区中的内容，同时放入了暂存区，但是还没提交，此时要抛弃这些修改，怎么办？

 

（1）你希望将暂存区里加入进去的内容撤销掉，然后在工作区里面重新修改代码

 

git reset HEAD = git reset --mixed HEAD，一样的命令，就是将仓库中HEAD指向的上一个commit，恢复到暂存区，同时恢复到工作区

 

（2）你希望将暂存区和工作区里面的内容一次性抹掉：git reset --hard HEAD

 

3）如果修改了工作区中的内容，放入了暂存区，还提交了，怎么办？

 

本来今天的代码你都提交了，但是你的leader说，这块代码不要了，此时怎么办？回退版本，直接回退到没做任何代码修改的那个版本

 

git reset --hard HEAD^，git reset --hard commit标识符

 

git reset --hard HEAD^，分支指针指向上一个commit，同时上一个commit的内容恢复到暂存区，也恢复到工作区

 

git reset --hard v1_commit_标识符

 

git reset --hard HEAD^^

 

（4）总结

 

上面说的，同样都是针对本地的commit，一旦你已经push到了远程仓库，就不能这样了，否则会弄乱你的提交历史

 

 

 

 

 

 

 

 

 

 

 

 

 

773692218