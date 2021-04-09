
之前我们是基于比较low的方式手工搭建了一个git服务器，然后在上面放了一个git仓库，而且企业级里面一般不会这么干

 

实际上，我们可能会需要在git服务器上用可视化界面的方式去查看所有的提交，分支，以及代码，包括需要在git服务器上去进行分支的合并，而不是在本地去执行分支合并，分支合并之前，我们可能需要先执行code review，可以在git服务器上可以看到本次修改了哪些代码？

 

包括，还有很多其他的事情可以做，然后一些代码的bug缺陷管理和追踪

 

真正在企业里，都是用专用的git服务器，gitlab，其他的一些git管理软件

 

硬件准备，给一台2核4G内存的虚拟机，因此需要使用64位的centos

 

1、Gitlab安装

 

（1）第一步：在系统防火墙上开启允许ssh和http访问

 

yum install -y curl policycoreutils-python openssh-server cronie

lokkit -s http -s ssh

 

（2）第二步：安装postfix来支持gitlab发送邮件

 

yum install -y postfix

service postfix start

chkconfig postfix on

 

（3）第三步：安装gitlab

 

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

EXTERNAL_URL="http://192.168.31.80" yum -y install gitlab-ee

 

此时会自动安装和配置gitlab，同时在指定的url启动gitlab

 

（4）第四步：查看gitlab状态

 

gitlab-ctl status

 

（4）第四步：按照上面指定的url访问gitlab

 

重新设置密码，然后使用root和自己设置的密码来访问gitlab

 

2、使用Gitlab

 

（1）以管理员身份登录进去之后，可以创建项目了，但是创建之前，可以先创建组和用户，然后创建项目的时候，需要将项目选择归属到组或者用户。一般一个组就是一个大的部门，一般选择为private私有的。

 

实践经验

 

比如说，你的公司比较大，有一个团队是基础架构团队，做的都是kv，nosql，缓存，开源项目的二次开发、运维和管理；还有一个团队是做的面向终端用户的，可能做的就是电商业务系统

 

你的不同技术团队，开发的项目是不同的，可以一个group作为一个团队

 

每个Group里面有对应的多个用户

 

（2）进入项目，在settings->members中，还可以手动设置除了组之外的其他人有权限访问这个项目

 

（3）将自己本地生成的ssh key公钥，添加到gitlab中去。使用开发人员自己的账号登录gitlab，然后选择右上角的profile setting -> ssh keys，将自己的ssh key加入进去。

 

（4）将oa项目的代码修改成远程仓库为gitlab上的oa-parent

 

需要在gitlab上，去掉对oa-parent项目的一个master分支的强制保护

 

默认情况下，gitlab做的非常好，就是将master分支给保护住了，不让任何研发人员直接push代码到master分支的，要求必须采用pull request方式，在gitlab上提交merge的请求，通过管理员的code review

 

如果一个成员是master权限，那么他就可以将代码直接push到master，同时只有master权限的可以去review pull request

 

git remote remove origin

重新再次：git remote add origin gitlab项目地址

 

张三和李四的本地仓库都做一样的事情

 

张三，执行：git push -u origin master，再次将本地仓库的代码推送到gitlab上去

 

查查看gitlab上的仓库，就已经有代码了

 

（5）此时试着在张三本地仓库修改代码，推送到远程仓库，然后李四拉取代码，都走通，就ok了

 

正常来说，如果李四就是第一次从gitlab拉取代码下来，那么其实是会自动将本地master分支和origin/master关联在一起的

 

但是我们刚才的情况，是都更改了一下remote origin

 

所以这里要重新手动将本地master分支和远程origin/master分支做一下关联：git branch --set-upstream-to=origin/master master

 

3、维护gitlab

 

启动gitlab：gitlab-ctl start

停止gitlab：gitlab-ctl stop

重启gitlab：gitlab-ctl restart

 

gitlab日志：/var/log/gitlab

 

查看gitlab日志：gitlab-ctl tail

查看gitlab对应的Nginx访问日志：gitlab-ctl tail nginx/gitlab_access.log

查看gitlab对应的数据库postgre-sql的日志：gitlab-ctl tail postgresql

 

gitlab数据存放目录：/var/opt/gitlab/git-data

 

gitlab是一种开源的管理软件，实际上做到这里为止的话，基本上你就可以玩儿起来了

 

授人以鱼不如授人以渔。。。。我不太想给大家讲里面的所有的功能，你可以自己去探索，而且可以自己去看官方文档

 

gitlab使用文档：http://docs.gitlab.com/ce/

 

 

 

 

 

 

 

 

 

773692218