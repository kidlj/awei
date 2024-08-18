---
title: Git
---


### 配置 Git

	$ git config --system user.name	'Jian Li'   # `/etc/gitconfig'
	$ git config --global user.name	'Jian Li'   # `~/.gitconfig'
	$ git config user.name 'Jian Li'            # `.git/config'


### Git diff

	$ git diff              # 未暂存的更新
	$ git diff --staged     # 未提交的更新
	$ git diff --check      # 检查多余的空白字符
    $ git diff <commit> <commit> [--] [<path>...] # diff 两个分支的同一个文件

### Git log

	$ git log -1 --stat                 # 获取最近一次提交列出更改的文件
	$ git log -p -2                     # 获取近两次提交并展示差异
	$ git log -p --word-diff            # 获取单词层面的差异对比
	$ git log --pretty=oneline          # 单行模式
	$ git log --pretty=format:"%h %s"   # 定制显示选项
	$ git log --graph                   # 显示分支合并历史
	$ git log --grep='tweak'            # 搜索提交说明
	$ git log --author="Jian Li" --commiter="Not Me" --all-match
	$ git log --since="2014-08-28" -- <path>
    $ git log -- <path>                 # commit logs
    $ git log -p -- <path>              # commit diffs
    $ git log --follow -p  -- <path>    # follow RENAME

### Git tag

    $ git tag 0.1.0     # new tag
    $ git tag           # list tags
    $ git push --tags   # push tags


### 撤销操作

修改上一次的提交说明：

	$ git commit --amend

如果上次提交时忘了暂存某些修改，可以先补上暂存操作，然后再`--amend`提交，这样不会产生新的提交：

	$ git add forgotten_file
	$ git commit --amend

取消已经暂存的文件：

	$ git reset HEAD <file>

取消对文件做出的修改：

	$ git checkout -- <file> 


### Git别名

	$ git config --global alias.co checkout
	$ git config --global alias.br branch
	$ git config --global alias.ci commit
	$ git config --global alias.st status
	$ git config --global alias.unstage 'reset HEAD --'
	$ git config --global alias.last 'log -1 HEAD'
	$ git config --global alias.visual "!gitk"

### 集成已有仓库代码

    $ git remote add origin <>
    $ git fetch origin master
    $ git merge origin/master --allow-unrelated-histories


### 理解分支

1. 每次提交保存一个提交对象，该对象指向一个树对象，树对象可以用来恢复当前目录树的快照；

2. 分支本质上是指向某个提交对象的指针，检出某个分支就是恢复到该提交对象所保存（指向）的目录树，然后 HEAD 指针同时指着这个提交对象用来区分当前所在分支；

3. 分支上每提交一次本次提交对象就指向上一次提交对象，（分支指针和 HEAD 指针指向这个最新的提交对象），从而使每个分支形成一条线，而两个分支又能有共同的合并基础（两条线前面的共同父对象，分支分叉点）； 
											
### 远程分支

1. 推送本地的`serverfix`分支到远程服务器上：

		$ git push origin serverfix
		$ git push origin serverfix:serverfix
		$ git push origin serverfix:awesomebranch

2. 接下来，当你的协作者再次从服务器上获取数据时，他们将得到一个新的远程分支`origin/serverfix`：

		$ git fetch orgin

		> ...
		> From git@github.com:kidlj/...
		>  [new branch] serverfix  ->  origin/serverfix

	值得注意的是，在 fetch 操作下载好新的远程分支之后，你仍然无法在本地编辑该远程仓库中的分支。换句话说，在本例中，你不会有一个新的 serverfix 分支，有的只是一个你无法移动的 origin/serverfix 指针。

3. 如果要把该分支内容合并到当前分支，可以运行：

		$ git merge origin/serverfix

4. 如果他们想要一份自己的`serverfix`来开发，可以在远程分支的基础上分化出一个新的分支来：

		$ git checkout -b serverfix origin/serverfix

	这会切换到新建的`serverfix`本地分支，其内容同远程分支`origin/serverfix`一致，你可以在里面继续开发了。

	由此检出的本地分支叫做跟踪分支(tracking branch), 因为这个操作很常用所以它有一个快捷方式：

		$ git checkout --track origin/serverfix

    在跟踪分支上执行`git pull`操作，Git 能自动识别去哪个服务器拉取数据并且合并到哪个分支。特别的是，当克隆一个仓库时，Git 通常会自动地创建一个跟踪`origin/master`的`master`分支。

5. 如果不再需要某个远程分支了，比如搞定了某个特性并把它合并到了远程的`master`分支里，可以用这个无厘头的语法来删除它：

		$ git push origin :serverfix

	意味着「在这里提取空白然后把它变成远程分支」，即删除远程分支。


### 衍合


1. 基础衍合：把 c3 衍合入 c4

	首先，分支情况如下：

						 <--[c3] (experiment)
						/
		[c0]<--[c1]<--[c2]<--[c4] (master)

	将 c3 对 c2 开始的变化打到 c4 上：

		$ git checkout experiment
		$ git rebase master

	于是生成一个新的提交 c3', 如下所示：

								 (experiment)
									  |
		[c0]<--[c1]<--[c2]<--[c4]<--[c3']
							  |
						   (master)

	现在，回到 master 分支然后进行一次快进合并。

		$ git checkout master
		$ git merge experiment

	衍合完成，提交历史如下：

								 (experiment)
									  |
		[c0]<--[c1]<--[c2]<--[c4]<--[c3']
									  |
								   (master)


	总的来说，衍合就是把本不能快进的合并变成快进合并。

2. 高级衍合：把 client 分支的 c8, c9 衍合入 master 分支

	首先，从一个特性分支里在分出一个特性分支的历史：：

		[c1]<--[c2]<--[c5]<--[c6] (master)
				 \
				  *<--[c3]<--[c4]<--[c10] (server)
						\
						 *<--[c8]<--[c9] (client)

	现在想要仅仅把 client 分支的 c8, c9 衍合入 master 分支。

		$ git rebase --onto master server client

	这基本上是说“检出 client 分支，找出 client 分支和 server 分支的共同祖	  先之后的变化(即c8和c9)，然后把它们在 master 上重演一遍。

	现在可以快进 master 分支了：

		$ git checkout master
		$ git merge client

	结果如下图所示：

										   (client)
											  |
		[c1]<--[c2]<--[c5]<--[c6]<--[c8']<--[c9'] (master)
				 \
				  *<--[c3]<--[c4]<--[c10] (server)

3. 现在你决定把 server 分支的变化也包含进来。

	可以直接把 server 分支衍合到 master 而不用手动转到 server 分支再衍合。

		$ git rebase master server

	于是 server 的进度应用到 master 的基础上，而后快进主分支 master ：

		$ git checkout master
		$ git merge server

	现在 client 和 server 分支的变化都被整合了，不妨删掉它们:

		$ git branch -d client
		$ git branch -d server

	把提交历史变成如下图的样子：

															      (server)
																	  |
																	  |
		[c1]<--[c2]<--[c5]<--[c6]<--[c8']<--[c9']<--[c3']<--[c4']<--[c10']
											  |                       |
											  |                       |
										   (client)               (master)

4. 衍合的风险

	一句话可以总结：

	> 永远不要衍合那些已经推送到公共仓库的更新。
	> Do not rebase commits that you have pushed to a public repository.

	(其实就是说一定要在分享之前衍合，分享出去之后就不要再衍合。)

### Remove sensitive data[1]

    $ git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch _wiki/proxy.mkd' --prune-empty --tag-name-filter cat -- --all
    $ git push origin --force --all
    $ git push origin --force --tags

    $ git add .
    $ git commit -m 'add file back'
    $ git push

[1]: https://help.github.com/en/articles/removing-sensitive-data-from-a-repository