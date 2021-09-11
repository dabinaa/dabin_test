## Git 使用总结以及原理梳理



### 	版本控制系统（VCS）

- ##### 中央式版本控制系统

  -   保存版本历史
  -   同步团队代码

- ##### 分布式版本控制系统（DVCS）

二者的差别就是分布式系统通常都会多一个本地仓库，分布式将保存版本历史的功能移交到本地仓库实现，

#### DVCS的优缺点

##### 	优点：

- 不受网络的限制，大多数操作都可以在本地实现，减少了开发者的网络限制以及物理位置的限制
- 可以分布提交，代码提交做的更细致

#####    缺点：

- 初次拉取项目的时候比较耗时，因为本地也需要维护一个完整的仓库
- 本地的占用的存储比中央式高（个人理解就是针对历史的维护，）

  

```
git clone 项目的远程仓库地址
	其中会多出一个.git 目录，这个目录就是你的本地仓库。你的所有版本信息都会存在这里。而 .git 所在的这个根目录，称为 Git 的工作目录（Working Directory），它保存了你当前从仓库中签出（checkout）的内容。
```



```
git status 查看当前工作目录当前状态
如下图：
这段文字表述了很多项信息：

你在 master branch
当前 branch 没有落后于 origin/master
你有 untracked files （未追踪的文件），文件名是 shopping list.txt。
你可以使用 git add 来开始追踪文件。
```

![image-20210111133356955](C:\Users\dabin\AppData\Roaming\Typora\typora-user-images\image-20210111133356955.png)



##### Git 的管理是目录级别，而不是设备级别，所以可以一台电脑维护多个版本



#### 多人开发，可能存在的提交问题

​	当一个人先于另一个人 `push` 代码（这种情况必然会发生），那么后 `push` 的这个人就会由于中央仓库上含有本地没有的提交而导致 `push` 失败。

```
为什么会提交失败？
	因为Git的push其实就是使用本地的commits去覆盖远端仓库的commits记录（简化的概念）。如果远端仓库含有本地没有的commits的时候，push如果成功的话会导致远端的commits被清除，这种结果是不可存在的，所以现在push的时候hi去进行检查，如果存在如上情况的话，push会失败。
```



## HEAD、master和branch；（引用的本质其实就是一个个字符串）

#### HEAD：当前commit的引用。

​	可以理解为：当前commit在哪里，HEAD就在哪里。这是一个永远自动指向当前commit的引用。所以可以使用HEAD来操作当前commit。

#### Branch：

​		Git除了HEAD（唯一）这个引用之外还存在一种引用，branch。HEAD通过指向branch，会通过这个branch 来间接的指向某个commit。当HEAD提交的时候，他会带着他指向的branch 一起移动。

```
branch的通俗化的理解：
	虽然branch 只是一个指向commit的引用，但是可以将他理解为从初始化commit 到branch所指向的commit 之间的所有commits的一个串。

注意点：
	1、所有的branch 都是平等的。（即master除了一下的特点与其他的branch 都没有什么区别）
	2、branch包含了从初始化commit到它的所有路径。而不是一条路径。并且，这些路径之间也是彼此平等的。
```

#### master（main）：默认的branch。

- 新建的仓库（repository）是没有任何的commit的，但是它创建第一个commit 的时候，会把master指向它，并把HEAD指向master。
- 当有人使用git clone的时候，除了从远程仓库将.git 这个仓库目录下载到工作目录，还会将checkout master （`checkout` 的意思就是把某个 `commit` 作为当前 `commit`，把 `HEAD` 移动过去，并把工作目录的文件内容替换成这个 `commit` 所对应的内容）。
- 出于安全考虑，没有合并到master的分支在删除的时候失败（git branch -d 分支名）。如果确定要删除的时候执行（git branch -D 分支名）



#### branch的创建、切换以及删除注意点：

- git branch 新分支  的时候，HEAD不会自动只想到新分支，可以使用 git branch -b 新分支。又或者在执行 git checkout 新分支。
- 删除的时候需要注意：
  - HEAD指向的branch 不能删除，如果删除HEAD指向的branch ，需要先checkout到其他的分支。
  - 由于git中branch 只是一个引用，所以删除branch的时候只是删除这个引用，并不会删除这个commit。（不过如果一个 `commit` 不在任何一个 `branch` 的「路径」上，或者换句话说，如果没有任何一个 `branch` 可以回溯到这条 `commit`（也许可以称为野生 `commit`？），那么在一定时间后，它会被 Git 的回收机制删除掉。）

###### 远程仓库的HEAD只会指向的默认的分支，并且随着默认分支的移动而移动。并不会把本地的HEAD的指向一起上传到远程仓库，





















