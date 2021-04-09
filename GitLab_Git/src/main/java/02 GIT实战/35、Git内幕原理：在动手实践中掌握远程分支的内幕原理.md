对于远程仓库的分支，本地都是有一个对应的分支是追踪那个远程分支的，同时本地还有一个分支是跟本地追踪分支关联起来的

 

远程仓库：master

 

本地仓库

 

1、origin/master，本地追踪分支，跟远程的master完全是需要对应起来的，远程的master指向哪个commit，每次git fetch origin，就会将远程仓库的提交历史拉取到本地跟本地的提交历史进行合并，同时将本地的origin/master指向远程仓库的master指向的那个commit

 

本地分支（origin/*），追踪了远程的分支（*），每次git fetch之后，提交历史合并之后，本地分支（origin/*）指向的commit都会跟远程分支保持一致

 

不只是origin/master，origin/feature/001，origin/feature/002，origin/release/v1.0

 

2、master，跟origin/master关联起来的

 

本地的master，是跟本地的origin/master关联起来的，作用是在git pull和git push

 

（1）如果执行git pull，会执行远程和本地的提交历史进行合并，origin/master会指向远程仓库master指向的那个commit，同时将origin/master指向的commit跟本地的master指向的commit进行合并，合并之后会出现一个新的commit，同时master会指向那个合并后的commit，然后那个origin/master不变还是指向原来的那个commit

 

git pull，相当于是两个命令：git fetch origin + git merge origin/master

 

相当于就是将远程仓库的提交历史拉取下来，然后合并在一起，本地的origin/master指向最新的commit，接着将本地的master与origin/master进行合并

 

1）将远程仓库的提交历史拉取到本地进行合并，一个commit可能会出来多个分叉

2）同时将origin/master跟远程仓库的master指向一个commit

3）将本地的master和origin/master进行合并，可能需要解决冲突

 

（2）如果执行git push，会将本地的origin/master指向master指向的那个指针，同时会将本地提交历史推送到远程进行合并，然后远程仓库的master分支会指向最新的那个commit

 

每次执行git remote add命令之后，实际上就会在.git/config文件中加入一段配置，类似下面：

 

[remote "origin"]

​    url = git@192.168.31.80:OA/test-project.git

​    fetch = +refs/heads/*:refs/remotes/origin/*

[branch "feature/001"]

​    remote = origin

​    merge = refs/heads/feature/001

[branch "master"]

​    remote = origin

​    merge = refs/heads/master

 

我们之前跟大家说过，如果你在本地想搞一个跟类似origin/feature/001关联起来的本地分支，让git push和git pull有效果，git checkout -b feature/001 origin/feature/001，这段命令执行之后，就会在.git/config中加入[branch]配置，标注了本地分支跟origin/*追踪分支之间的关联关系

 

那么什么是refspce呢？实际上就是fetch后面的那串东西，+refs/heads/*:refs/remotes/origin/*，就是refspce

 

refs/heads/*，代表的是远程仓库被追踪的分支，远程仓库的master

refs/remotes/origin/*，代表的是本地仓库用来追踪远程仓库的分支，本地仓库的origin/master分支

+，代表的是在执行git pull的时候，不是要将远程分支和本地分支进行merge么？即使不是fast-forward的类型，是3-way merge也是要去执行的

 

在执行git remote add命令的时候，本地的git客户端会执行fetch命令获取远程仓库的refs/heads/*下面的分支，然后将其写入本地的refs/remotes/origin/*下面去。所以之前说的那些本地追踪远程的分支，比如origin/master之类的，实际上指的就是refs/remotes/origin/master代表的分支

 

origin/master等等追踪分支，在本地是放在哪儿的呢？

 

我们是可以手动指定多个refspce的

 

根本就不用的下面的语法，介绍一下，refspec指定多个，一般不用

 

[remote "origin"]

  url = https://github.com/schacon/simplegit-progit

  fetch = +refs/heads/master:refs/remotes/origin/master

  fetch = +refs/heads/qa/*:refs/remotes/origin/qa/*

 

[remote "origin"]

  url = https://github.com/schacon/simplegit-progit

  fetch = +refs/heads/*:refs/remotes/origin/*

  push = refs/heads/master:refs/heads/qa/master

 

773692218