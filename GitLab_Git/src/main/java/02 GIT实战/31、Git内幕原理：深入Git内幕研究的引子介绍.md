从这一讲往后，我们会不少讲，是进入这个git课程的最后一块内容，就是git内幕原理773692218

 

简单带一句，我现在的一个授课思想，刚开始学习一个技术，必须先短平快引入，入门要快，把这个技术最核心需要用到的一些东西，用一点简单的demo带一些，快速的就入门了

 

在入门的同时，用的是那个技术最核心的一些东西，直接把这个技术最最根本，最最基础的原理，给大家阐明

 

刚有个感觉，体验，功能背后的一套比较根本的原理是那样子的，三个区域

 

就是纯实战驱动，这个技术在实际企业中是怎么来运用的，如果是类似maven，git这种技术，是没法做项目实战的，融入在我们后面的大项目里的，贯穿了整个做项目的环节

 

但是我们可以把企业中常见的一个一个的实战场景，应用场景，反过来推出，我现在有这么一个企业里的而应用场景，需求，问题，然后呢，我的git技术有哪个命令或者说是功能，可以解决他？？

 

每个技术讲解完之后，每个git的命令，如果有可能，再给大家带一带，深入剖析一下命令背后的原理。功能分支工作流实战切入，分支那一套的内幕原理，后面的一系列的命令

 

实战驱动，深入原理

 

收尾的那一段，精华，比如说git，其实它是有自己的一套内幕原理的，非常深入的去探索一下git内幕里的原理，把深入的最后一块东西给讲好，整个这个技术学习，完美

 

（1）学习一些开源技术背后的原理和思想

 

（2）加深你的技术内功，会用一大堆技术有什么用？但是如果你对很多技术背后的内幕原理和整体上的架构设计思想，都很精通，那么你的技术内功会很强悍 => 你在后面做项目的时候，设计，解决问题，得心应手 => 内功越深厚，你以后在学习类似技术，或者其他技术的时候，速度就会更快

 

（3)直接能够提升你的以后做自己项目时候的技术能力 => 吸收了很多开源技术的设计思想，架构，方案，机制 => 运用到了你自己的项目里，你会发现很多问题和场景，都可以吸取他们的思想去解决

 

（4）如果你的很多技术内功很深厚的，就业，职场发展 => 99个程序员的简历上写的都是熟悉spring boot，spring cloud等技术的使用，有过什么什么经验 => 深入阅读过spring boot和spring cloud的源码，对其架构体系和设计原理较为精通 => BAT，简历上的亮点 => 面试的时候，如果你能把这些深入的东西讲出来，架构师，技术专家，高级工程师，BAT，中小型公司，都会有很大的优势

 

一张思维导图带你梳理清楚学习到的技术体系，maven课程，反馈都很好

 

授人以鱼不如授人以渔：学习方法，如何去自己去未来更加深入的研究和学习相关的技术，自己如何能够走得越来越远

 

包括我之前讲的很多讲，Git高阶实战技巧，讲了一大通，“废话”，精华

 

光讲技术，是没什么稀奇的

 

稀奇的是，我在公司里什么场景下，是怎么用这个技术的，经验，会花很多时间去给大家铺垫各种git命令讲解之前的场景，那些东西我认为比git命令更重要，几十个git命令，你都不知道用来干嘛的，过了2个月，保证你全部忘记

 

记忆曲线，如果脑海中有个场景，有个图片帮助你记忆，你的记忆的遗忘会慢很多，记忆清晰很多，记忆的时间会长很多

 

我觉得，知其然而知其所以然，你不要光学，你为什么要学，你学了以后有什么用

 

BAT，晋级答辩，职级体系，升级打怪

 

每次去要网上晋升一级的时候，非常强调你的思考的能力，你不要告诉我你做了什么？你告诉我，你为什么要做，你为什么要这样做，你这样做了以后效果是什么，作用是什么？

 

技术评审委员会的成员之一，参加过很多初中高级工程师的晋级答辩，工程师，what，我做了什么

 

技术工程师，必须要懂得思考，梳理，总结，抽象，提取。为什么做，为什么要这样做，这样做以后我对公司的产出是什么，价值是什么，业务的促进是什么

 

软素质真正强悍的人，才可以去做技术经理，技术总监，CTO，CEO

 

 

 

1、Git内幕研究的一个思路

 

git add，git commit，git push，这些命令背后在干什么？git内幕原理

 

不是纯讲，要在动手实践中，去给大家讲请清楚这些命令背后的原理是什么？

 

porcelain命令，这些命令，指的就是类似git add，日常开发中，程序员会直接使用的git命令

 

这些porcelain命令底层都是调用的一些git的底层命令，plumbing命令，实际上是不是给我们用的，是给git的那些高阶命令去用的

 

我们在这里讲解git add这些高阶命令背后的原来，是带着大家动手实践去体验一下底层的命令是用来干嘛的，当你把那些对应的底层命令都学会之后，明白那些底层命令在干什么，接着你就会瞬间明白，git add，git commit这些高阶命令，你在执行的时候，他们背后在干什么 => 内幕原理，你就全部清楚了

 

2、.git目录研究

 

平时我们如果执行git init之后，就会生成一个.git目录，其中就是git存放所有数据的地方。如果我们拷贝这个.git目录，那么就相当于备份了一个完整的git项目的数据。

 

在.git目录中，有如下一些内容：

 

config

description

HEAD

hooks/

info/

objects/

refs/

 

description文件仅仅是由Git WebUI程序去使用的，我们一般不用去管这个文件

 

config文件包含了这个git项目的所有配置项

 

info目录中包含了我们在.gitignore文件中定义的不需要git追踪的那些被排除掉的文件

 

hooks目录中包含了我们的client-side或者是server-side的钩子脚本

 

其实上面那些都不是最重要的，最重要的是这几个东西：HEAD文件，indexes文件，objects目录，以及refs目录。

 

objects目录存储了git中所有的数据，refs目录中存储了所有的指针（包括了branch，tag，remote，等等，指向了objects目录中的数据），HEAD指向了当前我们所处的分支指针，indexes文件存储了暂存区中的内容。

 

把接下来的课程听懂之后，你就会了解常见的git命令背后的原理，还有git命令结合.git目录的存储相关的原理

 