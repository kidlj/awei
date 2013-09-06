---
layout: post
title: Progit读书笔记
---

Progit读书笔记
=============


第1章：起步
------------

### Git基础要点 ###

**设计哲学**

* 传统VCS系统保存项目前后不同版本各个文件间的Patch, 然后根据这些Patch计算出
  各个版本的文件内容。
  Git并不保存这些前后变化的差异数据(patch), 而是直接对变化后的文件做
  快照，并保存一个指向这次快照的索引。
* Git的绝大数操作都只需要访问本地文件和资源，而CVS差不多每次操作都需要
  连接网络。
* 如果要查看当前版本的文件和一个月前的版本之间有何差异，Git会取出一个月
  前的快照和当前文件做一次差异运算。
* 时刻保存数据完整性。每次保存之前，Git都会对所有数据进行内容的校验和(checksum)
  计算，并将结果(指纹字符串)作为数据的唯一标识和索引。换句话说，不可能在修改了目录
  或文件内容后，Git一无所知。这项特性作为Git的设计哲学，建在整体架构的最底层。

  

**三种状态**

对于任何一个文件，在Git内部都只有三种状态，已修改(modified), 已暂存(staged)
和已提交(commited)。

从项目中取出某个版本的所有文件和目录，用以开始后续工作的叫做工作目录。
所谓暂存区不过是个简单的文件，含有下一次提交将要发生的内容的信息。

基本的Git工作流程如下所示：

1. 修改：在工作目录中修改某些文件;
2. 暂存：对这些修改了的文件做快照,并将其保存到暂存区域;
3. 提交：提交更新，将保存在暂存区的文件快照永久储存到Git目录中。

**配置Git**

	$ git config --system user.name		`/etc/gitconfig`
	$ git config --global user.name		`~/.gitconfig`
	$ git config ???	  user.name		`.git/config`

**获取帮助**

	$ git help <command>
	$ git <command> --help
	$ man git-<command>


第2章：Git基础
--------------

### 2.1 取得项目的Git仓库 ###

**在项目目录初始化**

	$ cd ~/Notes
	$ git init

**从现有项目克隆**

	$ git clone git://url/to/.git [mygitrepo_name]

现在我们已经有了一个真实项目的Git仓库，并且从仓库中取出了最新版本
的文件拷贝。

工作目录下的所有文件不外乎这两种状态：已跟踪或未跟踪。初次克隆某个
仓库时，工作目录所有的文件都属于已跟踪文件，而且状态属于未更改。而
在一个已存在目录内初始化Git仓库后，工作目录所有文件都属于未跟踪。

### 2.2 记录每次更新到仓库 ###

**检查当前文件状态**
	
	$ git status

**跟踪新文件**

	$ mv ~/Progit.md Noets/
	$ git add Progit.md

**暂存已修改文件**

	$ git add Progit.md

(`git add`是个多功能命令，根据目标文件状态的不同，此命令的效果也不同：
可以用它开始跟踪新文件，或者把已跟踪并修改的文件放入暂存区，还能用于
合并时把有冲突的文件标记为已解决状态等。)

**忽略某些文件**

	$ cat .gitignore

	# 该行为注释
	*.swp

**查看已暂存和未暂存的更新**

查看哪些更新还没有暂存：

	$ git diff

查看哪些更新已经暂存起来等待下次提交：

	$ git diff --staged

或者
	
	$ git diff --cached


**提交更新**

	$ git commit -m "the very first commit"

**跳过使用暂存区域**

尽管使用暂存区域可以精心地准备要提交的细节，可有时候略显繁琐。
在提交的时候，给`git commit`加上`-a`选项，Git就会把所有已跟踪并
更改了的文件暂存起来一并提交，从而跳过`git add`步骤。

	$ git commit -a 

**移除文件**

1. 工作目录先删除一个文件，提交这一变化至Git仓库，流程跟添加一个
新文件是一致的。

		$ rm README		#删除目录上的文件(对应目录内创建新文件)
		$ git rm README		#解除跟踪(对应`git add`的添加跟踪)
		$ git commit  	#提交“解除跟踪”

2. 如果一个新跟踪文件，`git rm README`:

	>error: README has changes staged in the index  
	>(use --cached to keep the file, or -f to force removal)

		$ git rm --cached README
		
		$ git rm -f README

3. 如果一个文件已在Git仓库里被跟踪且未修改:

		$ git rm README
		$ git commit 

		$ git rm --cached README
		$ git commit

4. 如果一个文件已在Git仓库里被跟踪，已修改但未暂存，`git rm README`：
	
	>error: README has local modifications  
	>(use --cached to keep the file, or -f to force removal)

		$ git rm --cached README
		$ git commit

		$ git rm -f README
		$ git commit

5. 如果一个文件已在Git仓库里被跟踪，已修改也已暂存，`git rm README`:

	>error: README has changes staged in the index  
	>(use --cached to keep the file, or -f to force removal)

		$ git rm --cached README
		$ git commit

		$ git rm -f README
		$ git commit

**移动文件**

给被Git跟踪的某个文件重命名：
	
	$ git mv README.txt README
	$ git commit

实际上相当于运行了下面三个命令：
	
	$ mv README.txt README
	$ git rm README.txt
	$ git add README
	$ git commit

### 2.3 查看提交历史 ###

	$ git log [arguments]

	$ git log -<n>
	$ git log --pretty=[arg]
	$ git log --pretty=format
	$ git log -p
	$ git log --author --commiter --all-match
	$ git log --since="date" --before="date"  --<file>

### 2.4 撤销操作 ###

**修改最近一次提交**

	$ git commit --amend

此命令将使用当前的暂存快照提交，并且有机会修改提交说明。

如果上次提交时忘了暂存某些修改，可以先不上暂存操作，然后再`--amend`提交：

	$ git commmit -m 'yet another commit'
	$ git add forgotten_file
	$ git commit --amend

**取消已经暂存的文件**

	$ git reset HEAD <file>

**取消对文件的修改**

	$ git checkout -- <file> 

这条命令有些危险，所有对文件`file`刚才的修改都没了。如果只是想要回退版本，同时
保留刚才的修改以便将来继续工作，可以使用下章介绍的stashing和分支来处理。

记住，任何提交到Git的数据，都可以被恢复。可能失去的数据仅限于没有提交果的，
对Git来说它们就像从未存在过一样。

### 2.5 远程仓库的使用 ###

**查看当前的远程库**
		
	$ git remote [-v]

在克隆完某个项目后，至少可以看到一个名为origin的远程库，Git默认使用这个名字
来标识你所克隆的原始仓库。

**添加远程仓库**
	
	$ git remote add [remote-name] [url]

**从远程仓库抓取数据**

	$ git fetch [remote-name]

该命令回到远程仓库中拉取所有你本地仓库中还没有的数据。运行完成后，你就可以在
本地访问该远程仓库中的所有分支，将其中某个分支合并到本地，或者只是取出某个分支
一探究竟。

`fetch`命令只是将远端的数据拉到本地仓库，并不自动合并到当前分支，当准备好的时候
可以手动合并。而`pull`命令则将远端分支自动合并到工作目录中当前分支。

**推送数据到远程仓库**

	$ git push [remote-name] [branch-name]

如果要把本地的`master`分支推送到`origin`服务器上(`git clone`操作会自动使用
`master`和`origin`名字), 可以运行下面这个命令：

	$ git push origin master

只有在克隆的服务器上有写权限，或者同一时刻没有人在推数据，这条命令才可如期
完成任务。如果你在推数据前，已有人推送了若干更新，那你的推送操作就会被驳回。
你必须先把它们的更新抓取到本地，合并到自己的项目中，然后才可以再次推送。

**查看远程仓库信息**

我们可以通过命令
	
	$ git remote show [remote-name]

来查看某个仓库的远程信息(需要连接网络)。比如要查看所克隆的远程仓库，可以运行：

	$ git remote show origin

**远程仓库的删除和重命名**

	$ git remote rename pb paul

	$ git remote rm paul

### 2.6 打标签 ###

**列显已有的标签**

	$ git tag

**新建标签**

Git使用的标签有两种类型：轻量级的(lightweight)和含附注的(annotated). 轻量级
标签就是个指向某次特定提交的引用(指针);而含附注标签则是储存在仓库中的一个独立对象，
它有自身的校验和信息，包含着打标签人的名字，电子邮件和打标签日期，以及标签说明，
标签本身也允许使用GNU Privacy Guard(GPG)来签署和验证。

**含附注的标签**

	$ git tag -a v0.1 -m 'my version 0.1'

	$ git show v0.1

	$ git tag -s v0.2 -m 'my signed version 0.2'

**轻量级标签**

	$ git tag v0.1-lw
	$ git show v0.1-lw

**验证标签**

	$ git tag -v [tag-name]

此命令会条用GPG来验证签名，所以你需要有签署者的公钥，存放在keyring中才能验证。

**后期加注标签**
	
可以在后期对早先的某次提交加注：

	$ git log pretty=oneline

	$ git tag -a v1.2 9fceb02

	$ git tag

	$ git show v1.2

**分享标签**

默认情况下，`git push`并不会把标签传送到远程服务器上，只有通过显示命令才能分享
标签到远端仓库。其命令格式如同推送分支：

	$ git push origin [tag-name]

如果要一次推送所有(本地新增的)标签上去，可以使用`--tags`选项：

	$ git push origin --tags

现在，其他人克隆共享仓库或拉取数据同步后，也会看到这些标签。

### 2.7 技巧和窍门 ###

**自动补全**

	$ cp git-src/contrib/completion/git-completion.bash ~/.git-completion.bash

	$ echo "source ~/.git-completion.bash" >> ~/.bashrc

或者系统级别启用：

	$ cp git-src/contrib/completion/git-completion.bash /etc/bash_completion.d/
	
**Git别名**

	$ git config --global alias.co checkout
	$ git config --global alias.br branch
	$ git config --global alias.ci commit
	$ git config --global alias.st status
	$ git config --global alias.unstage 'reset HEAD --'
	$ git config --global alias.last 'log -1 HEAD'
	$ git config --global alias.visual "!gitk"


第3章：Git分支
--------------

### 3.1 何谓分支 ###

几乎每一种版本控制系统都以某种形式支持分支。使用分支意味着你可以从开发主线上分离
开来，然后在不影响主线的同时继续开展自己的工作。

在Git提交时，会在Git仓库里保存三种对象：

* blob对象 - 分别保存暂存区各个文件内容;
* tree对象 - 保存暂存区各个文件与blob对象的对应索引;
* commit对象 - 保存父`commit对象`索引，`tree对象`索引和提交信息元数据。

假设暂存区有三个文件，那么提交后仓库里会多出5个对象(三个blob对象，一个tree对象和
一个commit对象). 如果做些修改后再次提交，那么这次的commit对象里会包含一个指向
上次commit对象的指针。

**Git分支**

Git中的分支，其实本质上仅仅是个指向某个`commit对象`的可移动指针。Git会使用master
作为分支的缺省名字。在若干次提交后，你其实已经有了一个指向最后一次`commit对象`的
master分支，它在每次提交的时候都会自动向前移动。

**创建分支**

那么在Git里，创建一个新的分支就是创建一个新的分支指针。比如创建一个`testing`分支：

	$ git branch testing

这会在当前`commit对象`上新建一个分支指针(此时共有`master`和`testing`两个分支指针
指着当前`commit对象`)。

**切换分支**

Git保存着一个名为HEAD的特别指针，它指向那个分支指针，那么我们就在哪个分支上。
要切换Git分支：

	$ git checkout testing

这样HEAD就指向了`testing`分支。

现在我们可以在`testing`分支任意提交数据，而`master`分支仍然指着切换分支时所指着的
那个`commit对象`。

我们随时可以用

	$ git checkout master

命令回到`master分支`，这条命令做了两件事。它把HEAD指针移回到`master`分支，并把
工作目录中的文件换成了`master`分支所指向的快照内容。

我们可以在不同分支里来回切换，各自开发，所做的改变都孤立在各自的分支里，可以在
时机成熟的时候将这些改变合并到一起。

由于Git中的分支仅仅是一个包含所指`commit对象`校验和(40个字符长度SHA-1字串)的文件，
所以创建和销毁一个分支就变得非常廉价。

同时，因为每次提交都记录了父`commit对象`的信息，所以以后要合并分支时，寻找合适的
合并基础(`merge base`, 即相同的父`commit对象`)的工作已经自动完成。

### 3.2 基本的分支与合并 ###

**应用场景举例**

应用到简单的分支与合并工作流的例子：

1. 开发一个网站。
2. 为实现某个新的需求，创建一个分支。
3. 在这个新分支上开展工作。

假设此时，突然接到一个电话说有一个很严重的问题需要修补，那么可按照下面的方式处理：

1. 返回到原先已经发布到生产服务器上的分支。
2. 为这次紧急修补建立一个分支。
3. 测试通过后，将此修补分支合并，再推送到生产服务器山。
4. 切换到之前实现新需求的分支，继续工作。

**基本分支**

现在，为实现这个新的需求，创建一个分支`new_feature`:

	$ git checkout -b new_feature

相当于

	$ git branch new_feature
	$ git checkout new_feature

正在此分支上工作，突然电话打来需要修复生产服务器上某个问题，回到master分支：

	$ git checkout master

(在转换分支之前，留心暂存区或工作目录里那些还没提交的修改，它会和你即将检出的分支
产生冲突而阻止Git为你转换分支). 转换分支的时候最好保持一个清洁的工作目录。稍后会
介绍几个绕过这种问题的方法(分别叫做`stashing`和`amending`).

切换分支后，有一点值得牢记：Git会把工作目录的内容恢复成检出某分支时它所指向的那个
commit快照。它会自动添加、删除和修改文件以确保目录的内容和上次提交时一模一样。

接下来我们创建一个紧急修补分支来开展工作，直到搞定：

	$ git checkout -b hotfix

有必要做些测试，确保修补是成功的，然后把它合并到master分支并发布到生产服务器。

	$ git checkout master
	$ git merge hotfix		# Fast forward

现在`master`分支和`hotfix`分支指向相同的`commit对象`，现在可以删掉`hotfix`分支：

	$ git branch -d hotfix

现在可以回到`new_feature`分支上继续工作：

	$ git checkout new_feature

现在两个分支的情况是这样的：

				       (master)
					 	  |
						  |
	[c0]<--[c1]<--[c2]<--[c4]
					\
					 *<--[c3]<--[c5]
					  			 |
								 |
							(new_feature)

不要担心hotfix的内容还没包含在`new_feature`中。如果确实需要纳入此次修补，可以用
`git merge master`把`master`分支合并到`new_feature`，或者等完成后，将`new_feature`
分支中的更新并入`master`。
					 
**基本合并**

在`new_feature`的工作完成之后，可以合并回master分支：

	$ git checkout master
	$ git merge new_feature

现在分支情况是这样的：

				       				   (master)
					 	  				  |
					 					  |
									 (snapshot to 
			(common ancestor)		  merge into)
					|				      |
	[c0]<--[c1]<--[c2]<--[c4]<----------[c6]
					\				    /	
					 *<--[c3]<--[c5]<--*
					 			 |
							(snapshot to
							  merge in)
					  			 |
								 |
							(new_feature)

这次合并的实现，并不同于之前hotfix的并入方式。这一次，你的开发历史是从更早的地方
开始分叉的。由于当前`master`分支所指向的`commit[c4]`并非想要并入分支`new_feature`
的直接祖先，因此Git不得不进行一些处理。此时，Git会用两个分支的末端([c4]和[c5])和
它们的共同祖先[c2]进行一次简单的三方合并计算。

Git没有简单地把分支指针右移，而是对三方合并的结果作一新的快照，并自动创建一个指向
它的commit([c6])。我们把这个特殊的commit称作合并提交(merge commit), 因为它的祖先
不止一个。

值得一提的是Git可以自己裁决哪个共同祖先才是最佳合并基础，所以此特性让Git的合并操作
比其它系统要简单不少。

既然工作结果已经合并了，`new_feature`分支也就没有用了。删除之：

	$ git branch -d new_feature


**冲突的合并**

有时候合并操作并不会如此顺利。如果你修改了两个待合并分支里同一个文件的同一个部分，
Git就无法干净地把两者合到一起。如果你在`new_feature`的开发过程中修改了hotfix中修
改的部分，合并将产生冲突，只能由人手动merge那个文件。

可以用`git status`查看哪些文件合并发生了冲突。任何包含未解决冲突的文件都会以未合
并(`unmerged`)状态列出。

	$ git status

此时可以选取合并工具来合并冲突文件：

	$ git mergetool

在解决了所有文件的冲突以后，将它们标记为已解决：

	$ git add <the_file>

最后提交：

	$ git commit -m 'the conflicts merge information'

提交信息应该含有冲突合并细节。


### 3.3 分支管理 ###


列出所有分支，带`*`的为当前所在分支:

	$ git branch [-v]

查看哪些分支已经(或尚未)与当前分支合并：

	$ git branch --merged
	$ git branch --no-merged

合并过的分支可以用

	$ git branch -d <branch-name>

删除掉。未与当前分支合并的分支不可以用`-d`删除：

	$ git branch -d testing

> error: The branch 'testing' is not an ancestor of your current HEAD.


### 3.4 分支式工作流程 ###

**长期分支**

	(master)[c1]---------------------------------->
		\
	 (develop)[c2]<---[c3]-------------------->
					    \
					 (topic)[c4]<---[c5]------>

可以用这种方式维护不同层次的稳定性。

**特性分支**



											<--[c12]<--[c13] (dumbidea)
										   /
	(master)[c0]<--[c1]<--[c3]<--[c9]<--[c10]
					 \
					  *<--[c2]<--[c4]<--[c5]<--[c6] (iss91)
					           	   \
								    *<--[c7]<--[c8]<--[c11] (iss91v2) 

提交到[c1]的时候，从`master`分出`iss91`分支来解决#91号问题。在该分支提交到[c6]
时有了一个更好的修补方法，便从[c4]开始另分出一个分支`iss91v2`来解决#91号问题。
同时在`master`分支提交到[c10]的时候，项目有了一个新想法，便分出分支`dumbidea`来
支持该想法。到最后，发现`dumbidea`和`iss91v2`都非常好，便舍弃`iss91`分支的[c5]
和[c6]，将这两个完成的分支并入到`master`分支。

合并以后提交历史变成这样：


											  (dumbidea)
												  |
	[c0]<--[c1]<--[c3]<--[c9]<--[c10]<--[c12]<--[c13]<--[c14] (master)
			\											  /
			 *<--[c2]<--[c4]<--[c7]<--[c8]<--[c11]<------*
			 								   |
											(iss91v2)
											
### 3.5 远程分支 ###

远程分支(remote branch)是对远程仓库状态的索引。它们是一些无法移动的本地分支，只
有在进行Git的网络活动时才会更新。远程分支就相当于书签，提醒着你上次连接远程仓库时
上边各个分支的位置。

我们用“远程仓库名/分支名”这样的形式表示远程分支。

一次Git克隆会建立你自己的本地分支`master`和远程分支`origin/master`，它们都指向
`orgin/master`分支的最后一次提交。

		     (origin/master)  <== remote branch
			 		|
	[c1]<--[c2]<--[c3]
			 		|
				(master)	  <== local branch

要是你在本地`master`分支做了会儿事情，与此同时，其他人向该例中的远程仓库推送了
内容，更新了上面的`master`分支，那么你的提交历史会开始朝不同方向发展。不过只要你
不和服务器通讯，你的`origin/master`指针不会移动。

可以运行`git fetch orgin`命令来进行同步。该命令首先找到`origin`是哪个服务器，从
上面获取你尚未拥有的数据，更新你本地的数据库，然后把`origin/master`的指针移动到
它最新的位置。

**推送**

要想和其它人分享某个分支，你需要把它推送到一个你拥有写权限的远程仓库。你的本地
分支不会被自动同步到你引入的远程分支中，除非你明确执行推送操作。换句话说，对于
无意分享的，你尽可以保留为私人分支，而只推送哪些协同工作的特性分支。

如果你有个叫`serverfix`的分支需要与别人协同开发，可以运行：

	$ git push origin serverfix

这其实是简便写法。Git自动把`serverfix`分支名扩展为`refs/heads/serverfix:refs/
heads/serverfix`，意味取出我的`serverfix`本地分支，推送它来更新远程仓库的`serverfix`
分支。(以后将介绍`/refs/heads/`细节)，不过一般使用的时候可以省略它。

也可以这样写：

	$ git push origin serverfix:serverfix

还可以给远程分支起个跟本地分支不同的名字：

	$ git push origin serverfix:awesomebranch

接下来，当你的协作者再次从服务器上获取数据时，他们将得到一个新的远程分支`origin/
serverfix`.

	$ git fetch orgin

> ...  
> From git@github.com:kdlijian/...  
>  [new branch]		serverfix  ->  origin/serverfix

值得注意的是，在`fetch`操作(按：不是`clone`)抓来某些远程分支之后，你得不到相应的
可编辑的本地分支。比如，在本例中，你的合作者`fetch`到了一个新远程分支`serverfix`，
他们并不会得到一个本地分支`serverfix`，有的只是一个无法移动的`original/serverfix`
分支指针。

如果要把该内容合并到当前分支，可以运行:

	$ git merge origin/serverfix

如果他们想要一份自己的`serverfix`来开发，可以在远程分支的基础上分化出一个新的分支来：

	$ git checkout -b serverfix origin/serverfix

这会切换到新建的`serverfix`本地分支，其内容同远程分支`origin/serverfix`一致，你可以
在里面继续开发了。

**跟踪分支**

从远程分支检出(分化出，见上例)的本地分支叫做跟踪分支(tracking brach)。跟踪分支是一
种和远程分支有直接联系的本地分支。在跟踪分支里输入`git push`, Git会自行判断向哪个
服务器的哪个分支推送数据;反过来，在这些分支里运行`git pull`会获取所有远程索引，并
把他们的数据都合并到本地分支中来。

在克隆仓库时，Git通常会自动建立一个`master`分支来跟踪`origin/master`。这正是`git 
push`和`git pull`一开始就能正常工作的原因。当然，也可以设定其它跟踪分支：

	$ git checkout -b [分支名] [远程仓库名]/[分支名]

或者

	$ git checkout --track origin/serverfix

可以改变名字：

	$ git checkout -b sf origin/serverfix

现在你的本地分支`sf`会自动向`origin/serverfix`抓取和推送数据了。

**删除远程分支**

如果不再需要某个远程分支了，比如搞定了某个特性并把它合并到了远程的`master`分支里，
可以用这个无厘头的语法来删除它：

	$ git push origin :serverfix

意味“在这里提取空白然后把它变成远程分支”，即删除远程分支。


### 3.6 衍合 ###

把一个分支整合到另一个分支的办法有两种，`merge(合并)`和`rebase(衍合)`。

**衍合基础** 

						  
					 <--[c3] (experiment)
					/
	[c0]<--[c1]<--[c2]<--[c4] (master)

之前介绍过，最容易的整合分支的方法是`merge`命令，它会把两个分支最新的快照([c3]和
[c4])以及二者共同的祖先([c2])进行三方合并。合并的结果作一快照提交为[c5]:

					 <--[c3]<---
					/			\
	[c0]<--[c1]<--[c2]<--[c4]<---[c5]

其实，还有另外一个选择：你可以把在[c3]里产生的变化补丁重新在[c4]的基础上打一遍。
在Git里，这种操作叫做衍合(rebasing)。有了`rebase`命令，就可以把在一个分支里提交的
改变在另一个分支里重演一遍。

在这个例子里，可以运行下面的命令：

	$ git checkout experiment
	$ git rebase master

它的原理是回到两个分支(你所在分支`experiement`和你想要衍合入的分支`master`)的共同
祖先([c2]),提取你所在分支(`experiment`)每次提交时产生的差异(diff),把这些差异分别保存
到临时文件里，然后从当前分支转换到你需要衍合入的分支(`master`)，(在[c4]基础上)依序
施用每一个差异补丁文件。

结果下图所示：

							 (experiment)
								  |
								  |
	[c0]<--[c1]<--[c2]<--[c4]<--[c3']
						  |
						  |
					   (master)


现在，回到`master`分支然后进行一次快进合并。

	$ git checkout master
	$ git merge experiment

							 (experiment)
								  |
								  |
	[c0]<--[c1]<--[c2]<--[c4]<--[c3']
								  |
								  |
							   (master)

合并以后，[c3']所指的快照，同三方合并例子中的[c5]所指的快照内容一模一样了。最后
整合得到的结果没有任何区别，但衍合能产生一个更为整洁的提交历史。如果视察一个衍合
过的分支的历史记录，看起来更清楚：仿佛所有修改都是先后进行的，尽管实际上他们原来
是同时发生的。

(按：总的来说，衍合就是把本不能快进的合并变成快进合并。请在有需要用的时候再使用。)

**更多有趣的衍合**


	[c1]<--[c2]<--[c5]<--[c6]	(master)
			 \
			  *<--[c3]<--[c4]<--[c10]	(server)
			  		\
					 *<--[c8]<--[c9]	(client)

(从一个特性分支里再分出一个特性分支的历史)

如图，你创建了一个特性分支`server`来给服务器端代码添加一些功能，然后提交c3和c4。
然后从c3的地方再增加一个`client`分支来对客户端代码进行一些修改，提交c8和c9。最后
又回到`server`分支提交了c10。

假设在接下来的一次软件发布中，你决定把客户端的修改先合并到主线中，而暂缓并入服务器
端软件的修改(因为还需要进一步测试)。你可以仅提取对客户端的改变(c8和c9)，然后通过
`git rebase`的`--onto`选项来把它们在`master`分支上重演。

	$ git rebase --onto master server client

这基本上是说“检出`client`分支，找出`client`分支和`server`分支的共同祖先之后的变化
(即c8和c9)，然后把它们在`master`上重演一遍。

现在可以快进`master`分支了：

	$ git checkout master
	$ git merge client

结果如下图所示：

	[c1]<--[c2]<--[c5]<--[c6]<--[c8']<--[c9']	(master)
		     \
			  *<--[c3]<--[c4]<--[c10]	(server)

现在你决定把`server`分支的变化也包含进来。可以直接把`server`分支衍合到`master`而
不用手动转到`server`分支再衍合。`git rebase [主分支] [特性分支]`命令会先检出特性
分支`server`，然后在主分支`master`上重演：

	$ git rebase master server

于是`server`的进度应用到`master`的基础上，而后快进主分支`master`：

	$ git checkout master
	$ git merge server

现在`client`和`server`分支的变化都被整合了，不妨删掉它们:

	$ git branch -d client
	$ git branch -d server

把提交历史变成如下图的样子：

															   (server)
																  |
																  |
	[c1]<--[c2]<--[c5]<--[c6]<--[c8']<--[c9']<--[c3']<--[c4']<--[c10'] 
										  |						  |
										  |						  |
									   (client)				   (master)

**衍合的风险**

一句话可以总结：

> 永远不要衍合那些已经推送到公共仓库的更新。  
> Do not rebase commits that you have pushed to a public repository.

(其实就是说一定要在分享之前衍合，分享出去之后就不要再衍合.)

在衍合的时候，实际上抛弃了一些现存的commit而创建了一些类似但不同的新commit。如果
你把commit推送到某处然后其他人下载并在其基础上工作，然后你用`git rebase`和`git push
 --force`重写了这些commit再推送一次，你的合作者就不得不重新合并这些内容，尽管之前
已经做过一次了。衍合会改变这些commit的SHA-1值，这样Git会把它们当作新的commit，然而
这时候在他们的提交历史里早就有了那些内容。

如果把衍合当作一种在推送之前清理提交历史的手段，而且仅仅衍合那些永远不会公开的commit，
那就不会有任何问题。

第4章：服务器上的Git
--------------------

Git远程仓库仅仅是一个纯仓库-- 一个没有当前工作目录的仓库。仓库里仅仅是Git的数据，
也即项目里`.git`目录里的内容，别无他物。

### 4.1 协议 ###

Git使用四种主要的协议来传输数据：本地传输，SSH协议，Git协议和HTTP协议。
除了HTTP协议以外，其它协议都要求在服务器端安装并运行Git。

**本地协议**

克隆一个本地仓库：

	$ git clone /path/to/project.git

添加一个本地仓库到现有Git工程：

	$ git remote add local_proj /path/to/project.git

然后就可以像在网络上一样向这个远程仓库推送和获取数据了。

**SSH协议**

克隆一个仓库：

	$ git clone ssh://git@github.com:kdlijian/kdlijian.github.com.git

或者不指明某个协议，这是Git会默认使用SSH：

	$ git clone git@github.com:kdlijian/kdlijian.github.com.git

**Git协议**

匿名访问，通常不能用来推送。传输最快。

**HTTP(s)协议**

	$ cd /var/www/htdocs
	$ git clone --bare /path/to/project/ git-project.git
	$ cd git-project.git
	$ cp hooks/post-update.sample hooks/post-update
	$ chmod a+x hooks/post-update

第5章：分布式Git
----------------

### 5.1 分布式工作流程 ###
### 5.2 为项目做贡献 ###

**私有的小型团队**

工作流(按事件发生顺序)：

1. John克隆了仓库(A)，做了些更新，在本地提交(B)。
		
		$ git clone git@github.com:sample.git

2. Jessica，同样克隆了仓库(A)，在本地提交更新(C)。

		$ git clone git@github.com:sample.git

3. Jessica将她的工作推送到服务器上。

		$ git push origin master

4. John也尝试这么做，但被驳回。他不得不把Jessica的更新拉下来，并与自己
   的`master`分支合并(D)。

		$ git fetch orgin
		$ git merge orgin/master

5. John将合并结果(D)推送到服务器上。

		$ git push origin master

6. Jessica此时已创建特性分支`issue54`并已在其上提交了三次更新(E),(F),(G).

		$ git checkout -b issue54 

7. Jessica与服务器同步，她的本地仓库历史多出了John的两次提交(B)和(D), 
   她想看看这两次提交是什么。

   		$ git fetch orgin
		$ git log --no-merges origin/master ^issue54

8. Jessica将特性分支`issue54`合并到`master`分支。

		$ git checkout master
		$ git merge issue54

9. Jessica将`origin/master`合并到自己的`master`分支。

		$ git merge origin/master

10. Jessica将自己的`master`分支推送到服务器上。

		$ git push origin master
		
John的提交历史如下：

	 1.o/m   5.o/m
	   |       |
	---A---B---D (master)
	    \     /
		 C---*
		 |
	   3.o/m
		 

Jessica的提交历史如下：

				  issue54  10.o/m
					   |   |
	---A---C---E---F---G---H (master)
	    \   \		      /
		 B---D-----------*

注：第8和第9步，谁先谁后都没有关系，最终合并后的内容快照是一样的，而仅仅
是提交历史看起来会有些先后差别。

**私有团队间协作**

集成管理员工作流(Intergration Manager Workflow):

几个小组分头负责若干特性的开发，每个组有一位管理员负责集成本组代码，及更新
项目组仓库的`master`分支。所有开发在都在代表小组的分支上进行。

假设John和Jessica一起负责开发某项特性A，同时Jessica和Josie负责开发另一项
功能B。

Jessica的工作流程如下：

1. 克隆生成本地仓库后(A), 创建分支`featureA`以开发特性A。

		$ git checkout -b featureA

2. 提交更新(B)，因为Jessica没有权限推送数据到仓库的`master`分支(只有集成管理
员有此权限)，所以只能将`featureA`分支推送至服务器，并邮件告知John取用。

		$ git push origin featureA

3. 等待John来信过程中以`origin/master`为基础创建`featureB`分支以开发特性B
(注：与`featureA`分支同基础)，并在其上提交了若干更新(E,F)。

		$ git fetch origin
		$ git checkout -b featureB origin/master

4. Jessica正准备推送`featureB`分支到服务器上，Josie来信说已将自己的工作推送到
服务器上的`featureBee`分支了(G)。于是Jessica下载Josie的代码，合并到自己的分支中，
然后推送到`featureBee`分支上(H)。

		$ git fetch origin
		$ git merge origin/featureBee
		$ git push origin featureB:featureBee

5. John来信告知说他已往`featureA`上推送了另一个更新(C)，Jessica将其下载，查看
了所做修改以后合并到本地`featureA`分支，再此基础上做了些修改，又推回到服务器上(D)。

		$ git fetch orgin
		$ git log origin/featureA ^featureA
		$ git checkout featureA
		$ git merge origin/featureA
		$ git push origin featureA

6. 最后集成管理员将`featureA`和`featureBee`分支并入`master`(I)。

提交历史如下：

	   			 o/fA
				   |
	   m		  fA   o/m
	   |		   |	|
	---A---B---C---D----I
	   |\			   /
	    \*--E---F--H--*
	     \		  /|
		  G------* |
		  		  fB
				   |
				 o/fBee


**公开的小型项目**

上面说的是私有项目合作，但要给公开项目作贡献，情况就有些不同了。因为你没有更新
主仓库分支的权限，得寻求其它方式把工作成果交给项目维护人。有两种方法，一种是使
用Git托管服务商提供的仓库复制(fork)功能，另一种是通过电子邮件寄送文件补丁。

但不管哪种方式，一开始做的总是这些工作：

	$ git clone [url]
	$ cd [projec]
	$ git checkout -b featureA
	$ (work)
	$ git commit
	...

完成特性分支`feature`以后，提交给项目维护者之前，先到原始项目的页面上点击`fork`，
创建一个自己可写的公共仓库，然后把它加为本地的第二个远端仓库。

	$ git add myfork [url]

这个时候，我们直接把`featureA`分支整个推送到该远程仓库：

	$ git push myfork featureA

然后通知项目管理员，来抓取你的代码，这个过程叫做`Pull Request`.

	$ git pull-request origin/master myfork

`git pull-request`命令接受两个参数，第一个参数`origin/master`是本地特征分支开始
前的原始分支，第二个参数`myfork`是请求对方来抓取的Git仓库地址。

输出的内容可以直接邮件发给邮件管理者，他们就会明白这是从哪次提交开始旁支出去的，
该到哪里去抓取新的代码，以及新的代码增加了哪些功能等等。或者直接用Github等提供的
`pull-request`功能自动发出请求通知。

像这样随时保持自己的`master`分支和官方`origin/master`同步，并将自己的工作限制在
特性分支上的做法，既方便又灵活，采纳和丢弃都轻而易举。就算原始主干发生变化，我们
也能重新衍合提供新的补丁。

如果现在要开始第二个特性的开发，不要在原来已推送的特性分支上继续，还是按原始`master`
开始：

	$ git checkout -b featureB origin/master
	$ (work)
	$ git commit
	...
	$ git push myfork featureB
	$ git pull request...
	
现在从原始仓库拉取更新:

	$ git fetch origin

这时提交历史是这样的：

		   m  o/m 
		   |   |
	---A---B---C    fA
		   |\		|
		   | D------E 
		    \
			 F 
		   	 |
			fB

假设项目管理员接纳了许多别人提交的补丁之后，准备要采纳你提交的第一个分支，却
发现因为代码基准不一样(`origin/master`已经前移了)，合并工作无法干净地完成。这就
需要把提交历史衍合到最新的`origin/master`上，然后重新推送分支`featureA`:

	$ git checkout featureA
	$ git rebase origin/master

	$ git push -f myfork featureA

该`-f`选项将替换掉远程已有的`featureA`分支，因为新的commit并非原来commit的后续
更新。当然也可以直接推送到另一个新的分支上去，比如`featureAv2`.

管理员看过第二个分支后觉得思路新颖，但想请你改下具体实现。我们只需已当前
`origin/master`分支为基准，开始一个新的特性分支`featureBv2`，然后把原来的`featureB`
更新用于其上(squash, 压制)。

	$ git checkout -b featureBv2 origin/master
	$ git merge --no-commit --squash featureB
	$ (reimplement)
	$ git commit
	$ git push myfork featureBv2

这里的`--squash`选项将目标分支里的所有更改拿来用在当前分支上，而`--no-commit`选项
告诉Git不要自动提交生成和记录(merge)提交。

这时的提交历史：

		   m  o/m
		   |   |
	---A---B---C   	 fA
	       |   |\    |
		   |   | D'--E'
			\	\ 
			 F   G
			 |   |
			fB   fBv2


**公开的大型项目**

许多大型项目都会立有一套自己的接受补丁流程，你应该注意一下其中细节。但多数项目都
允许通过开发者邮件列表接受补丁。

工作流程：

	$ git clone [url]
	$ git checkout -b topicA
	$ (work)
	$ git commit
	$ (work)
	$ git commit

如此一番后，有两个提交需要发送到邮件列表。我们可以用`git format-patch`命令来生成
`mbox`格式的文件然后作为附件发送。每个提交都会封装成一个`.patch`后缀的`mbox`文件。

	$ git format-patch -M origin/master

`-M`选项允许Git检查是否有对文件重命名的提交。

可以用邮件客户端发送这些补丁文件，也可以直接在命令行发送[略]。

### 5.3 项目的管理 ###

**使用特性分支进行工作**

管理远程仓库，如果想要集成新的代码进来，最好局限在特性分支上做。等测试完毕后，再
决定是否要并入主干分支。现在从当前主干分支为基础，新建临时分支：

	$ git branch sc/ruby_client master
或
	
	$ git checkout -b sc/ruby_client master

**采纳来自邮件的补丁**

使用`git apply`命令应用补丁：

如果收到的补丁是用`git diff`或由其它Unix的`diff`命令生成，就应该用`git apply`命令
应用补丁。

	$ git apply /tmp/patch-ruby-client.patch

这会修改当前的工作目录，效果基本与运行`patch -p1`打补丁一样，但它更为严格，且不会
出现混乱。如果是`git diff`格式描述的补丁，此命令还会相应地添加，删除，重命名文件。
当然，普通的`patch`命令是不会这么做的。另外，`git apply`是一个事物性的操作的命令，
也就是说，要不所有补丁都打上去，要不全部放弃。因为是仅仅更新当前文件，所以此命令
不会自动生成提交对象，你得手动暂存相应文件的更新状态并执行提交命令。

在实际打补丁之前，可以检查补丁是否能够干净顺利地应用到当前分支中：

	$ git apply --check 0001-this-patch-was-generated-by-git-format-patch.patch

如果无输出表示可以顺利采纳，否则除了报告错误之外，还会返回一个非零的状态，所以可以
用于在shell脚本里检测状态。

使用`git am`命令应用补丁：

对于`git format-patch`命令制作的新式补丁，应当使用`git am`命令。技术上说，`git am`
能够读取`mbox`格式的文件。

	$ git am 0001-this-patch-was-generated-by-git-formate-patch.patch

它会被干净地应用到本地分支，并且自动创建了新的提交对象(包含作者、提交、日期等信息)。

有时候也会遇到打不上补丁的情况，比如你的主干分支与别人生成补丁时所用的主干分支相差
太远或者某些依赖补丁还未应用。

这时候可以手动修复有冲突的文件，暂存起来，然后再应用：

	$ git am --resolved

或者 

	$ git am --aborted

	$ git am --skip

如果想让Git更智能地处理冲突，可以使用`-3`选项进行三方合并。如果当前分支未该补丁的
基础代码或其祖先，那么三方合并就会失败，所以该选项默认为关闭状态。一般来说，如果
该补丁是基于某个公开的提交制作而成的话，你总可以通过同步来获取这个共同祖先，所以用
三方合并选项可以解决很多麻烦：

	$ git am -3 0001-seeing-if-this-helps-the-gem.patch

如果你的邮件客户端能将多封电邮导出为`mbox`格式的文件，就可以用`git am`一次性应用
所有导出的补丁，这时可以用`git am`命令的交互模式选项`-i`，这样就会在打每个补丁前
停住，询问该如何操作：

	$ git am -3 -i mbox

打完所有补丁后，如果测试下来新特性可以正常工作，那就可以安心地将当前特性分支合并到
长期分支中去了。

**检出远程分支**

如果贡献者有自己的远程仓库，并将修改推送到此仓库中，那么当你拿到仓库的远程地址和
分支名称后，就可以加为远程分支，然后在本地合并。

	$ git remote add jessica git@github.com/jessica/myproject.git
	$ git fetch jessica
	$ git checkout -b rubyclient jessica/rubyclient

使用远程分支一个最大的好处就是能够得到提交历史。不管代码合并是不是会有问题，至少我
们知道该分支的历史分叉点(_how?_)，所以默认会从共同祖先开始自动进行进行三方合并，无
需`-3`选项，也不用像打补丁那样祈祷存在共同的基准点。

如果只是临时合作，只需用`git pull`命令抓取远程仓库上的数据，合并到本地临时分支就
可以了。一次性的抓取操作自然不会该仓库地址加为远程仓库。

	$ git checkout -b temp_branch
	$ git pull git@github.com/jessica/myproject.git
	
**决断代码取舍**

现在特性分支上已经合并好了贡献者的代码，现在要查看该特性分支上都有哪些新增的提交。
比如在`contrib`特性分支上打了两个补丁，仅查看这两个补丁的提交信息：

	$ git log contrib --not master

加上`-p`选项将展示每次提交的内容差异。

如果要查看当前所在的特性分支同将要承接合并的另一分支(`master`)的完整内容差异，可以：

	$ git diff master

不过一旦主干`master`在特性分支创建后有所修改，那么通过该命令来比较的，是最新主干上
的提交快照。所以我们可以手工定位他们的共同祖先，然后与之比较：

	$ git merge-base contrib master
> 36c7dsudgi9289agosidogu8s....

	$ git diff 36c7ds

这么做稍显麻烦。Git提供了简便的`...`语法达成这一目的。

	$ git diff master...contrib

现在看到的，就是实际要引入的新代码。这一命令特别有用。

**衍合与挑拣(cherry-pick)**

衍合或者捡选贡献者的代码，而不是简单的合并，就能够保持线状的提交历史。

除了合并和衍合，另一个引入代码的方法是“挑拣”。挑拣类似于针对某次提交的衍合。它首先
提取某次提交的补丁，然后应用在当前分支上。如果某个特性分支有多个commits，而你只想
引入其中一个就用这个办法。

	A--B--C (master)
	    \
		 D--E (featureA)

如果只想把`提交D`衍合到`master`分支上，可以这样：

	$ git cherry-pick D.SHA-1

这会引入`提交D`的代码，但是会得到不同的SHA-1值，因为应用日期不同。现在提交历史看起
来是这样的：

	A--B--C--D' (master)

**给发行版签名**

	$ git tag -s v1.0 -m 'my signed 1.0 

**生成内部版本号(Build Number)**

	$ git describe master

**准备发布**

	$ git archive master --prefix='project/' | gzip > `git describe master`.tar.gz

或者

	$ git archive master --prefex='project/' --format=zip > `git describe master`.zip

**制作简报**

	$ git shortlog --no-merges master --not v1.0.1

该命令将显示自上次发布后所有提交的简介。


第6章：Git工具
--------------

### 6.1 修订版本(Revision)选择 ###

Git允许通过几种方法来指明特定的或者一定范围内的commits.

**单个修订版本**

	$ git show SHA-1
	$ git show Short_SHA-1

**分支引用**

显示一个分支的最后一次提交的对象：

	$ git show topic1

显示某个分支指向哪个特定的SHA-1，或者查看一个被简写的SHA-1：

	$ git rev-parse topic1
	
	$ git rev-parse d518840

**引用日志(RefLog)里的简称**

	$ git perflog

	$ git show HEAD@{5}

	$ git show master@{yesterday}

**祖先引用**

	$ git show d518840^		#该提交的父提交

	$ git show HEAD^		#HEAD的父提交

	$ git show d518840^2 	#该提交的第二父提交(限合并提交时)，第一父提交是
							#合并时所在分支，第二父提交是被并入的分支

	$ git show d518840~		#该提交的第一父提交

	$ git show d518840~3	#该[合并]提交在原所在分支的父父父提交。

	$ git show d518840^^^ 	#同上

**提交范围(Commit Ranges)**

+	双点：

	这种语法主要让Git区分可从一个分支中获得而不能从另一个分支中获得的提交。

	要查看你的`experiment`分支上哪些没有被提交到`master`分支，即“所有可从`experiment`
	分支获得而不能从`master`分支中获得的提交”：

		$ git log master..experiment

	也可以反过来使用：

		$ git log experiment..master
		
	查看你将把什么推送到远程，即显示在你当前分支上而不再远程`origin/master`上的提交：

		$ git log origin/master..HEAD

		$ git log origin/master..			#HEAD可以省略

+ 	多点：

	用于查看哪些提交被包含在某些分支，但不在另一些分支中(可以指定多于两个分支):

		$ git log refA..refB
		$ git log ^refA refB
		$ git log refB --not refA		#以上三个命令等同

	查看所有被`refA`和`refB`包含的但不被`refC`包含的提交：

		$ git log refA refB ^refC
		$ git log refA refB --not refC

+	三点：

	查看被两个引用中的一个包含但不被两者同时包含的提交：

		$ git log master...experiment

	一个常用参数，用于显示每个提交到底处于哪一侧的提交：

		$ git log --left-right master...experiment


### 6.2 交互式暂存 ###

Git提供了许多脚本用于辅助命令行任务。这里，你将看到一些交互式命令，他们帮助你方便地
构建只包含特定组合和特定文件的提交。在你修改了一大批文件然后决定将这些变更分布在几个
各有侧重的提交而不是一个又大又乱的提交时，这些工具非常有用。用这种方法，你可以确保
你的提交在逻辑上划分为相应的变更集，以便于供和你一起工作的开发者审阅。

	$ git add -i


### 6.3 储藏(Stashing) ###

在工作目录修改和暂存了一些文件，还没准备好提交，但是需要紧急切换到另一个分支上工作的
时候，可以使用将这些变更“储藏”起来，以得到一个干净的工作目录，还可以重新应用这些变更。

	$ git stash

	$ git stash list

	$ git stash apply@1
	$ git stash apply --index		#恢复原来的暂存状态

	$ git stash drop
	$ git stash pop

	$ git stash branch

### 6.4 重写历史 ###

很多时候，你也许会由于某种原因想要修改你的提交历史。Git的一个卓越之处就是它允许你在
最后可能时候再做决定。你可以在你即将提交暂存区时决定什么文件归入哪一次提交，可以使用
`stash`命令暂时搁置当前的工作，也可以重写已经发生的提交以使他们看起来是另一个样子。
这个包括改变提交顺序，提交说明或者修改提交中包含的文件，将提交归并、拆分或者完全删除
——这一切在你尚未将你的工作和别人共享前都是可以的。

**修改最近一次提交**

修改提交说明：

	$ git commit --amend

更改上次提交的快照内容：

	$ git add <>
	$ git rm <>
	$ git commit --amend

使用这项技术的时候要小心，因为修正会改变提交的SHA-1值。这个很像是一次非常小的`rebase`，
所以不要在最近一次提交被推送后再去修改它。

**修改多个提交**

要修改历史中更早的提交，你必须采用更复杂的工具。Git没有一个修改历史的工具，(?)但是你
可以使用`rebase`工具来衍合一系列的提交到它们原来所在的`HEAD`而不是移到新的上。

假如想要修改最近的三次提交，必须要指明想要修改的提交的父提交：

	$ git rebase -i HEAD~3

再次提醒这是一个衍合命令——`HEAD~3..HEAD`范围内的每一次提交(共三个)都会被重写，无论你
是否修改说明。不要涵盖你已经推送到共享服务器的提交，这么做会使其他开发者产生混乱，因为
你提供了同样变更的不同版本。

运行这个命令会为你的文本编辑器提供一个可编辑的提交列表(`.git/rebase-merge`), 看起来如下：

	pick f7f3f6d changed my name a bit
	pick 310154e updated README formatting and added blame
	pick a5f4a0d added cat-file

	# Rebase 710f0f8..a5f4a0d onto 710f0f8
	#
	# Commands:
	# p, pick = ...
	# e, edit = ...
	# ...
	# If you remove a line here THAT COMMIT WILL BE LOST.
	# ...

很重要的一点是你得注意这些提交的顺序与你通常通过`git log`命令看到的是相反的。

+	修改多个提交说明(or `reword`?)
	
	在vi里将脚本编辑成：

		edit f7f3f6d changed my name a bit
		pick 310154e updated README formatting and added blame
		pick a5f4a0d added cat-file		

		$ git commit --amend
		
		$ git rebase --continue

+	重排提交

	可以将原来的提交改成这个样子：

		pick a5f4a0d added cat-file
		pick 310154e updated README formating and added blame
		
+	压制(Squashing)提交

    将一系列提交压制为单一提交。将脚本编辑成下面这样：

		pick f7f3f6d changed my name a bit
		squash 310154e updated README formating and added blame
		squash a5f4a0d added cat-file

+	拆分提交

	拆分提交就是撤销一次提交，然后多次部分的暂存或提交直到结束。下例将中间的
	那次提交"updated README formating and added blame"拆分成两次提交：第一次
	为"updated README formating", 第二次为"added blame".

	编辑脚本：

		pick f7f3f6d changed my name a bit
		edit 310154e updated README formating and added blame
		pick a5f4a0d added cat-file

	当你保存编辑器，Git倒回到列表中第一次提交的父提交，应用第一次提交(f7f3f6d),
	应用第二次提交(310154e), 然后将你带回控制台。这时你可以对那次提交进行一次
	混合的重置，这将撤销那次提交并将修改的文件撤回。此时你可以暂存并提交文件，
	直到你拥有多次提交，结束后，运行`git rebase --continue`.

		$ git reset HEAD^
		$ git add README
		$ git commit -m 'updated README formatting"
		$ git add lib/simplegit.rb
		$ git commit -m 'added blame'
		$ git rebase --continue

	然后Git在脚本中应用了最后一次提交(a5f4a0d), 你的提交历史变成这样了：

		$ git log -4 --pretty=format:"%h %s"

		1c002dd added cat-file
		9b29157 added blame
		8s823j3 updated README formatting
		932kf32 changed my name a bit

	可以看到，这修改了列表中提交的SHA-1值，所以这样做之前应该确保这个列表里不
	包含你已经推送到共享仓库里的提交。


**核弹级选项：filter-branch**

如果你想用脚本的方式修改大量的提交，还有一个重写历史的选项可用——例如，全局性地
修改电子邮件地址或者将一个文件从所有文件中删除。这个命令是`filter-branch`, 这个
命令会大面积的修改你的历史，所以你很有可能不去用它，除非你的项目尚未公开，没有
其他人在你准备修改的提交的基础上工作。

更保险一点，在一个测试分支上做这些然后在你确定产物真的是你想要的之后，再`hard-reset`
你的主分支。

从所有提交中删除一个文件：

	$ git filter-branch --tree-filter 'rm -f passwd.txt' HEAD

	$ git filter-branch --tree-filter 'rm -f *.swp' HEAD

`--tree-filter`选项会在每次检出项目时先执行指定的命令然后重新提交结果。要在所有
分支上运行`filter-branch`的话可以加上`--all`选项。

将一个子目录设置为新的跟目录：

	$ git filter-branch --subdirectory-filter sub_dir HEAD

全局性地更换电子邮件地址：

	$ git filter-branch --commit-filter '
		if [ "$GIT_AUTHOR_EMAIL" = "mellon@collie" ];
		then
			GIT_AUTHOR_NAME="mellon";
			GIT_AUTHOR_EMAIL="mellon@example.com";
			git commit-tree "$@";
 		else
			git commit-tree "$@";
		fi' HEAD

这会遍历并重写所有提交使之拥有你的新地址。因为提交里包含了它们的父提交的SHA-1值，
这个命令会修改你的历史中的所有提交，而不仅仅是包含了匹配了电子邮件地址的哪些。


### 6.5 使用Git调试 ###

**文件标注**

如果你在追踪代码中的缺陷想直到这是什么时候为什么被引进来，文件标注将会是最佳工具。
它会显示文件中对每一行进行修改的最近一次提交。

	$ git blame -L 900,910 progit.markdown

如果加上`-C`选项，还可以看到代码段的原始出处：

	$ git blame -C -L 900,910 progit.markdown


**二分查找**

标注文件在你知道问题是哪里引入的时候会有帮助。如果你不知道，并且上次代码可用的
状态已经经历了上百次提交，你可能就要求助于`bisect`明了了。`bisect`会在你的提交
历史中进行二分查找来尽快地确定哪一次提交引入了错误。

	$ git bisect start
	$ git bisect bad [HEAD]
	$ git bisect good v1.0

事实上，如果你有一个脚本在工程正常时返回0, 不正常时返回非0的话，可以完全自动地
执行`git bisect`. 

	$ git bisect start HEAD v1.0
	$ git bisect run test-error.sh

这样会自动地在每一个检出的提交里运行这个脚本直到Git找到第一个破损的提交。你也可以
运行`make`或`make test`等来为你执行自动化测试。


### 6.6 子模块 ###

经常有这样的事，当你在一个项目上工作时，你需要在其中使用另一个项目。也许它是一个
第三方开发的库或者是你自己独立开发并在多个父项目中使用的。你想将两个项目单独处理
但又需要在其中一个使用另外一个。

如果采用包含库的方法，那么不管用什么方法都很难去定制这个库，部署就更加困难了，因
为你必须确保每个客户都拥有这个库。把代码拷贝到自己的项目中带来的问题是，当上游被
修改时，任何你进行的定制化的修改都很难归并。

Git通过子模块处理这个问题。子模块允许你将一个Git仓库当作另一个Git仓库的子目录。
这允许你克隆另外一个仓库到你的项目中并保持你的提交相对独立。

**子模块初步**

假设你想把Rack库(一个Ruby的web服务器网关接口)加入到你的项目中，可能又要保持自己
的变更，又要延续上游的变更。首先你把外部的仓库克隆到你的子目录中，然后将外部项目
加为子模块：

	$ cd project/
	$ git submodule add git://github.com/somebody/rack.git rack

现在就在项目里的`rack`目录里有了一个`rack`项目。你可以进入那个目录，进行变更，
加入你自己的远程可写仓库来推送你的变更，从原始仓库拉取和归并等等。

此时运行`git status`会发现有`new file: .gitmodules`和`new file: rack`.

`.gitmodules`是一个配置文件，保存了项目URL和你拉取到的本地子目录。很重要的一点是
这个文件跟其他文件一样也是处于版本控制之下的，就像你的`.gitignore`文件一样，它跟
项目里的其他文件一样可以被推送和拉取。这是其他克隆此项目的人获知子模块项目来源的
途径。

...

-EOF- 
