---
title: git 命令
date: 2019-01-06 23:50:33
categories:
- tools
tags:
- tools
- git
---
# git 命令

git是分布式版本控制系统，同一个git仓库可以分布到不同的机器上，每一台机器上的版本都是一样的，没有主次之分。

## 配置
当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：
```shell
$ git config --global user.name "sunchuanxi"
$ git config --global user.email sunchuanxi@126.com
```
<!--more-->
```shell
# 检查的配置，来列出所有 Git 当时能找到的配置
$ git config --list

# 可以通过输入 git config <key> 来检查 Git 的某一项配置
$ git config user.email
```

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore 的文件。
```text
# no .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in the build/ directory
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```

## 基础操作

```shell
# 如果你打算使用 Git 来对现有的项目进行管理，你只需要进入该项目目录并初始化仓库：
$ git init

# 也克隆现有的仓库
$ git clone git@github.com:WesternSun/WesternSun.github.io.git
```

```shell
# 检查当前文件状态
$ git status

# 状态简览，一种更为紧凑的格式输出
$ git status -s

# 跟踪新文件
git add README.md
# 把当前修改添加到暂存区以备commit(它实际是把当前的修改放到暂存区，如果你在add后再去修改，那么它不会管理到这后续的改动）
```
add命令，是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。

如果在git add README.md之后，又对README.md进行修改，然后用git status命令查看会发现，现在README.md 文件同时出现在暂存区和非暂存区。 这怎么可能呢？ 好吧，实际上 Git 只不过暂存了你运行 git add 命令时的版本， 如果你现在提交，CONTRIBUTING.md 的版本是你最后一次运行 git add 命令时的那个版本，而不是你运行 git commit 时，在工作目录中的当前版本。 所以，运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来


```shell
# 查看尚未暂存的文件更新了哪些部分
$ git diff

# 若要查看已暂存的将要添加到下次提交里的内容
$ git diff --staged
```

```shell
# 现在的暂存区域已经准备妥当可以提交了
# 每次准备提交前，先用 git status 看下，是不是都已暂存起来了
$ git commit -m "add README.md"

# 删除文件也是一个修改操作，要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交
$ git rm README.md
# 下一次提交后，该文件就不再纳入版本管理了
# 如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f（译注：即 force 的首字母）。 
# 这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被 Git 恢复。

# 想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 
$ git rm --cached README.md

# Git 中对文件改名
$ git mv file_from file_to
```

## 历史记录

```shell
# 想回顾下提交历史，按提交时间列出所有的更新，最近的更新排在最上面
$ git log

# 一个常用的选项是 -p，用来显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交
$ git log -p -2

# 每次提交的简略的统计信息
$ git log --stat

# 添加了一些ASCII字符串来形象地展示你的分支、合并历史
$ git log --pretty=format:"%h %s" --graph
```

# 撤消操作

$ git commit --amend

这个命令会将暂存区中的文件提交。 如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令），那么快照会保持不变，而你所修改的只是提交信息。

例如，你提交后发现忘记了暂存某些需要的修改，可以像下面这样操作：
```shell
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

```shell
# 就退回到上一个版本
$ git reset --hard HEAD^ 

# 指定回到某个特定的版本  commit id 可以不用写全
$ git reset --hard 5498a   

# 把文件在工作区的修改全部撤销
$ git checkout -- <file>   
# 如果<file>还没有add到暂存区（stage），checkout 后回滚到上一个commit版本
# 如果<file>已经add到stage，checkout 后回滚到刚刚add到stage的状态

# 把文件修改从stage剔除
$ git reset HEAD <file> 
```

## 远程仓库

```shell
# 显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL
$ git remote -v

# 添加远程仓库，
$ git remote add origin git@github.com:WesternSun/WesternSun.github.io.git

# 修改一个远程仓库的简写名
$ git remote origin org

# 远程仓库的移除
$ git remote rm dev

# 查看远程仓库
$ git remote show origin

# 推送到远程仓库
$ git push origin master

# 从远程仓库中获得数据
$ $ git fetch [remote-name]
# 必须注意 git fetch 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作

# 从克隆的服务器上抓取数据并自动尝试合并到当前所在的分支
$ git pull
```

## 标签
``` shell
# 列出标签
$ git tag

#创建标签
git tag -a v1.4 -m 'my version 1.4'

# 查看标签信息与对应的提交信息
git show v1.4

# 推送标签
git push origin v1.5
# 默认情况下，git push 命令并不会传送标签到远程仓库服务器上。
```