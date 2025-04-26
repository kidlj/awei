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

### Remove sensitive data[github]

    $ git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch _wiki/proxy.mkd' --prune-empty --tag-name-filter cat -- --all
    $ git push origin --force --all
    $ git push origin --force --tags

    $ git add .
    $ git commit -m 'add file back'
    $ git push

[github]: https://help.github.com/en/articles/removing-sensitive-data-from-a-repository