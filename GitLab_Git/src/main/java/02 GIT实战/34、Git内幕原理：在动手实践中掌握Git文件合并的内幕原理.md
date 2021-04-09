
我们来通过一个大文件的存储以及git对这个文件多版本存储的优化，作为例子，来看一看git对大量的文件在存储上是如何做优化的773692218

 

用下面的命令将一个大文件存入git中：

 

curl https://raw.githubusercontent.com/mojombo/grit/master/lib/grit/repo.rb > repo.rb

git checkout master

git add repo.rb

git commit -m 'added repo.rb'

 

然后看一下master指向的commit object对应的tree object包含的内容，master分支指向了一个commit object，commit object是指向了一个tree object，tree object指向了此时此刻项目中所有文件的一个最新版本

 

git cat-file -p master^{tree}

 

用下面的命令看一下新加入进去的文件有多大：

 

git cat-file -s 033b4468fa6b2a9547a70d88d1bbe8bf3f9ed0d5

 

在文件中加入一些新的内容，然后再次提交：

 

echo '# testing' >> repo.rb

 

git commit -am 'modified repo a bit'：直接将工作区中的修改过的文件加入暂存区，再执行一次提交，但是我一般不常用

 

再次查看master对应的最新一次commit object对应的tree包含的blob，发现原来的文件换了一个，也就是修改后的文件重新存储了一份

 

git cat-file -p master^{tree}

 

总结：哪怕是很大的文件，比如说20kb的一个代码文件，每次改动一点代码，git也是存储改动后的版本对应的一个完整的文件；如果这个20kb的代码文件修改了100次，会怎么样？存储100个版本对应的文件，也就是100个文件，2000kb ~ 2MB。所以这样的话对存储的压力还是蛮大的。。。。

 

git实际上是可以更加智能的处理这种文件改动的。默认情况下，git会使用loose格式存储文件（loose，松散格式，采取这个文件的每个版本，就存储一个完整的单独的文件），但是一段时间过后，git会将多个loose格式的松散文件打包到一个packfile二进制文件中，来节省磁盘空间。

 

什么时候git会进行packfile打包的操作呢？两个时机：我们手动执行git gc操作，或者是将文件push到远程服务器的时候，git会看一下loose格式文件是不是太多了，如果是，则进行打包合并。

 

git gc：garbage collector，jvm gc是不一样的，git用来处理自己的文件的

 

如果我们将本地的commit数据push到了远程服务器后，就代表远程服务器包含了我们的数据了，此时git就可能会执行一次packfile打包

 

此时我们可以手动执行一下git gc命令，来看看效果

 

此时再次用下面的命令看一下objects目录下的内容：find .git/objects -type f，之前应该是一堆hash值组成的目录和文件，包含完了十几个object（blob，tree，commit），会发现出现了objects/info/packs和objects/pack等新的目录。

 

我们是将之前所有的hash值组成的文件打包成了一个packfile文件，进行了压缩以及合并

 

pack包含了两个文件，一个是packfile包含了打包的所有数据，一个是index文件，包含了每个blob object在packfile中的offset，通过.idx文件标识出每个object在.pack文件里是哪一个部分

 

git做文件存储的优化，两块：多个文件打包成给一个，压缩，二进制格式，packfile，节省空间；在打packfile的时候，就会对我们刚才说的那种情况，大ruby文件，仅仅保留第一个版本对应的完整内容，然后之后的版本都是直接保留的是它的那些delta增量内容，后面的版本就不是全部保留全量内容了，进一步节约空间

 

git，git gc；git push，自动会做打packfile

 

实际上，在git进行打包packfile的时候，就会对一个文件的多个版本，仅仅保留其每次增量修改数据，避免对一个文件保留多个全量的版本快照。

 

用下面的命令看一下：

 

git verify-pack -v .git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.idx

 

会发现ruby大文件，第一个版本仅仅包含9kb的内容，但是第二个版本包含了22kb的内容，所以这里就做了增量处理和优化，仅仅对一个大文件的最新的版本保留了全量的内容，但是对之前的版本都是增量内容

 

总结一下

 

（1）git gc / git push，会执行packfile的压缩优化

（2）将.git/objects下的各种object对应的文件，压缩成一个packfile

（3）每个packfile都对应一个.pack文件和一个.idx文件，.pack文件包含了多个object的内容，.idx文件包含了每个object在那个.pack里面的offset偏移量

（4）在打packfile的意义：多个文件合并一个，节省空间：对大文件进行增量处理，仅仅是最新的版本保留全量内容，之前的版本保留增量内容

（5）用git verify-pack -v命令，可以查看.pack文件里包含的具体内容

 

 

 