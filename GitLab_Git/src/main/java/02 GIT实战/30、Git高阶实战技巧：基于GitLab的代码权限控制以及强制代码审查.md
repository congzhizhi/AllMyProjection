Git高阶实战技巧这一部分的最后一讲，在实际生产环境中，各种高阶的实战技巧773692218

 

代码权限这回事儿

 

实际上，如果没有gitlab，你会发现你做代码权限的控制很不方便

 

但是，有了gitlab之后，你会发现很方便

 

在实际企业中，结合gitlab，代码权限控制

 

（1）第一件事情，有些分支是需要被保护起来的，就是说不能直接被push，尤其是master分支。

 

在settings->repository，有一个protected branch，就是master分支默认是被保护起来的，只有这个项目的master角色的开发人员，才能直接对master分支进行push操作

 

绝对不允许任何普通的研发人员随意就可以往master分支上push代码

 

也绝对不允许普通的研发人员，可以有权利把merge request合并到master分支上去

 

只有拥有master权限的用户才可以s

 

我们实践中是怎么弄的，一个仓库对应着一个项目，一个项目总有一个leader，项目负责人，高工，架构师

 

总之，一遍就是这个项目里的高工和架构师才有master的权限，可以push代码到master，还有将merge request合并到master去

 

（2）仓库全都是private私有的，如果在一个部门中建立了一个项目，那么默认这个部门下的研发人员对这个项目都是有权限的，但是有的人是master，有的人是developer

 

developer是只有权限对普通的分支进行各种操作的，feature，develop，release

 

但是最后developer是没有权限直接push代码到master分支的

 

而且，developer最后是需要提交merge request，将release分支合并到master分支的，此时只有master权限的人才可以去review merge request，将其他分支merge到master分支

 

其他团队的人，其他group的人，默认是没有权限来访问我们部门的项目的

 

每个部门的人，默认就只能访问自己部门的项目和代码

 

每个项目的话呢，又只有master负责人有权限操作master分支

 

其他团队的人可以作为guest来访问我们团队的项目，因为经常会遇到什么呢？有其他团队的人说，让我看一下你们的代码行不行啊，那此时可以啊，给一个reporter权限就可以了

 

guest，你可以给一些大领导，有些大领导是不需要修改代码的，可能也不看代码，然后就是来看看你的项目里的一些issue，gitlab管理一些缺陷，大领导，可能会关注一些bug的情况，会进来看看bug相关的一些issue

 

（3）实践过的一个技巧：强制code review

 

强制code review，每个工程师在feature分支中开发好和单元测试好之后，他们本来是需要将feature合并到develop去做集成的

 

其实这个环节是需要进行code review，要求他们必须提交merge request到develop分支，让项目里的master权限的高工去review他们的代码

 

但是如果默认情况下，没有将develop分支设置为protected，那么普通人是可以绕过高工，直接在本地跟develop分支进行合并，然后push到远程的develop分支上去的

 

如果将develop分支保护起来之后，就是说，每个研发人员第一次写好feature分支的代码，都必须通过merge request往develop分支合并代码，此时项目里的高工会负责review他们的代码

 

review通过之后，可以合并到develop分支，做持续集成

 

后续是要在develop分支做持续集成测试的，可能需要修改bug，修复集成测试的bug，还是在feature分支上进行。每次修复好一批bug之后，push到feature分支，然后提交merge request合并到develop分支，高工继续review代码。

 

就是说卡在持续集成这个环节，将代码质量给严格把关，每次提交的代码，都必须高工过目，看一眼，不能写的太烂

 

通过develop持续集成之后，会从develop拉那个release分支，接下来，每个研发人员就可以在release分支直接进行bug的修复了

 

 

 

 

 

 

 

 

 

 

 

 

 

 