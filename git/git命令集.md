---
title: git命令集
date: 2020-05-9 14:32:33
tags: 
	- 命令行
	- git
categories: 
	- git
---

# git

## 基本操作

![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

### 新建代码库

```bash
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

### 增加/删除文件

```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 递归删除
git rm -r *  

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

### 代码提交

```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

### 分支

```
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

#使用--no-ff参数后，会执行正常合并，在Master分支上生成一个新节点，为了保证版本演进的清晰。
git merge --no-ff

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]
# 一次转移多个提交
$ git cherry-pick <HashA> <HashB>
#如果想要转移一系列的连续提交，可以使用下面的简便语法。不包括A
$ git cherry-pick A..B 
#注意，使用上面的命令，提交 A 将不会包含在 Cherry pick 中。如果要包含提交 A，可以使用下面的语法。
git cherry-pick A^..B 

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

### 标签

```
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

### 查看信息

```
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```

### 远程同步

```bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 删除一个远程仓库
$ git remote rm origin

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

### 撤销

```

# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 查看所有暂存
git stash list

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop

```

## 配置相关

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件 global是全局配置
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"

# 为某个分支单独设置，这里是设置 dev 分支
git config branch.dev.rebase true
# 全局设置，所有的分支 git pull 均使用 --rebase
git config --global pull.rebase true
git config --global branch.autoSetupRebase always

# 删除全局代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 设置全局代理
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'

# 只对github进行代理，对国内的仓库不影响
git config --global http.https://github.com.proxy https://127.0.0.1:1080
git config --global https.https://github.com.proxy https://127.0.0.1:1080

# 取消上面的代理
git config --global --unset https.https://github.com.proxy
git config --global --unset http.https://github.com.proxy

```

## 统计

### 提交数统计

统计提交(commit)次数

```
git log --oneline | wc -l
```

### 统计每个人增删行数

```
git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
```

**结果**

```
fengxing	added lines: 56705, removed lines: 21012, total lines: 35693
jiaozhegang	added lines: 10477, removed lines: 10156, total lines: 321
unknown	added lines: 10469, removed lines: 14, total lines: 10455
wangkai	added lines: 5, removed lines: 4, total lines: 1
zhangyun	added lines: 33495, removed lines: 8966, total lines: 24529
zrj	added lines: 7841, removed lines: 15648, total lines: -7807
冯海超	added lines: 2, removed lines: 1, total lines: 1
```

```
71012679	added lines: 2, removed lines: 2, total lines: 0
chouyanhejiutangtou	added lines: 8979, removed lines: 1925, total lines: 7054
fengxing	added lines: 26709, removed lines: 7520, total lines: 19189
unknown	added lines: , removed lines: , total lines:
wangkai	added lines: 17284, removed lines: 8779, total lines: 8505
zhangyun	added lines: 22444, removed lines: 6506, total lines: 15938
```

>git log 参数说明：
>–author 指定作者
>–stat 显示每次更新的文件修改统计信息，会列出具体文件列表
>–shortstat 统计每个commit 的文件修改行数，包括增加，删除，但不列出文件列表：
>–numstat 统计每个commit 的文件修改行数，包括增加，删除，并列出文件列表：
>-p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新
>例如：git log -p -2
>–name-only 仅在提交信息后显示已修改的文件清单
>–name-status 显示新增、修改、删除的文件清单
>–abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符
>–relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）
>–graph 显示 ASCII 图形表示的分支合并历史
>–pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）
>例如： git log --pretty=oneline ; git log --pretty=short ; git log --pretty=full ; git log --pretty=fuller
>–pretty=tformat: 可以定制要显示的记录格式，这样的输出便于后期编程提取分析
>例如：git log --pretty=format:""%h - %an, %ar : %s""
>下面列出了常用的格式占位符写法及其代表的意义。
>选项 说明
>%H 提交对象（commit）的完整哈希字串
>%h 提交对象的简短哈希字串
>%T 树对象（tree）的完整哈希字串
>%t 树对象的简短哈希字串
>%P 父对象（parent）的完整哈希字串
>%p 父对象的简短哈希字串
>%an 作者（author）的名字
>%ae 作者的电子邮件地址
>%ad 作者修订日期（可以用 -date= 选项定制格式）
>%ar 作者修订日期，按多久以前的方式显示
>%cn 提交者(committer)的名字
>%ce 提交者的电子邮件地址
>%cd 提交日期
>%cr 提交日期，按多久以前的方式显示
>%s 提交说明
>–since 限制显示输出的范围，
>例如： git log --since=2.weeks 显示最近两周的提交
>选项 说明
>-(n) 仅显示最近的 n 条提交
>–since, --after 仅显示指定时间之后的提交。
>–until, --before 仅显示指定时间之前的提交。
>–author 仅显示指定作者相关的提交。
>–committer 仅显示指定提交者相关的提交。

### 自定义格式显示提交历史

```
git log --pretty=format:"%h - %an, %ar : %s"
```

### 查看最近一天的代码提交情况

```
git log --since=1.days

# 简洁版
git log --since=1.days --oneline
```

### 按照时间显示输出范围

```
# 查看最近一周的代码提交情况
git log --since=1.weeks

# 一天之内的log
git log --since=1.day.ago

# 一分钟之前的所有 log
git log --until=1.minute.ago
 
# 一个小时之内的 log
git log --since=1.hour.ago 

# 一个月之前到半个月之前的log
git log --since=`.month.ago --until=2.weeks.ago 

# 某个时间段的 log
git log --since ==2013-08.01 --until=2013-09-07 
```

### 查看git上的个人代码量

使用的使用把`fengxing`换成自己的用户名

```
git log --author="fengxing" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
```

### 代码总行数

```
find . "(" -name "*.java" ")" -print | xargs wc -l
```

### 贡献者统计

```
git log --pretty='%aN' | sort -u | wc -l
```

### 查看仓库提交者排名前 5

```
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```

### 显示所有提交过的用户，按提交次数排序

```bash
git shortlog -sn
```

