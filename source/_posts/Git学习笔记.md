###  Git介绍

分布式版本控制工具 VS 集中式版本控制





### Git安装

### Git命令

基于开发案例，详细讲解git 的常用命令

### 版本穿梭

```git
命令git reflog 可以查看当前本地库的版本提交状态， 每个版本的地址引用，通过这个地址，我们可以进行版本穿梭
== git reflog （简略信息版）

b7a61e6 (HEAD -> master) HEAD@{0}: checkout: moving from hot-fix to master
c813c2a (hot-fix) HEAD@{1}: checkout: moving from master to hot-fix
b7a61e6 (HEAD -> master) HEAD@{2}: commit (merge): merge
98c9077 HEAD@{3}: checkout: moving from hot-fix to master
c813c2a (hot-fix) HEAD@{4}: commit: hot-fix commit
aeb4bd5 HEAD@{5}: checkout: moving from master to hot-fix
98c9077 HEAD@{6}: commit: master test
aeb4bd5 HEAD@{7}: merge hot-fix: Fast-forward
1a3ca2e HEAD@{8}: checkout: moving from hot-fix to master
aeb4bd5 HEAD@{9}: commit: hot-fix first commit hello.txt
1a3ca2e HEAD@{10}: checkout: moving from master to hot-fix
1a3ca2e HEAD@{11}: reset: moving to 1a3ca2e
087386a HEAD@{12}: commit: third commit hello.txt
1a3ca2e HEAD@{13}: commit: second commit hello.txt
afdb66a HEAD@{14}: commit (initial): first commit hello.txt

== git log  (详细信息版)
commit b7a61e67ea93e3705b06d47baaab95456caf03ab (HEAD -> master)
Merge: 98c9077 c813c2a
Author: yulu <2723946289@qq.com>
Date:   Sun May 9 01:11:53 2021 +0800

    merge

commit c813c2a329b9ca158421ddb9178c2d4111678cc8 (hot-fix)
Author: yulu <2723946289@qq.com>
Date:   Sun May 9 01:08:02 2021 +0800

    hot-fix commit

commit 98c9077bfe85e15065502271722f0731fd7d7c81
Author: yulu <2723946289@qq.com>
Date:   Sun May 9 01:07:01 2021 +0800

    master test

commit aeb4bd56e09974ebc1499531eb7caefbc03010c6
Author: yulu <2723946289@qq.com>
Date:   Sun May 9 01:01:48 2021 +0800

    hot-fix first commit hello.txt

commit 1a3ca2e393f832c36b02bbdffef7896b48826e0a
Author: yulu <2723946289@qq.com>
Date:   Sun May 9 00:39:40 2021 +0800

    second commit hello.txt

commit afdb66ab517b91b429e68dd181ca8ed00f7e05e5
Author: yulu <2723946289@qq.com>
Date:   Sun May 9 00:35:50 2021 +0800

    first commit hello.txt

通过 git reset --hard 穿梭的版本地址 可以进行版本穿梭


```





### Git分支

分支特性、分支创建、分支转换、分支合并、代码合并冲突解决

![未命名文件 (1)](../../../../Downloads/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6%20(1).png)

git的分支操作其实实际上就是操纵两个重要的指针：

1. HEAD指向分支的指针
2. Master（具体分支）指向版本的指针

> 在分支merge时， 我们当前所在的分支为我们的目标的分支， 通过git merge命令可以将指定的分支合并到 **当前所在的分支**



#### 分支冲突

当两个分支进行合并时，如果两个分支在<font color=red>同一文件的同一个位置</font>有两套完全不同的修改，git无法替我们决定使用哪一个。必须人为决定新代码的内容。（需要人为修改当前分支下的产生冲突的文件）

例如：

1. master 分支 当前版本 的hello.txt文件的内容：

> hello world	10
>
> hello world	20
>
> hello world
>
> hello world
>
> hello world
>
> hello world master test
>
> hello world

2. Hot-fix 分支 当前版本 的hello.txt文件的内容：

> hello world	10
>
> hello world	20
>
> hello world
>
> hello world
>
> hello world
>
> hello world
>
> hello world hot-fix test



这样这两个分支在合并的时候，就会产生合并冲突，无法自动进行分支合并，我们需要在执行 git merge hot-fix之后，人为的修改master分支下的hello.txt文件

执行完命令之后，hello.txt文件中会多出两个版本的信息，我们人为的修改我们想要保留的信息，然后add，commit（不加文件名）即可



### Git 团队协作机制

#### 团队内协作

![image-20210509013649215](../../../../Library/Application%20Support/typora-user-images/image-20210509013649215.png)



形象的比喻：

岳不群，将自己本地库的**华山剑法 **push（推送）到**远程库**

令狐冲就可以从**自己团队的远程库中**==clone==这套华山剑法到自己的**本地库**进行剑法的修炼，在练剑的同时，令狐冲有新研究出两招剑式，添加到自己的本地库中，令狐冲通过push，再将修改后的代码push到团队的远程库中(<font color=red>需要团队内的权限才可以push成功</font>)

岳不群再从远程库pull拉去修改后的华山剑法





#### 跨团队协作

![image-20210509014532309](../../../../Library/Application%20Support/typora-user-images/image-20210509014532309.png)

形象的例子：

令狐冲和岳不群：都觉得华山剑法不太精妙， 于是令狐冲找来了东方不败，将**华山剑法**升级为**辟邪剑法**

这就会有两种情况：

​		1：将东方不败加入到自己的团队（<font color=green>东方不败有自己的团队，堂堂日月神教CEO岂能加入到小小的华山派</font>）

​		2: 东方不败跨团队协助令狐冲完成华山剑法的升级

针对第二种情况：

​	首先：东方不败需要从**岳不群的远程库** <font color=red>fork</font>出一份岳不群的远程库， fork成自己的远程库

​	然后：东方不败从自己的远程库 将华山剑法 clone下来，升级完成之后，在push到自己的远程库

​	接下来：提交pr：pull request拉去请求（东方不败的远程库向岳不群的远程库**提交pr请求**，待岳不群进行**审核**完成之后进行两个远程库的**合并**）

​	最后：两个不同团队之间的远程库合并完成之后，岳不群就可以拉去 **辟邪剑法**，进行**自宫修炼**



#### 远程库创建别名

```shell
git remote -v 查看当前已经配置的远程库别名

git remote add 别名 地址
	例如： git remote add gitDemo https://github.com/xiaowodi/gitDemo.git
didi@bogon gitDemo$ git remote -v
didi@bogon gitDemo$ git remote add gitDemo https://github.com/xiaowodi/gitDemo.git
didi@bogon gitDemo$ git remote -v
gitDemo	https://github.com/xiaowodi/gitDemo.git (fetch)
gitDemo	https://github.com/xiaowodi/gitDemo.git (push)
```



#### 本地库 push 代码 到 远程库

```shell
git push 远程库https 远程库分支
didi@bogon gitDemo$ git push gitDemo master
```



#### 拉去远程库到本地库（拉取需要登录账号）

当远程库进行修改了之后，而本地库没有被修改时，可以通过pull，将远程库拉取到本地

```shell
$ git pull 远程库地址 分支
$ git pull gitDemo	master
```

**注意：**<font color=red>pull 可以将远程库下修改之后的文件内容 拉去到本地</font>  可以将远程库与本地库保持同步，使得**本地库的转态保持最新状态**

​			<font color=red>而clone + 远程地址：这种方式只能表面的文件，如果原始文件存在，那么 即使远程库修改了内容，通过clone也不会将变化的内容克隆到本地</font>

git pull 作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。

> eg:$ git pull ##远程主机（origin） #远程分支(next)#：#本地分支(master)#
>
> 如果远程分支是与当前分支合并，则冒号后面的部分可以省略。 eg:$ git pull #远程主机(origin)#　 #远程分之(next)#



#### 克隆远程库

克隆公共仓库的代码，不需要登录账号的

克隆可以在任意的文件夹下，将远程库的代码克隆到本地库

```shell
$ mkdir newgitDemo
$ cd newgitDemo
$ git clone https://github.com/xiaowodi/gitDemo.git
```

克隆之后，克隆下来的文件夹就自动的配置了git的配置，就连远程库的别名都已经起好了

克隆大致会经历的三个步骤：

	1. 拉去代码
	2. 初始化本地仓库
	3. 创建远程库地址别名



### 日常push的主要步骤

如果本地库的想要push的版本，低于远程库的版本，那么push是被拒绝的，

一定要保证本地库的版本要高于远程库的版本

<font color=red>一般需要先pull远程库的代码，将远程库最新的版本拉取到本地库，然后在此基础上进行修改，提交，最后在push到远程库进行合并</font>



<font color=green>在拉取远程库代码的过程中：如果远程库的代码与本地库的代码不一致，会自动合并，如果自动合并失败，还会涉及到手动解决冲突的问题</font>

### Git内部原理

#### .git目录结构

4个重要的条目：HEAD文件、（尚未创建的）index文件、objects目录、refs目录。

```shell
didi@localhost LeetCode-Daily$ ls -al .git
total 64
drwxr-xr-x  16 didi  staff   512  5 23 14:33 .
drwxr-xr-x   8 didi  staff   256  5 23 11:35 ..
-rw-r--r--   1 didi  staff    12  5 23 11:58 COMMIT_EDITMSG
-rw-r--r--   1 didi  staff   205  5 23 00:44 FETCH_HEAD
-rw-r--r--   1 didi  staff    25  5 23 10:59 HEAD							#####   HEAD文件指向目前被检出的分支   #####
-rw-r--r--   1 didi  staff    41  5 23 11:09 ORIG_HEAD
-rw-r--r--   1 didi  staff   483  5 23 11:35 config						#	config文件包含项目特有的配置选项。
-rw-r--r--   1 didi  staff    73  5 23 00:24 description			#	description文件仅供gitWeb程序使用
drwxr-xr-x  15 didi  staff   480  5 23 00:24 hooks						# hooks目录包含客户端或服务端的钩子脚本（hook script）
-rw-r--r--   1 didi  staff   916  5 23 11:58 index						#
drwxr-xr-x   3 didi  staff    96  5 23 14:34 info							# info 目录包含一个全局性排除（global exclude）文件，用以放置那些不希望被记录在.gitignore文件中的忽略模式（ignored patterns）。
drwxr-xr-x   4 didi  staff   128  5 23 00:24 logs
drwxr-xr-x   3 didi  staff    96  5 23 11:35 modules
drwxr-xr-x  55 didi  staff  1760  5 23 11:58 objects					#####   objects目录存储所有数据内容   #####
-rw-r--r--   1 didi  staff   182  5 23 00:24 packed-refs
drwxr-xr-x   6 didi  staff   192  5 23 11:11 refs							#####   refs目录存储指向数据（分支、远程仓库和标签等）   #####
```



#### Git对象（objects）

==git对象主要包括三种：数据对象（blob），树对象（tree），提交对象（commit）==

##### 数据对象：blob

> 数据对象记录index（暂存区中的文件数据）

Git是一个内容寻址文件系统，Git的核心部分是一个简单的键值对数据库（key-value data store）。可以想Git仓库中插入任意类型的内容，他会返回一个唯一的建，通过该键可以在任意时刻再次取回该内容。

**git hash-objec**t 可以将任意数据保存与.git/objects目录（即**对象数据库**，一般我们commit 对象之后的存储位置）

```sheel
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

在这种最简单的形式中，`git hash-object`会接受你传给它的东西，而它只会返回可以存储在Git仓库中的唯一键。`-w Actually write the object into the object database.`选项会指示该命令不要只返回键，还要将该对象写入数据库中。`--stdin`选项则指示该命令从标准输入读取内容；若不指定，则需在命令尾部给出待存储文件的路径。

此命令输出一个长度为40个字符的校验和。这是一个SHA-1哈希值-- 一个将待存储的数据外加一个头部信息（header）一起做SHA-1校验运算而得的校验和。

校验和的前两个字符用于命名子目录，余下的38个字符则用作文件名。

```shell
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

一旦你将内容存储在了对象数据库中，那么可以通过 `cat-file`命令从git那里取回数据。`-p`选项可指示该命令自动判断内容的类型。

至此，目前已经介绍了如何向git中存入内容，以及如何将他们取出。接下来，模拟对一个文件进行简单的版本控制。

```shell
# 1.首先创建一个新文件并将其内容存入数据库：
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30	
# 2.向文件中添加内容并将其内容存入数据库
$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	# 文件内容发生改变，其键值对的键值就会发生改变
# 3.从新提交为改变的文件，测试结果如何
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	# 如果文件没有发生变化，内容没有变化，重新添加文件，会返回和上次相同的key
# 再来看看.git的目录变化
$ tree .git/
.git/
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects																					# objects目录下多出了两个目录 1f 和 83，这两个目录的名字对应40位校验和的前两个字符
│   ├── 1f
│   │   └── 7a7a472abf3dd9643fd615f6da379c4acb3e3a  # 对象实际存放的位置
│   ├── 83
│   │   └── baae61804e65cc73a7201a7252750c76066a30	# 对象实际存放的位置
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

10 directories, 19 files

# 4.然后我们在本地删除这个文件
$ ls
test.txt
$ rm test.txt 
$ ls
# 5.接下来我们可以从git数据库中恢复任意的版本；使用git cat-file
# 这里从第一个版本恢复
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt 
version 1
```

> 然而，记住文件的每个版本所对应的SHA-1值并不现实；另一个问题是，在这个（简单的版本控制）系统中，文件名并没有被保存--我们仅保存了文件的内容。
>
> 上述类型的对象我们称之为**数据对象（blob object）**。利用`git cat-file -t`命令，可以让Git告诉我们其内部存储的任何对象的类型，我们只需提供该对象的SHA-1值：
>
> ```shell
> $ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
>   blob
> ```

##### 树对象

树对象能解决文件名保存的问题，也允许我们将多个文件组织到一起。所有内容均以树对象**tree**和数据对象**blob**的形式存储，其中树对象对应了unix中的目录项。

例如：

```shell
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859		README		# 数据对象	代表一个文件数据
100644 blob 8f94139338f9404f26296befa88755fc2598c289		Rakefile	# 数据对象
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0		lib				# tree 树对象 	代表一个目录
```

<font color=green>master^{tree}</font> 语法表示<font color=green> master </font>分支上最新的提交所指向的树对象

第一列为文件模式：

| 文件模式 | 说明                 | 备注                                                        |
| -------- | -------------------- | ----------------------------------------------------------- |
| 100644   | 表明这是一个普通文件 |                                                             |
| 100755   | 表示一个可执行 文件  |                                                             |
| 120000   | 表示一个符号链接     |                                                             |
|          |                      | 上述三种文件模式即为git文件的所有模式（参考unix的文件格式） |
| 040000   | 表示一个目录项       |                                                             |

第二列表示树对象中节点的类型

| 类型 | 说明                               | 备注 |
| ---- | ---------------------------------- | ---- |
| blob | 节点数据，表示该节点是一个文件数据 |      |
| tree | 树数据，表示该节点是一个目录项     |      |



###### Git 内部存储的数据（tree对象）

<img src="../../../../Library/Application%20Support/typora-user-images/image-20210615175949733.png" alt="image-20210615175949733" style="zoom:50%;" />



###### 底层命令

















<font color=blue>特此说明：</font>

- 我们每次commit提交后的数据，都会存储成树对象，每次提交后会自动的在objects目录下创建两个对象（tree对象 和 commit对象）
- 添加到暂存区中的数据，会在objects下创建一个blob类型的数据对象，记录文件的内容

| blob对象   | 记录index暂存区中文件的实际内容                             | **$** git cat-file -p 3b18e512dba79e4c8300dd08aeb37f8e728b8dad<br />hello world |
| ---------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| 树对象     | 记录了提交的暂存区中的文件信息（包括类型，SHA-1值和文件名） | **$** git cat-file -p c3b8bb102afeca86037d5b5dd89ceeb0090eae9d<br />100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad	test.txt |
| commit对象 | 记录了提交信息                                              | **$** git cat-file -p 27edfec26df5483fc78e009e446fc063891989e3<br />tree c3b8bb102afeca86037d5b5dd89ceeb0090eae9d<br />author yulu <2723946289@qq.com> 1623757703 +0800<br />committer yulu <2723946289@qq.com> 1623757703 +0800<br /><br />first commit |



代码小结：

```shell
##############################
# 数据对象
##############################

# 1. 创建gitTest目录 并 切换到次目录
# 2. git init初始化一个git项目
# 3. tree命令查看.git目录
$ tree .git/
.git/
├── HEAD
├── config
├── description
├── hooks
│   ├── ...........
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

8 directories, 17 files

# 4. 通过git hash-object 创建 数据对象blob
$ echo 'version 1' > test.txt # 先创建一个文件（相当于我们在工作区创建了一个文件）
$ git hash-object -w test.txt	# -w 选项表示 将对象写入git的对象数据库
83baae61804e65cc73a7201a7252750c76066a30  # 对象的SHA-1编码：前两位 83 代表对象存储的实际目录
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30	# git cat-file 查看实际对象数据库中的对象，-p 显示内容
version 1

# 接下来，我们往工作区总的test.txt中添加一段内容
$ echo 'version 2' > test.txt 
# 5. 再通过git hash-object将更改后的文件存储到对象数据库中
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

# 我们来看看底层是如何存储对象的
.git/
├── HEAD
├── config
├── description
├── hooks
│   ├── .....................
├── info
│   └── exclude
├── objects
│   ├── 1f
│   │   └── 7a7a472abf3dd9643fd615f6da379c4acb3e3a  # 对应于我们更改后的第二个版本的test.txt文件
│   ├── 83
│   │   └── baae61804e65cc73a7201a7252750c76066a30	# 对应于我们更改后的第一个版本的test.txt文件
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

10 directories, 19 files
# 通过上述存储在对象数据库中的信息，可以随时恢复工作区中的内容
# 6. 例如：我们将工作区中的第二版的test.txt删除，然后恢复至第一个版本
$ rm test.txt 
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt   
version 1    # 恢复至第一个版本

# cat-file -t 选项可以查看对象的类型
git cat-file -t 83baae61804e65cc73a7201a7252750c76066a30
blob		# blob 数据对象

##############################
# 树对象： 能够解决文件名保存的问题 # 
# 通常情况下，Git根据某一时刻暂存区所表示的状态创建并记录一个对象的树对象；换句话说，创建一个树对象，我们首先要暂存一些文件（对象）
##############################

# 暂存对象数据库中的对象，通过 git update-index命令来实现
 --add                 不忽略新的文件
--cacheinfo <存取模式>,<对象>,<路径>
                          添加指定的条目到索引区
$ git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt # 将第一个版本的数据添加到暂存区test.txt文件中
# 通过status来验证暂存区中的内容
$ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
	新文件：   test.txt	# 暂存区中新增了test.txt文件
# 7. 调用git write-tree 命令，自动将暂存区中的内容写入一个树对象中
$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
100644 blob 83baae61804e65cc73a7201a7252750c76066a30	test.txt  # 树对象内容的基本格式
# 查看一下这个对象的git对象类型
$ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579
tree
# 查看objects目录
$ tree .git/
.git/
├── HEAD
├── config
├── description
├── hooks
│   ├── ............................
├── index
├── info
│   └── exclude
├── objects
│   ├── 1f
│   │   └── 7a7a472abf3dd9643fd615f6da379c4acb3e3a
│   ├── 83
│   │   └── baae61804e65cc73a7201a7252750c76066a30 
│   ├── d8
│   │   └── 329fc1cc938780ffdd9f94e0d364e0ea74f579 # 树对象跟blob对象都是以相同的形式存储在objects中
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

11 directories, 21 files
# 接下来创建一个新的树对象，它包括 test.txt 文件的第二个版本，以及一个新的文件：
$ echo 'new file' > new.txt
$ git update-index --add --cacheinfo 100644 \ 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
$ git update-index --add new.txt
# 查看git的当前状态
$ git status   # 包括一个修改的test.txt和一个新文件new.txt
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
	新文件：   new.txt
	新文件：   test.txt

尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
	修改：     test.txt
# 8.创建新的tree对象，这个对象中包含两个暂存区中的文件
$ git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
# 查看它的内容
$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341			# 一个tree对象包含了两个blob对象
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt

# git read-tree 是write-tree的反向操作，可将一个树对象读取到暂存区中
 --prefix <子目录>/    读取树对象到索引的 <子目录>/ 下
# 9. 通过read-tree命令将第一个版本test.txt对应的树对象d8329fc1cc938780ffdd9f94e0d364e0ea74f579 读取到暂存区bak目录下
$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
	新文件：   bak/test.txt
	新文件：   new.txt
	新文件：   test.txt
# 此时暂存区中有三个文件，其中test.txt一版的存储在bak目录下
# 10.再创建树对象
$ git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614
# 查看这个树对象的内容
$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579	bak  		# 如果存在目录，那么就会再创建一个树对象
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt



 
```

代码中第10步的树对象3c4e9cd789d88d8d89c1073707c3585e41b0e614对应的结构示意图

<img src="../../../../Library/Application%20Support/typora-user-images/image-20210615213930961.png" alt="image-20210615213930961" style="zoom:50%;" />

```shell
##########################
# 虽然此时，底层已经通过树对象把对象保存起来了，但是是不是感觉缺了点什么？
# 正常我们提交一次暂存区中的内容，不只是把对象存储起来而已，我们还会附带一些提交信息，这些提交信息Git通过提交对象进行存储。
# 提交对象commit
##########################
# 创建提交对象commit，可以通过 git commit-tree SHA-1值进行创建
$ echo 'first commit' | git commit-tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
f5f0cda7464b026ddb429129224cd58138555bfb
$ git cat-file -p f5f0cda7464b026ddb429129224cd58138555bfb
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author yulu <2723946289@qq.com> 1623930083 +0800
committer yulu <2723946289@qq.com> 1623930083 +0800

first commit
# 第二次提交， -p 选项指定父提交
$ echo 'second commit' | git commit-tree 0155eb -p f5f0cda7464b026ddb429129224cd58138555bfb
cbeda08bbfc505e2f029fe7bdd4e32af8f60a33c
# 查看第二次提交的对象
$ git cat-file -p cbeda08bbfc505e2f029fe7bdd4e32af8f60a33c
tree 0155eb4229851634a0f03eb265b69f5a2d56f341
parent f5f0cda7464b026ddb429129224cd58138555bfb
author yulu <2723946289@qq.com> 1623930720 +0800
committer yulu <2723946289@qq.com> 1623930720 +0800

second commit
# 第三次提交的对象
$ echo 'third commit'  | git commit-tree 3c4e9c -p cbeda0
c759b5a289338e9d55333ff7900fdc4ee0526c33

### ###
#通过上述操作，已经创建了3个commit对象
### ###
```









































































































