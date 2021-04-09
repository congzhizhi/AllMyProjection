
1、在出现冲突的时候773692218

 

如果两个分支merge出现了冲突，临时打算取消注意，不想merge了，那么此时可以执行git merge --abort命令即可

 

2、如果仅仅是在本地完成了merge后想要撤回

 

很简单，直接执行git reset --hard HEAD^，将分支挪回merge之前的那个commit即可，就相当于没执行过merge

 

3、如果是在推送了merge后的commit到远程仓库之后想要撤回

 

我给大家说一个非常经典的实际的案例，工作中之前经常遇到的

 

比如说，我们之前讲过，按照GitFlow工作流来走，最后是在release分支进行QA测试，测试通过之后，会合并到master分支，准备进行预发布测试和上线

 

但是。。。。。

 

不靠谱的队友，不靠谱的pm

 

不靠谱的队友，你是跟其他团队协作一起上线的，此时你的仓库的代码都合并到了master，确认没问题了，服务A，依赖于其他团队开发的服务B，在不同的仓库里的两个项目；但是。。。你要一起上线的其他团队的同学，服务B，突然发现在QA环境，临时测试出来了一个大bug，此时pm沟通了一下，评估了一下风险，决定，今天取消上线，下周再上

 

尴尬了

 

你的代码都合并到master分支了。。。但是说不上线了，此时你是不能将不准备上线的代码留在master分支里的

 

如果你有其他需求在最近一两天也要上线，也会合并到master分支，然后如果那个需求上线，master分支会把你的那个不准备上线的功能的代码，给带上线

 

所以这种情况，是不能接受的

 

常见于大公司，上百人协作的那种大项目，比如某巨头公司的广告系统，最核心的系统，牵扯到上百人，数百人一起在搞，架构升级

 

如果碰到这样的情况，你是需要将本次之心的merge给撤回的，无论是本地还是远程，都要撤回

 

但是如果push到了远程仓库以后，你是不能再本地reset一下，再push的，因为那样会搞乱你的提交历史的

 

 

不上线了。。。撤回release/v1.2.0到master分支的合并

 

需要在本地执行一个命令：git revert -m 1 HEAD

 

git revert实现的效果，不是说指针往回挪动，而是说创建一个新的commit，对应的代码版本跟合并之前的那个commit是一模一样的

 

-m 1的意思就是，恢复到当前这个分支合并之前的那个状态，此时会创建一个新的commit，这个新的commit对应的版本跟合并之前的commit是一模一样的

 

这就完美撤回了merge操作，同时此时如果执行git push origin master，会让远程仓库的master分支也回退到merge之前的状态，但是也多了一个新的commit出来

 

但是此时有一个问题，就是如果再次执行git merge feature/001，你会发现，git不让你merge了，因为之前已经merge过了，很尴尬

 

这个时候，过了几天了，然后呢，已经准备好可以上线了，你需要重新将release分支的代码合并到master分支上来，此时该怎么办呢？

 

而且此时一般是说feature分支需要继续修改代码，比如又加入了一些代码，但是如果这个时候你胡乱再次合并，会发现master分支仅仅merge进去了feature分支后来修改的那一点代码而已。。。。

 

此时，要做的事情是，继续对feature分支进行修改，完成最终版本的代码

 

然后，再次执行git revert HEAD，将master分支再次进行revert，回退到之前merge过后的那个状态，这个状态是包含了feature分支merge进来的部分代码的，然后再次执行git merge feature/001，将feature分支最新的修改也merge进来

 

ok了，完美解决，再次git push origin master，推送到远程分支去

 

4、结合gitlab实战一下分支合并的撤回操作

 

梳理一下

 

（1）如果是要撤回已经push到远程仓库的merge操作

 

（2）在本地执行git revert -m 1 HEAD，再执行git push origin master，此时本地和远程的提交历史都会多一个commit出来，该commit的内容和合并之前的master指向的那个commit是一样的，同时master此时指向最新的那个commit

 

（3）但是后面，如果要再次将release分支和master分支进行合并，此时是需要特殊处理的，再次在本地执行：git revert HEAD，git push origin master。就是再次将master还原到指向一个新的commit，该commit的内容与上一次merge后的那个commit一样，包含merge的内容

 

（4）最后再次将release分支与master分支进行合并，此时可以保证release分支所有的内容，都是合并到master分支的

 

5、接着上一讲，如果在本地做了commit，同时还推送到了远程仓库，此时要撤回，怎么办？

 

简单了，比撤回merge简单多了，直接git revert HEAD，不就回到了上一个commit的状态，然后再次git push origin master，完美同步撤回远程仓库的commit操作了

 

 

 

 

 

 

 

 

 

 