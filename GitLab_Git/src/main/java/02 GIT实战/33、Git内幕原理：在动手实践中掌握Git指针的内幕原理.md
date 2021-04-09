.git/objects里面的内容是怎么回事儿，跟git hash-object/git update-index/git write-tree/git commit-tree等底层命令的关系是怎么回事儿，底层命令跟git add和git commit命令结合起来是怎么回事儿，图解

 

git提交那一块，你完全掌握了底层的内幕原理了

 

除了存储那些数据，git里面有很多种指针，tree/commit里面本来就含有指针，指向其他的blob或者是tree，暂存区里面也是指针

 

分支、HEAD、tag、remote

 

.git/refs这个目录里面的内容

 

（1）分支指针

 

如果我们要查看某个commit，那么还是要知道它的hash值，然后用git log hash_value来查看这个commit。但是这还是很麻烦，所以最好是可以有个文件存储那个hash值，然后我们直接用一个较为简单的名称来引用。

 

.git/refs目录中，有几个子目录，包括了heads和tags两个子目录，在这里就有各种指向objects中的指针。

 

.git/refs/heads目录下，有一个master文件，实际上是git自动创建一个初始的分支，分支其实就是一个指针而已，指向了某个commit object的，在master文件中就包含了一个40位的hash值，当前就是指向了最近一次commit object

 

.git/refs/heads目录下，会包含很多个文件，就是每次创建一个分支，就会在这个里面对应新建一个文件，文件名称就是分支的名称，文件里的内容就是分支指针当前指向的那个commit object的hash值，代表着这个分支当前指向了哪个commit object

 

我们下面动手实践一下，用git底层的命令，看看，我们手动去创建分支会是什么样的效果

 

我们可以用下面的命令创建一个名叫feature/001的指针，指向了第一次提交的commit object

 

echo 1a410efbd13591db07496601ebc7a059dd55cfe9 > .git/refs/heads/feature/001

 

接着用下面的命令，就可以查看master指向的commit object之前的commit历史

 

git log --pretty=oneline feature/001

 

总结一下，git里面的分支的内幕原理，大家就彻底清除了，新建一个分支是很轻量级的一个事情，不是像其他的版本控制系统那样，新建一个分支，是将所有的文件内容和版本都拷贝一份

 

其实在git中，提交历史，blob+tree+commit，是最核心的，所有的数据都是通过这个方式来存储一份

 

然后呢？新建一个分支，实际上就是在.git/refs/heads下面新建对应的目录和文件，在文件里面维护一个指针，一个hash值，指向了.git/objects中某个commit object的hash值

 

通常不建议直接自己手动修改refs文件的内容，可以用下面的命令更新某个refs文件指针指向的commit object：

 

git update-ref refs/heads/feature/001 1a410efbd13591db07496601ebc7a059dd55cfe9

 

其实上面就已经揭示出来了，这就是git中的分支机制的根本原理，分支就是指针而已，分支指针指向了不同的commit object。用下面的命令可以再创建一个test分支，让其指向某个commit object：

 

git update-ref refs/heads/test cac0ca

 

实际上我们如果执行git branch命令，就是执行update-ref命令，创建那个分支对应的文件，然后文件内容就是对应的commit object的SHA-1 hash值。

 

删除一个分支很简单的，就是删除.git/refs/heads下面的分支对应的目录和文件即可

 

（2）HEAD指针

 

HEAD文件中，保存了对某个分支指针的引用，其实就是某个分支对应的文件。

 

HEAD其实也是一个文件，但是也是一个指针，这个HEAD呢，就是指向了当前你所处的那个分支而已，保存的是refs/heads下面的分支对应的目录和文件名

 

比如当前你在master分支，那么HEAD文件中的内容就是refs/heads/master，代表了HEAD当前指向了master分支

 

再比如当前你在feature/001分支，那么在HEAD文件中，内容就是refs/heads/feature/001，代表了HEAD当前指向了feature/001分支

 

因此如果我们比如在master分支上，创建并且切换到了另外一个分支，那么此时会根据HEAD找到当前分支文件，再找到当前分支指向的commit object，那么新建的分支就会同样指向那个commit object。

 

从比如master分支切换到feature/001分支，实际上就是一个非常轻量级的操作

 

（1）HEAD文件里的内容，从refs/heads/master，变为refs/heads/feature/001，代表着HEAD指针指向了feature/001分支

 

（2）将feature/001分支指向的那个commit对应的tree，对应的那些blob对应的文件版本的内容，一次性从仓库里恢复到工作区中，工作区中所有文件的版本和内容，会变成那个分支指向的commit当时的那个快照版本

 

每次切换分支之后，HEAD都会指向那个分支的文件

 

可以通过git checkout命令+cat .git/HEAD命令结合起来，然后就可以看到每次切换后的HEAD值

 

每次我们执行一次git commit操作，都会新建一个commit object出来，然后会让当前分支指向最新的commit object。

 

通过下面的命令可以读取和修改HEAD文件的指针：

 

git symbolic-ref HEAD

git symbolic-ref HEAD refs/heads/test

 

到这里为止，.git/refs下面最重要的东西就讲解完了，分支和HEAD

 

分支就是一个文件，保存了指向的commit的40位hash置

 

HEAD也是一个文件，保存了指向的那个分支对应的文件名称

 

（3）tag object

 

其实除了blob、tree和commit三种object之外，还有一种object就是tag object。tag object是指向某个commit object的，包含了标签时间，标签名称，等等。而且tag object永远就指向创建时指定的那个commit object，不能修改。

 

我们可以通过下面的命令创建一个轻量级的tag object

 

就只是一个指针而已，在refs/tags下面，会有一个tag名称对应的文件，然后里面就是那个tag指向的commit的40位hash值

 

git tag v1.0：轻量级的tag，直接就指向了当前分支指向的那个commit object，就是在refs/tags下面搞一个问加你，作为一个指针而已

 

git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d

 

当然也可以创建一个带备注的标签：git tag -a v1.1 -m 'test tag'

 

此时查看cat .git/refs/tags/v1.1，会发现返回的不是我们指定的那个commit object，是另外一个新创建的tag object的SHA-1 hash

 

git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2

 

再次查看那个tag object的hash值，此时可以显示出来其指向的那个commit object，同时包含了tag的创建时间、创建人、备注，等等信息。

 

（4）remote

 

最后一种引用类型，就是remote。

 

如果我们添加了一个remote，同时push了一些东西到那个remote，git会在refs/remotes中保留对每个分支，最近一次push到远程服务器对应的commit。

 

比如下面的命令：

 

git remote add origin git@github.com:schacon/simplegit-progit.git

git push origin master

 

cat .git/refs/remotes/origin/master

 

查看一下上面的文件，会看到一个SHA-1 hash值，就是这个分支最近一次push到服务器的object commit的SHA-1 hash值。

 

FETCH_HEAD，就是最近一次fetch下来的每个分支对应的commit hash值，就在这个里面

 

（5）梳理一下

 

分支：.git/refs/heads

HEAD：.git/HEAD

tag：.git/refs/tags

remote：.git/refs/remotes

FETCH_EHAD：.git/FETCH_HEAD

 

 

773692218