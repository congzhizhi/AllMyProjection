（1）git数据存储机制

 

1）git hash-object初体验

 

git的数据存储格式实际上是一种非常简单的key-value存储格式。就是说，git支持将任何东西存储到其数据库中，一般就是存储文件的内容，然后用一个key值来引用它。

 

首先随便找一个目录，初始化为git项目：

 

git init

 

此时去.git/objects目录中检查一下，发现只有两个子目录，info和pack，没有任何文件，此时数据库是空的。

 

用下面命令，可以手动将一个字符串作为value存储到git的数据库（objects，git的数据库，用了比较简单的方式去存储）中去，会看到返回了一个hash值来引用那个字符串value：

 

echo 'hello world' | git hash-object -w --stdin

 

解释一下上面的命令：

 

git hash-object就是git的一个底层命令，一般我们是不会直接用的，一般都是git add，git commit之类的命令在调用这种底层命令。

 

git hash-object命令，默认的表现是这样的，接受一个value，然后针对这个value返回一个hash值，默认是不将value写入自己的数据库的

 

但是使用了-w之后，git就会将value写入数据库之中，同时用hash值作为key来引用那个value对应的文件。同时--stdin指的是从命令行接收value，也就是echo ‘test content’中的test content。如果不使用--stdin，那么要求是在hash-object之后跟一个文件名，它就会将一个文件作为value存储到数据库中去，然后返回一个hash值。

 

git返回的是SHA-1 hash值，是40位的字符串，是根据文件内容计算出来的一个checksum，校验和。

 

2）git的文件存储机制（轻量级索引机制）

 

你往git里存储一个文件，git一定会根据文件的内容计算一个40位的hash值出来，作为那个文件的名称，也是指针，40位hash值就可以来引用这个文件。存储的时候，40位hash的头2位，会作为目录名称放在objects里面，然后那个存储的文件会放在那个目录中，文件名是40位hash值的后38位。

 

大家来思考一下，git作为一个高性能的版本控制系统，一大特点就是性能超高，在切换分支的时候，都需要进行大量的磁盘读写，磁盘存储机制是比较值得我们注意的

 

将40位hash值的头2位作为目录，其实你可以认为是一种轻量级的索引机制

 

这样的话，有一个好处，就是每次你要根据一个hash值定位一个数据的时候，直接根据头2位先定位到一个目录，然后再在这个目录下去查找，linus写git的时候，就是用了一种非常轻量级的文件索引机制

 

直接查看git存储的文件的内容是不ok的，我们后面会讲解要用git专用的底层命令来查看

 

3）git cat-file体验

 

使用下面的命令可以查看刚存储进去的文件值，git cat-file是另外一个底层命令，就是专门用来查看存储到git中的文件的内容的：

 

git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4

 

4）一个文件的多个版本存储机制的体验

 

下面的命令可以将一个文件存储到git数据库中：

 

echo 'version 1' > test.txt

git hash-object -w test.txt

 

echo 'version 2' > test.txt

git hash-object -w test.txt

 

git会将一个文件的多个不同版本，以完整快照的方式存储在objects数据库中，每个版本都用一个单独的文件来存储，每个版本的文件都有一个不同的hash值。

 

使用下面的命令可以查看.git/objects中所有的文件：find .git/objects -type f

 

接着可以试着将这个文件给删除，然后从git数据库中再恢复回来这个文件：

 

rm -rf test.txt

git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt

cat test.txt

 

也就是说，我们只要将数据存储到了git中，那么就不用担心丢失了，随时都可以找回来这个文件的任意一个版本

 

我们用下面的命令可以查看一下git中存储的文件的类型，默认都是blob：

 

git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

 

这里，我们就发现跟之前给大家讲解的那个原理就可以串起来了，因为实际的文件的内容，每个文件的版本快照，会作为一个blob object存储在git数据库中

 

git数据库，git仓库，database，repository

 

5）梳理一下

 

实际上，就是一个文件的不同的版本，都会用一个完整的快照的形式，作为一个单独的文件存储在git数据库中，所谓的完整快照，每个版本，都是用一个文件存储这个版本的完整的内容的

 

每个版本的文件内容，都是通过blob object的类型存储在git数据库中的

 

（2）tree object

 

git有另外一种object叫做tree object，每次将一堆文件的一个版本存储到git仓库中，都是用blob object来存储那些文件的版本，然后用一个tree object来引用多个blob object。tree object通常会包含指向其他多个blob的指针，或者是其他tree object的指针。用下面的命令，可以查看master分支指针指向的那个tree object：

 

git cat-file -p master^{tree}

 

git通常是基于暂存区中的内容来创建一个tree object的，我们首先需要在暂存区中加入一些内容，然后可以试着手动创建一个tree object。可以将仓库中存储的某个commit object的内容放入暂存区中：

 

git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt

 

git update-index：底层命令，一般是git add命令去调用的，将数据库中存储的某个blob数据的引用放到暂存区中

 

--add：因为暂存区中还没有这个文件，你可以认为这个文件是第一次进入暂存区，所以需要这个选项

 

--cacheinfo：不是从工作区中加入文件放到暂存区，是从git仓库（git数据库）中加入文件

 

100644 mode：这种模式的意思是，这个是一个普通的文件

 

接着使用下面的命令基于暂存区中的内容，创建一个tree object：

 

git write-tree：将暂存区中的内容创建为一个tree object，tree object也是放入了git仓库，主要是包含了对一个blob obejct的引用

 

git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579

git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579

 

下面我们再加入一个new.txt文件放入暂存区中，同时将test.txt文件的另外一个版本放入暂存区中，再次创建一个tree object：

 

echo 'new file' > new.txt

git update-index --add new.txt，这个就是将工作区里的内容直接放入暂存区

git update-index --add --cacheinfo 100644 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt

 

git write-tree

git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341

 

此时会发现这个tree object对应着两个文件

 

下面的命令可以从仓库中读取一个tree object放入暂存区，然后再基于这个tree object创建一个新的tree object出来：

 

所谓的将仓库里的东西读出来放到暂存区，实际上只是将一些引用放到暂存区，但是blob，tree，实际还是存储在objects中的

 

git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579

git write-tree

git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614

 

此时tree object包含两个文件和一个tree object，如果将这个tree object的数据恢复到工作区中，就会有两个文件和一个叫做bak的文件夹，bak文件夹中也包含了一个文件。

 

（3）commit object

 

随着你不断的在工作区中编写代码，然后呢，每次如果执行了一些相关的操作，可以将当前这个时刻，有变化的文件的版本都存储到git仓库中，同时放入暂存区中。

 

此时，暂存区中，没有变化的文件，还是保持着之前的版本；有变化的文件，暂存区中会存放最新版本的blob对应的引用

 

然后，如果此时创建一个tree object，就是基于暂存区中当前所有的文件的版本的blob，创建一个tree

 

此时，这个tree object就代表了这个项目的所有代码的此时此刻的一个完整的快照

 

现在我们实际上是有3个tree object，分别代表了项目的不同时刻的快照。但是此时此刻，我们有个文件，就是不知道那些tree object是谁添加的，为什么添加，什么时候添加的，什么都不知道。

 

但是，光有tree object是不够的

 

此时就需要commit object了。

 

用下面的命令，基于已有的tree object创建一个commit object，同时给这个commit object一个提交备注：

 

echo 'first commit' | git commit-tree d8329f

 

然后查看一下这个commit object，会发现包含了一个tree object，同时又作者，提交日期，以及提交备注：

 

git cat-file -p fdf4fc3

 

commit object的含义很简单，就是指向了一个tree object，而那个tree object指向了多个blob，其实这个tree object就代表了某一时刻项目中所有代码文件的快照版本，同时那个commit object包含了作者、提交时间、提交备注，此外还包含了指向上一次commit object的指针。

 

同样的，我们可以为另外两颗tree object分别创建一个commit object，代表了另外两次提交：

 

echo 'second commit' | git commit-tree 0155eb -p fdf4fc3

echo 'third commit' | git commit-tree 3c4e9c -p cac0cab

 

此时相当于我们就有了三次跟真实一样的提交了，可以用git log来查看一下提交历史：

 

git log --stat

 

就会看到最近的3次提交

 

其实上面的实验步骤就彻底揭示了git的底层原理了git add和git commit命令执行时使用的就是上面那些低级别的命令：

 

git add：相当于是将你的有修改的文件作为一个新的版本，采用单独的blob文件存储到仓库中去，同时将仓库中的文件更新到暂存区中

 

git commit：相当于就是将暂存区中的文件快照，也就是对应的blob指针创建一个tree object，写入仓库中，同时再创建一个commit object包含了提交人/提交时间/提交备注，来指向那个tree object，代表了一次提交对应的仓库快照

 

blob、tree和commit，三种object都是作为一个独立的文件在objects目录中存储的，存储时的子目录命名和文件名，都是基于hash值来的

 

最后用一张图来说明，到目前为止，整个commit object、tree object和blob的关系。

 

（4）SHA-1 hash值如何计算

 

首先blob的内容分为两块，一个是header，一个是content。header就是类型+空格+长度+\u0000。然后会用header+content来计算出一个SHA-1校验和，作为hash值。

 

（5）梳理一下

 

git update-index --add 文件名：其实就是，我们在工作区里修改了文件以后，会用这个底层命令，将工作区中有修改的文件的一个版本，放入git仓库中作为一个blob去存储这个版本，同时将这个blob的引用放入暂存区中

 

多次有git update-index之后，git仓库中会存储项目中每个文件的所有版本，同时在暂存区中保留了每个文件的最新版本，没有变化的就保留值钱的版本

 

执行git write-tree，就可以针对暂存区中当前项目的最新版本，创建一个tree object，引用项目中所有的文件当前此时此刻的一个版本对应的blob，tree object，就代表了项目当前的一个完整快照版本

 

接着执行git commit-tree，就可以基于这个tree object创建一个commit object，包含了tree object的创建人，创建时间，备注信息，等等，作为一次提交

 

（6）将上层命令和底层命令结合起来

 

1）我们首先在工作里各种新增、修改文件

 

2）我们执行git add --all .这个命令：此时会将工作区中有变化的文件，作为一个新的版本，直接存储到git仓库中去，作为一个blob来存储；同时在暂存区中加入这些最新版本的blob的引用，替换同一个文件之前在暂存区中的版本

 

git update-index

 

3）我们执行git commit，同时给出一个提交备注：此时会针对暂存区中现在的所有的版本，创建一个tree object，作为项目此时此刻的一个快照版本，放入仓库；另外再创建一个commit object，包含了提交人，提交时间，提交备注，来引用这个tree object

 

4）到此为止，完成一次git add和git commit

 

5）整个git内幕原理，剖析的非常清楚了

 

（7）最后再来一张图，结合命令，演示一下

 

1）编写两个文件，test1.txt和test2.txt

 

2）git add --all .

 

将两个文件的第一个版本，作为两个blob保存到了Git仓库中，同时也放入了暂存区中

 

3）git commit -m "第一次提交"

 

基于暂存区里的两个文件的第一个版本，创建了一个tree

 

然后基于提交信息创建了一个commit，引用了那个tree，此时这个commit就代表当前这个项目此时此刻的一个完整的版本

 

4）修改test1.txt位第二个版本

 

5）git add --all .

 

将test1.txt的第二个版本作为blob存储到git仓库中

 

同时将test1.txt的第二个版本覆盖了暂存区中的test1.txt的第一个版本

 

6）git commit -m "第二次提交"

 

创建了一个tree，引用了test1.txt的第二个版本，test2.txt的第一个版本

 

另外创建了一个commit，引用了那个tree，代表了此时此刻项目的一个完整的版本

 

7）每个commit，就是当前项目的一个完整的快照版本，包含了项目的所有文件的一个最新版本

 

8）为什么大公司里，规范项目里，对git commit都有要求的，不要随便瞎提交，都是完成一个完整的功能或者模块之后再提交一次，尤其是每天提交一次，或者每天提交多次，每次代表了一个完整的功能和模块

 

9）整个你的项目的提交历史，就是项目版本的变迁的一个完整的历史，每个commit都代表引入了一个新的功能或者是模块

 

 

 

 

 

 

773692218