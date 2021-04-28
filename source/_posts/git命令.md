---
title: git操作整理
date: 2019-09-17 22:15:04
tags: 工具
categories: 工具
---

> **获得Git项目的仓库有两种方式：**
>
> 1. 在现有的目录下面通过导入所有的文件夹来创建Git仓库
>
> 2. 从已有的仓库克隆一个新新的镜像仓库来使用

<!--more-->

## git init

在现有的工作目录中初始化新仓库：

{% asset_img git_init.png %}

初始化完成后，在当前的目录下面会出现一个名为.git的目录。



## git clone

从现有的仓库克隆。

克隆的命令：`git clone [url]`

```javascript
git clone git@github.com:shuliqi/echarts-vue.git
```



## git status

检查当前文件的状态。

> git的工作目录下的所有文件都不外呼两种状态：已跟踪，未跟踪
>
> `已跟踪：` 指被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间之后，它们的状态可能是`未更新`，`已修改`，`已放入暂存入`
>
> `未跟踪：`没有上次的更新时的快照，也不在当前的暂存区域（即不是已跟踪文件就是未更新文件）。
>
> 初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为未修改。

{% asset_img no_status.png %}

说明当前的工作目录很干净。所有的跟踪文件在上次提交之后没有更新过。



我们新建一个文件，1.js。 通过 `git status` 可看出我们新建的文件在未跟踪的文件列表里面.

{% asset_img add-status.png %}



## git add

跟踪一个新文件。

例如，要跟踪一个新文件1.js, 则可以:

```javascript
git add 1.js
```

添加跟踪完之后，我们再根据`git status` 查看状态，可看出该文件已经添加到已追踪列表.

{% asset_img git_new_status.png %}

只要在 “Changes to be committed” 这行下面的，就说明是已暂存状态。如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中

 当我们在修改1.js 文件。再`git status`：

{% asset_img edit_status.png %}

文件 1.js  出现在 “Changes not staged for commit"  这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。要暂存这次更新，需要运行 `git add` 命令。



## .gitignore

有一些文件我们不需要纳入git版本控制。但是我们也不希望它出现在未跟踪的文件列表中。我们可以创建一个名为：.gitignore的文件。里面列出需要忽略的文件的模式。

{% asset_img gitignore.png %}

文件`.gitignore`的格式如下：

* 所有的以`#`符号开头，所有的空行都会被省略。
* 可以使用标准的 glob 模式匹配（指 shell 所使用的简化了的正则表达式）。
* 匹配模式最后跟反斜杠（`/`）说明要忽略的是目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。

```javascript
# 此为注释 – 将被 Git 忽略
# 忽略所有 .js 结尾的文件
*.js
# 但 lib.a 除外
!shuliqi.js
# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
/TODO
# 忽略 build/ 目录下的所有文件
build/
# 忽略 doc/ 目录下所有扩展名为 txt 的文件
doc/**/*.txt
# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
doc/*.txt

```



## git diff

`git status` 显示比较简单，仅仅是列出了修改过的文件。如果要看文件具体修改了哪些地方，可以使用`git diff`

如果我们修改了文件。再使用`git diff` 查看：

{% asset_img git_diff_1.png %}

比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。

**注意**

* `git diff ：` 对比工作区(未 git add)和暂存区(git add 之后)

* `git diff —cached:` 对比暂存区(git add 之后)和版本库(git commit 之后)

* `git diff HEAD:` 对比工作区(未 git add)和版本库(git commit 之后)



## git commit 

提交更新。

使用`git commit ` 提交暂存区代码。

```
git commit
```

这种方式会启动文本编辑器以便输入本次提交的说明。

{% asset_img git_commit.png %}

还有一种提交的方式，不需要启动文本编辑器来提交本次的说明。

{% asset_img git_commit_m.png %}

提交后它会告诉你，当前是在哪个分支（master）提交的，本次提交的完整 SHA-1 校验和是什么（`463dc4f`），以及在本次提交中，有多少文件修订过，多少行添改和删改过。



虽然使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。

那么使用 `git commit -a -m"跳过暂存区提交"`

{% asset_img git_commit_a.png %}

提交之前不再需要 `git add` 文件了。



**`git commit --amend` ** 此命令将使用当前的暂存区域快照提交。如果刚才提交完没有作任何改动，直接运行此命令的话，相当于有机会重新编辑提交说明，但将要提交的文件快照和之前的一样。

我们使用`git log  -1` 查看上一次commit的简要信息

{% asset_img git_commit_amend1.png %}

撤销上次的commit, 重新编辑提交说明。

```
git commit --amend
```

提交说明改成: “提交新东西(git commit —amend)”

{% asset_img git_commit_amend2.png %}

再`git log` 查看当前的提交历史

{% asset_img git_commit_amend3.png %}

从图中看看出，我们的提交说明已经改变了。并且提交历史也没有我们第一次的commit(我们已经改变了)



如果上次的提交我们有些文件改动没有暂存，我们可以补上暂存操作，然后再`git commit --ammend`提交。

例如： 我们提交了一次commit

{% asset_img git_commit_amend3.png %}

当前的更新历史为：

{% asset_img git_commit_amend5.png %}

补上一个暂存操作，再`git commit --ammend`，可更改上次的提交了.

{% asset_img git-commit-amend6.png %}

{% asset_img git-commit-amend7.png %}

总结：以下的操作只会产生一个commit

```javascript
git commit -m '提交'
git add forgotten_file
git commit --amend
```





## git rm

移除文件。

如果想在git 删除某个文件，就必须得从已跟踪（暂存区）的文件清单中移除。然后再提交。使用`git rm` 移除文件。那么移除的文件就不会出现在未跟踪的清单中了。

如果手动删除了一个文件，再使用`git status`查看：

{% asset_img git_rm.png %}

显示在“Changes not staged for commit”。也就是文件还在跟踪中。

如果使用`git rm`

{% asset_img git_rm_2.png %}

那么最后提交的时候，该文件就不再纳入版本管理了。

如果我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。则可以使用git rm --cached 文件

{% asset_img git_rm_cached.png %}



## git mv

移除文件（更改文件名）。

git 无法跟踪文件的移除操作。如果我们改了一个文件名。 Git 并不会体现出这一操作。如果想给一个文件改名，则可以使用：

```
git mv old_name new_name
```

{% asset_img git_mv.png %}

`git mv `命令相当于以下命令:

```
➜ mv old.txt new.txt
➜ git rm old.txt
➜ git add new.txt
```



## git log

查看提交历史。

使用`git log` 查看提交历史.

{% asset_img git_log.png %}

如果默认没有参数， 会按提交的时间显示所有的更新。最近的更新排在最上面。每一次的提交都有SHA-1 校验，作者的名字和邮箱，提交的时间， 提交声明。

`git log`	 一些常用的选项:

```javascript
git log -p // 展开显示每次提交的内容的差异
```

{% asset_img git_log_p.png %}

```javascript
git log -<n> // 加上n > 0 的数字， 显示最近几次的提交
```

例如：`git log -1`

{% asset_img git_log_1.png %}

```
git log -p --word-diff. // 单词层面的比较
```

{% asset_img git_log_word.png %}

```
git log --stat //显示简要的增改行数统计
```

{% asset_img git_log_stat.png %}



**整理的常用的选项：**

| 选项              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `-p`              | 按补丁格式显示每个更新之间的差异。                           |
| `--word-diff`     | 按 word diff 格式显示差异。                                  |
| `--stat`          | 显示每次更新的文件修改统计信息。                             |
| `--shortstat`     | 只显示 --stat 中最后的行数修改添加移除统计。                 |
| `--name-only`     | 仅在提交信息后显示已修改的文件清单。                         |
| `--name-status`   | 显示新增、修改、删除的文件清单。                             |
| `--abbrev-commit` | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。            |
| `--relative-date` | 使用较短的相对时间显示（比如，“2 weeks ago”）。              |
| `--graph`         | 显示 ASCII 图形表示的分支合并历史。                          |
| `--pretty`        | 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。 |
| `--oneline`       | `--pretty=oneline --abbrev-commit` 的简化用法。              |

其他的：

| 选项                | 说明                         |
| ------------------- | ---------------------------- |
| `-(n)`              | 仅显示最近的 n 条提交        |
| `--since, --after`  | 仅显示指定时间之后的提交。   |
| `--until, --before` | 仅显示指定时间之前的提交。   |
| `--author`          | 仅显示指定作者相关的提交。   |
| `--committer`       | 仅显示指定提交者相关的提交。 |





## git reset

取消已经暂存的文件。

假如我们不小心`git add` 了一个文件加到暂存区。但是我们想撤消该文件加到暂存区。则可以使用`git reset`进行撤销。

> 命令不需要我们强制记住，`git add` 之后我们使用`git status` 可以看到git 提示我们如何撤消。

{% asset_img git_reset.png %}

```
git reset HEAD <file> // 如何在暂存区撤销一个文件
```

{% asset_img git_reset1.png %}

即可撤消。



## git checkout -- file

取消对文件的修改。

如果我们觉得对一个文件的修改没有必要。想要取消修改。则可以使用`git checkout -- <file>`

> 命令不需要我们强制记住，`git add` 之后我们使用`git status` 可以看到git 提示我们如何取消对一个文件的修改。

{% asset_img git_checkout-file.png %}



{% asset_img git_checkout_file1.png %}

撤消之后，再`git status`该文件则已经回到没有修改过的状态了。



## git remote

查看远程仓库。

 `git remote`: 会显示每个远程仓库的简短的名字.如：

```basic
➜ git remoete
origin
```



`git remote -v`: 显示远程仓库的名字和对应的clone 地址。如：

```basic
➜ git remote -v
origin	git@github.com:shuliqi/rzfx.git (fetch)
origin	git@github.com:shuliqi/rzfx.git (push)
```



`git remote add [name] [url]` 添加远程仓库。如：

```basic
➜ git remote add shuliqi git@github.com:shuliqi/rzfx.git
➜ git:(master) git remote -v
origin	git@github.com:shuliqi/rzfx.git (fetch)
origin	git@github.com:shuliqi/rzfx.git (push)
shuliqi	git@github.com:shuliqi/rzfx.git (fetch)
shuliqi	git@github.com:shuliqi/rzfx.git (push)
```

可以通过 `git fetch shuliqi` 来抓取shuliqi仓库有的但是本地没有的信息。



`git remote show [remote-name] `: 查看某个远程仓库的相信信息。

例如：查看克隆的origin的仓库的分支。

```basic
➜ git remote show origin
* remote origin
  Fetch URL: git@github.com:shuliqi/rzfx.git
  Push  URL: git@github.com:shuliqi/rzfx.git
  HEAD branch: master
  Remote branches:
    master     tracked
    shu        tracked
    shu-branch tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local refs configured for 'git push':
    master pushes to master (fast-forwardable)
    shu    pushes to shu    (up to date)
```

给出了很多的信息：

* 仓库的克隆地址

* 提醒你要是在master 可以使用`git pull`抓取数据。

* 列出了所有的处于跟踪的分支（master, shu, shu-branch ）

  

`git remote rename [old-remote-name] [new-remote-name] ` 命令修改某个远程仓库在本地的简称。

例如：把本地仓库`origin`改成成`shu`

```basic
➜ git remote
origin
shuliqi
➜  git remote rename shuliqi shu
➜  git remote
origin
shu
```



git remote rm [remote-name]` 删除远端仓库。

例如：删除远程仓库`shu`

```basic
➜ git remote
origin
shu
➜  git remote rm shu
➜  git remote
origin
```





## git fetch

`git fetach <remote-name>`会到远程仓库中拉取所有你本地仓库中还没有的数据。

fetch 命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支。

```basic
➜ git fetch origin
```



## git push

将本地仓库中的数据推送到远程仓库 `git push [remote-name] [branch-name]` 

例如： 我们新建一个分支`shu`。对代码做了一些改动。并且推送到远程跟踪的相应的分支。

```basic
➜  git checkout -b shu
Switched to a new branch 'shu'
➜  git commit -a -m"将本地数据推送到远程"
[shu be180a1] 将本地数据推送到远程
 1 file changed, 1 insertion(+), 1 deletion(-)
➜  git push origin shu
Counting objects: 15, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (15/15), 1.41 KiB | 1.41 MiB/s, done.
Total 15 (delta 9), reused 0 (delta 0)
remote: Resolving deltas: 100% (9/9), completed with 4 local objects.
remote:
remote: Create a pull request for 'shu' on GitHub by visiting:
remote:      https://github.com/shuliqi/rzfx/pull/new/shu
remote:
remote:
remote:
remote: GitHub found 2 vulnerabilities on shuliqi/rzfx's default branch (1 high, 1 moderate). To find out more, visit:
remote:      https://github.com/shuliqi/rzfx/network/alerts
remote:
To github.com:shuliqi/rzfx.git
 * [new branch]      shu -> shu
```

这条命令如期执行的条件：

* 在该克隆的服务器上有读写权限。
* 在同一时刻，没有其他人提交更新

如果在推送前，有人已经做了若干更新。 那么你的更新会被驳回。你只能把他们的更新抓到本地。 并且和冰岛自己的项目中。然后进行再次推送才会成功。



## git tag

#### 查看标签

使用`git tag`	列出所有的标签

```basic
➜  git tag
v0.0.1
v0.0.2
v0.0.3
```



> Git 使用的标签有两种类型：
>
> * 轻量级的（lightweight）和含附注的（annotated）。轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。
> * 含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证。一般我们都建议使用含附注型的标签，以便保留相关信息；当然，如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签也没问题。



#### 新建含附注标签

如命令：`git tag -a v0.0.1 -m"新建一个标签，标记0.0.1版本"` 可以新建一个含附注标签。

```basic
➜  git tag -a v0.0.1 -m"新建一个标签，标记0.0.1版本"
➜  git tag
v0.0.1
```

`-m` 选项则指定了对应的标签说明，Git 会将此说明一同保存在标签对象中。如果没有给出该选项，Git 会启动文本编辑软件供你输入标签说明。

`git show [tag-name]`：查看一个标签的的信息。	

```basic
tag v0.0.1
Tagger: shuliqi <1169046371@qq.com>
Date:   Fri Sep 27 19:52:56 2019 +0800

新建一个标签，标记0.0.1版本
```

可看出：标签名字，标签说明等。



#### 轻量级标签

轻量级标签实际上就是一个保存着对应提交对象的校验和信息的文件。要创建这样的标签，一个 `-a`，`-s` 或 `-m` 选项都不用，直接给出标签名字即可。

```basic
➜  git tag v1.1.6
➜  git show v1.1.6
commit be180a1be40d9ce2ca8fe3a8c9bb2e9e7322275f (HEAD -> shu, tag: v1.1.6, tag: v0.0.3, tag: v0.0.2, tag: v0.0.1, tag: show, origin/shu)
Author: shuliqi <1169046371@qq.com>
Date:   Fri Sep 27 19:02:15 2019 +0800
```

可看出轻量级标签就只有相应的提交对象摘要。



#### 分享标签

可以使用`git push [remote-name] [tag-name]` 命令将某一个tag 推送到远端。

```basic
➜  git push origin v0.0.1
Counting objects: 1, done.
Writing objects: 100% (1/1), 200 bytes | 200.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To github.com:shuliqi/rzfx.git
 * [new tag]         v0.0.1 -> v0.0.1
```

可以使用`git push origin --tags` 命令推送所有的tag 到远端。

```basic
➜  git push origin --tags
Counting objects: 2, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 194 bytes | 194.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To github.com:shuliqi/rzfx.git
 * [new tag]         show -> show
 * [new tag]         v0.0.2 -> v0.0.2
 * [new tag]         v0.0.3 -> v0.0.3
 * [new tag]         v1.1.6 -> v1.1.6
```







​	

