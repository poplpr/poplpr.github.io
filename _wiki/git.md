---
layout: wiki
title: Git
cate1: Tools
cate2: Version Control
categories: Git
description: Git 常用操作记录。
keywords: Git, 版本控制
---

## 初次使用配置

### 总结一下常用配置

```bash
git config --global user.name "poplpr"
git config --global user.email 123456@qq.com
git config --global core.editor=vim
git config --global core.autocrlf input
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
git config --global core.quotepath false
```

### 三种配置

Git有三种配置，分别以文件的形式存放在三个不同的地方。可以在命令行中使用git config工具查看这些变量。

**系统配置（对所有用户都适用）**
存放在git的安装目录下：`%Git%/etc/gitconfig`；若使用 git config 时用 --system 选项，读写的就是这个文件：
```bash
git config --system core.autocrlf
```

**用户配置（只适用于该用户）**
存放在用户目录下。例如 linux 存放在：`~/.gitconfig`；若使用 git config 时用 --global 选项，读写的就是这个文件：
```bash
git config --global user.name
```

**仓库配置（只对当前项目有效）**
当前仓库的配置文件（也就是工作目录中的 `.git/config` 文件）；若使用 git config 时用 --local 选项，读写的就是这个文件：
```bash
git config --local remote.origin.url
```

注：每一个级别的配置都会覆盖上层的相同配置，例如 `.git/config` 里的配置会覆盖 `%Git%/etc/gitconfig` 中的同名变量。 

| 配置     | 命令                                         |
| -------- | -------------------------------------------- |
| 设置姓名 | git config --global user.name "poplpr"       |
| 设置邮箱 | git config --global user.email 123456@qq.com |

### 文本换行符配置

假如你正在 Windows 上写程序，又或者你正在和其他人合作，他们在 Windows 上编程，而你却在其他系统上，在这些情况下，你可能会遇到行尾结束符问题。这是因为 Windows 使用回车和换行两个字符来结束一行，而 Mac 和 Linux 只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。

Git 可以在你提交时自动地把行结束符 CRLF 转换成 LF，而在获取代码时把 LF 转换成 CRLF。用 `core.autocrlf` 来打开此项功能，如果是在 Windows 系统上，把它设置成`true`，这样当获取代码时，LF会被转换成CRLF：
```bash
$ git config --global core.autocrlf true
```

Linux 或 Mac 系统使用 LF 作为行结束符，因此你不想 Git 在切出文件时进行自动的转换；当一个以 CRLF 为行结束符的文件不小心被引入时你肯定想进行修正，把 `core.autocrlf` 设置成 `input` 来告诉Git在提交时把 CRLF 转换成 LF，获取代码时不转换，Windows 上 VsCode 可以设置行尾序列为 LF，然后也可以使用 `input`：

```bash
$ git config --global core.autocrlf input
```

这样会在Windows系统上的切出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。

如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置`false`取消此功能，把回车符记录在库中：
```bash
$ git config --global core.autocrlf false
``` 

### 文本编码配置

- `i18n.commitEncoding` 选项：用来让 `git commit log` 存储时，采用的编码，默认UTF-8。
- `i18n.logOutputEncoding` 选项：查看 `git log` 时，显示采用的编码，建议设置为UTF-8。 

下面的两个中文编码支持和显示路径的中文我一般不调。

#### 中文编码支持
```bash
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
```
#### 显示路径中的中文
```bash
git config --global core.quotepath false
``` 

### 与服务器的认证配置

**http / https协议认证**

- 设置口令缓存：
```bash
git config --global credential.helper store
```
- 添加HTTPS证书信任：
```bash
git config http.sslverify false
```

**ssh协议认证**

SSH协议是一种非常常用的Git仓库访问协议，使用公钥认证、无需输入密码，加密传输，操作便利又保证安全性，通过

```bash
ssh-keygen -t rsa -C 123456@qq.com
```

来生成 SSH key。然后我这里都设置的默认。

然后把生成的 `*.pub` 文件丢到远程服务器的 `~/.ssh/authorized_keys`。如果没有这个文件请输入 `touch ~/.ssh/authorized_keys` 创建一个。接下来可以写成 `echo 你的公钥 >> ~/.ssh/authorized_keys` 或者用 vim 进行导入。

和服务器的配置请慢慢自行配置吧。

## 常用命令

| 功能                                  | 命令                                        |
| :------------------------------------ | :------------------------------------------ |
| 初始化 Git 仓库                       | git init                                    |
| 添加文件/更改到暂存区                 | git add filename                            |
| 添加所有文件/更改到暂存区             | git add .                                   |
| 回退本地所有修改而未提交的文件        | git checkout .                              |
| 回退本地某一个修改而未提交文件        | git checkout filename                      |
| 把某一个提交单个 commit 内容改到 HEAD | git cherry-pick HASH1                       |
| 提交                                  | git commit -m msg                           |
| 一次性提交所有在暂存区文件            | git commit -am "msg"                        |
| 从远程仓库拉取最新代码                | git pull origin master                      |
| 推送到远程仓库                        | git push origin master                      |
| 查看配置信息                          | git config --list                           |
| 查看文件列表                          | git ls-files                                |
| 比较工作区和暂存区                    | git diff                                    |
| 比较暂存区和版本库                    | git diff --cached                           |
| 比较工作区和版本库                    | git diff HEAD                               |
| 比较两个节点的差异                    | git diff HASH1 HASH2                        |
| 比较两个分支之间的差异                | git diff master..branchA                    |
| 比较时只看文件列表                    | git diff --name-status                      |
| 获取分支代码，且不会自动合并          | git fetch origin remote_branch:local_branch |
| 查看历史                              | git log                                     |
| 从暂存区移除文件                      | git reset HEAD filename                     |
| 查看本地远程仓库配置                  | git remote -v                               |
| 硬回滚                                | git reset --hard 提交SHA                    |
| 将指定文件从当前分支的缓存区删除      | git rm filename                             |
| 移动文件/重命名文件                   | git mv pathA pathB                          |
| 强制推送到远程仓库                    | git push -f origin master                   |
| 修改上次 commit                       | git commit --amend                          |
| 从远端服务器 pull 下来自动合并        | git pull origin remote_branch:local_branch  |
| 推送 tags 到远程仓库                  | git push --tags                             |
| 常用推送命令格式                      | git push origin branch_name                 |
| 把本地分支推送到远端分支              | git push origin local_branch:remote_branch  |
| 推送单个 tag 到远程仓库               | git push origin [tagname]                   |
| 删除远程分支                          | git push origin --delete [branchName]       |
| 远程空分支（等同于删除）              | git push origin :[branchName]               |
| 显示工作区和暂存区的状态              | git status                                  |
| 查看所有分支历史                      | gitk --all                                  |
| 按日期排序显示历史                    | gitk --date-order                           |

## 分支

`git branch` 查看本地所有分支，加上 `-r` 查看远端服务器上的分支，如果想查看远端服务器和本地工程上所有分支，执行 `git branch -a` 即可。

- 新建分支：现在一般用 `git checkout -b new_branch` 新建分支了。
- 删除分支：`git branch -d` 删除，或者 `git branch -D` 删除，大写 D 是强制删除。
- 切换分支：`git checkout branch_name`，如果要强制切换，请加上 `-f`。
- 合并分支：`git merge branch_name` 或者 `git rebase branch_name`，但是后者会修改提交历史，使提交历史呈现线性。

## Q&A

### 如何解决gitk中文乱码，git ls-files 中文文件名乱码问题？

在~/.gitconfig中添加如下内容

```
[core]
   quotepath = false
[gui]
   encoding = utf-8
[i18n]
   commitencoding = utf-8
[svn]
   pathnameencoding = utf-8
```

参考 <http://zengrong.net/post/1249.htm>

或者像上文一样配置一下。

### 如何处理本地有更改需要从服务器合入新代码的情况？

```
git stash
git pull
git stash pop
```

### stash

查看 stash 列表：

```
git stash list
```

查看某一次 stash 的改动文件列表（不传最后一个参数默认显示最近一次）：

```
git stash show "stash@{0}"
```

以 patch 方式显示改动内容

```
git stash show -p "stash@{0}"
```

应用某次 stash 改动内容：

```
git stash apply "stash@{0}"
```

### 如何合并 fork 的仓库的上游更新？

```
git remote add upstream https://upstream-repo-url
git fetch upstream
git merge upstream/master
```

### 如何通过 TortoiseSVN 带的 TortoiseMerge.exe 处理 git 产生的 conflict？
* 将 TortoiseMerge.exe 所在路径添加到 `path` 环境变量。
* 运行命令 `git config --global merge.tool tortoisemerge` 将 TortoiseMerge.exe 设置为默认的 merge tool。
* 在产生 conflict 的目录运行 `git mergetool`，TortoiseMerge.exe 会跳出来供你 resolve conflict。

  > 也可以运行 `git mergetool -t vimdiff` 使用 `-t` 参数临时指定一个想要使用的 merge tool。

### 不想跟踪的文件已经被提交了，如何不再跟踪而保留本地文件？

`git rm --cached /path/to/file`，然后正常 add 和 commit 即可。

### 如何不建立一个没有 parent 的 branch？

```
git checkout --orphan newbranch
```

此时 `git branch` 是不会显示该 branch 的，直到你做完更改首次 commit。比如你可能会想建立一个空的 gh-pages branch，那么：

```
git checkout --orphan gh-pages
git rm -rf .
// add your gh-pages branch files
git add .
git commit -m "init commit"
```

### submodule 的常用命令

**添加 submodule**

```
git submodule add git@github.com:philsquared/Catch.git Catch
```

这会在仓库根目录下生成如下 .gitmodules 文件并 clone 该 submodule 到本地。

```
[submodule "Catch"]
path = Catch
url = git@github.com:philsquared/Catch.git
```

**更新 submodule**

```
git submodule update
```

当 submodule 的 remote 有更新的时候，需要

```
git submodule update --remote
```

当在本地拉取了 submodule 的远程更新，但是想反悔时：

```
git submodule update --init
```

**删除 submodule**

在 .gitmodules 中删除对应 submodule 的信息，然后使用如下命令删除子模块所有文件：

```
git rm --cached Catch
```

**clone 仓库时拉取 submodule**

```
git submodule update --init --recursive
```

### 删除远程 tag

```
git push origin --delete tag [tagname]
```

### 基于某次 commit 创建 tag

```
git tag <tag name> <commit id>
```

```
git tag v1.0.0 ef0120
```

### 清除未跟踪文件

```
git clean
```

可选项：

| 选项                    | 含义                             |
|-------------------------|----------------------------------|
| -q, --quiet             | 不显示删除文件名称               |
| -n, --dry-run           | 试运行                           |
| -f, --force             | 强制删除                         |
| -i, --interactive       | 交互式删除                       |
| -d                      | 删除文件夹                       |
| -e, --exclude <pattern> | 忽略符合 <pattern> 的文件        |
| -x                      | 清除包括 .gitignore 里忽略的文件 |
| -X                      | 只清除 .gitignore 里忽略的文件   |

### 忽略文件属性更改

因为临时需求对某个文件 chmod 了一下，结果这个就被记为了更改，有时候这是想要的，有时候这会造成困扰。

```
git config --global core.filemode false
```

参考：[How do I make Git ignore file mode (chmod) changes?](http://stackoverflow.com/questions/1580596/how-do-i-make-git-ignore-file-mode-chmod-changes)

### 忽略除某后缀名以外的所有文件

忽略除了 .c 后缀名以外的所有文件。

```
*
!*.c
!*/
```

gitignore 里，*、?、[] 可用作通配符。

### patch

将未添加到暂存区的更改生成 patch 文件：

```
git diff > demo.patch
```

将已添加到暂存区的更改生成 patch 文件：

```
git diff --cached > demo.patch
```

合并上面两条命令生成的 patch 文件包含的更改：

```
git apply demo.patch
```

将从 HEAD 之前的 3 次 commit 生成 3 个 patch 文件：

（HEAD 可以换成 sha1 码）

```
git format-patch -3 HEAD
```

生成 af8e2 与 eaf8e 之间的 commits 的 patch 文件：

（注意 af8e2 比 eaf8e 早）

```
git format-patch af8e2..eaf8e
```

合并 format-patch 命令生成的 patch 文件：

```
git am 0001-Update.patch
```

与 `git apply` 不同，这会直接 add 和 commit。

### 只下载最新代码

```
git clone --depth 1 git://xxxxxx
```

这样 clone 出来的仓库会是一个 shallow 的状态，要让它变成一个完整的版本：

```
git fetch --unshallow
```

或

```
git pull --unshallow
```

### 基于某次 commit 创建分支

```sh
git checkout -b test 5234ab
```

表示以 commit hash 为 `5234ab` 的代码为基础创建分支 `test`。

### 恢复单个文件到指定版本

```sh
git reset 5234ab MainActivity.java
```

恢复 MainActivity.java 文件到 commit hash 为 `5234ab` 时的状态。

### 设置全局 hooks

```sh
git config --global core.hooksPath C:/Users/mazhuang/git-hooks
```

然后把对应的 hooks 文件放在最后一个参数指定的目录即可。

比如想要设置在 commit 之前如果检测到没有从服务器同步则不允许 commit，那在以上目录下建立文件 pre-commit，内容如下：

```sh
#!/bin/sh

CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

git fetch origin $CURRENT_BRANCH

HEAD=$(git rev-parse HEAD)
FETCH_HEAD=$(git rev-parse FETCH_HEAD)

if [ "$FETCH_HEAD" = "$HEAD" ];
then
    echo "Pre-commit check passed"
    exit 0
fi

echo "Error: you need to update from remote first"

exit 1
```

### 查看某次 commit 的修改内容

```sh
git show <commit-hash-id>
```

### 查看某个文件的修改历史

```sh
git log -p <filename>
```

### 查看最近两次的修改内容

```sh
git log -p -2
```

### 应用已存在的某次更改 / merge 某一个 commit

```sh
git cherry-pick <commit-hash-id>
```

cherry-pick 有更多详细的用法，可以参见帮助文档。

### 命令行自动补全

在 shell 里加载 git-completion 系列脚本，详见 <https://github.com/git/git/tree/master/contrib/completion>

### 文件每一行变更明细

```sh
git blame <filename>
```

### 找回曾经的历史

```sh
git reflog
```

列出 HEAD 曾指向过的一系列 commit，它们只存在于本机，不是版本仓库的一部分。

还有：

```sh
git fsck
```

### 记住 http(s) 方式的用户名密码

在有些情况下无法使用 git 协议，比如公司的 git 服务器设置了 IP 白名单，只能在公司内网使用 ssh，那么在外面就只能使用 http(s) 上传下载源码了，但每次都手动输入用户名/密码特别惨，于是乎就记住吧。

设置记住密码（默认 15 分钟）：

```sh
git config --global credential.helper cache
```

自定义记住的时间（如下面是一小时）：

```sh
git config credential.helper 'cache --timeout=3600'
```

长期存储密码：

```sh
git config --global credential.helper store
```

### git commit 使用 vim 编辑 commit message 中文乱码

这个问题在 Windows 下出现了，没找到能完美解决的办法，一种方法是在 vim 打开后输入：

```sh
:set termencoding=GBK
```

这就有点太麻烦了，折衷的方法是改为使用 gVim 或其它你喜欢的编辑器来编辑 commit message：

```sh
git config --global core.editor gvim
```

参考：
* [How do I make Git use the editor of my choice for commits?](https://stackoverflow.com/questions/2596805/how-do-i-make-git-use-the-editor-of-my-choice-for-commits)
* [转：git windows中文 乱码问题解决汇总](http://www.cnblogs.com/youxin/p/3227961.html)

另外在升级 Vim 到 8.1 之后，由于 PATH 环境变量里加的还是 vim80 文件夹，导致 git commit 时提示：

```
error: cannot spawn gvim: No such file or directory
error: unable to start editor 'gvim'
Please supply the message using either -m or -F option.
```

使用 `which gvim` 查看：

```
$ which gvim
/usr/bin/which: no gvim in xxxxxxx
```

将 PATH 里添加的 vim80 路径改为 vim81 后解决。

### git log 中文乱码

只在 Windows 下遇到。

```sh
git config --global i18n.logoutputencoding gbk
```

编辑 git 安装目录下 etc/profile 文件，在最后添加如下内容：

```
export LESSCHARSET=utf-8
```

参考：[Git for windows 中文乱码解决方案](https://segmentfault.com/a/1190000000578037)

### git diff 中文乱码

只在 Windows 下遇到，目前尚未找到有效办法。

### git status 中文乱码

目前只在 Mac 下遇到。

```sh
git config --global core.quotepath false
```

### 统计代码行数

CMD 下直接执行可能失败，可以在右键，Git Bash here 里执行。

#### 统计某人的代码提交量

```sh
git log --author="$(git config --get user.name)" --pretty=tformat: --numstat | gawk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END { printf "added lines: %s removed lines : %s total lines: %s\n",add,subs,loc }'
```

#### 仓库提交者排名前 5

如果看全部，去掉 head 管道即可。

```sh
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```

#### 仓库提交者（邮箱）排名前 5

这个统计可能不太准，可能有同名。

```sh
git log --pretty=format:%ae | gawk -- '{ ++c[$0]; } END { for(cc in c) printf "%5d %s\n",c[cc],cc; }' | sort -u -n -r | head -n 5
```

#### 贡献者排名

```sh
git log --pretty='%aN' | sort -u | wc -l
```

#### 提交数统计

```sh
git log --oneline | wc -l
```

参考：[Git代码行统计命令集](http://blog.csdn.net/Dwarven/article/details/46550117)

### 修改文件名时的大小写问题

修改文件名大小写时，默认会被忽略（在 Windows 下是这样），让 git 对大小写敏感的方法：

```sh
git config --global core.ignorecase false
```

或者使用 `git mv oldname newname` 也是可以的。

### 修复 gitk 在 macOS 下显示模糊的问题

gitk 很方便，但是在 Mac 系统下默认显示很模糊，影响体验。

根据网上搜索的结果，解决方法有两种，我采用第一种解决，第二种未尝试。

方法一：

1. 重新启动机器，按 command + R 等 Logo 和进度条出现，会进入 Recovery 模式，选择顶部的实用工具——终端，运行以下命令：

    ```sh
    csrutil disable
    ```

2. 重新启动机器。

3. 编辑 Wish 程序的 plist，启动高分辨率屏支持。

    ```
    sudo gvim /System/Library/Frameworks/Tk.framework/Versions/Current/Resources/Wish.app/Contents/Info.plist
    ```

    在最后的 </dict> 前面加上以下代码

    ```sh
    <key>NSHighResolutionCapable</key>
    <true/>
    ```

4. 更新 Wish.app。

    ```sh
    sudo touch Wish.app
    ```

5. 再次用 1 步骤的方法进入 Recovery 模式，执行 `csrutil enable` 启动对系统文件保护，再重启即可。

参考：[Mac 中解决 gitk 模糊问题](http://roshanca.com/2017/make-gitk-retina-in-mac/)

方法二：

```sh
brew cask install retinizer
open /System/Library/Frameworks/Tk.framework/Versions/Current/Resources/
```

打开 retinizer，将 Wish.app 拖到 retinizer 的界面。

参考：[起底Git-Git基础](http://yanhaijing.com/git/2017/02/09/deep-git-4/)

### clone 时指定 master 以外的分支

```sh
git clone -b <branch name> --single-branch <repo address>
```

### 获取当前分支名称

```sh
git symbolic-ref --short -q HEAD
```

### 解决 no man viewer handled the request

运行命令 `git stash --help` 报错：

```sh
warning: failed to exec 'man': Invalid argument
fatal: no man viewer handled the request
```

原因是 Windows 下没有 man 命令。

可以修改 git 配置让命令的帮助文档通过浏览器打开。

```
git config --global help.format web
```

### 比较两个分支的差异

显示出所有差异详情：

```sh
git diff <branch_name_1> <branch_name_2>
```

显示有差异的文件列表：

```sh
git diff <branch_name_1> <branch_name_2> --stat
```

显示指定文件的差异详情：

```sh
git diff <branch_name_1> <branch_name_2> <filename>
```

查看 A 分支有，B 分支没有的提交：

```sh
git log <branch_name_A> ^<branch_name_B>
```

### git 操作时报警告

警告信息：

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@       WARNING: POSSIBLE DNS SPOOFING DETECTED!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
The ECDSA host key for gitlab.xxxx.com has changed,
and the key for the corresponding IP address 121.40.151.8
is unknown. This could either mean that
DNS SPOOFING is happening or the IP address for the host
and its host key have changed at the same time.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:bud2tDwxl9687vMOUUBGXlwZhjxDTu7eVF43ojAu1Pw.
Please contact your system administrator.
Add correct host key in /c/Users/mzlogin/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /c/Users/mzlogin/.ssh/known_hosts:1
ECDSA host key for gitlab.xxxx.com has changed and you have requested strict checking.
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

解决方案：

```
rm ~/.ssh/known_hosts
```

然后重新操作即可。

### 删除不存在对应远程分支的本地分支

（本小节有效性存疑，有时候并不好使。）

```sh
$ git remote show origin
develop                             tracked
master                              tracked
feature/new-ui                      tracked
refs/remotes/origin/feature/test    stale (use 'git remote prune' to remove)
...
```

其中 feature/test 就是不存在远程分支的本地分支。

```sh
$ git remote prune origin
```

清除完成。

### 删除已经合并的本地分支

```sh
git branch --merged | ggrep -E -v "(^\*|master|main|dev|develop|support/fat)" | xargs git branch -d
```

### 为 github 设置 SSH 代理

~/.ssh/config 文件：

```
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
    ProxyCommand nc -X 5 -v -x 127.0.0.1:54106 %h %p
```