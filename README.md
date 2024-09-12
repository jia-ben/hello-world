# hello-world
这是jia-ben的第一个存储库，Repository Name = hello-world.

# Git教程

## 开始
### 安装Git

安装windows上的git，检查安装结果
```
$ git -v
$ git --version
```

### 初始配置：
使用 `git config`来配置环境

> config 的配置变量有三个位置,分别使用
> ` git config --system`
> ` git config --global`
> ` git config --local`

通过 `$ git config --list --show-origin` 查看完整配置。

### 帮助信息

使用help获取帮助手册
```
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```
或者使用`-h` 获取快速参考
` $ git add -h`

## Git基础操作

### 仓库

1. 在本地初始化一个仓库使用`git init`
2. 拷贝已经存在的仓库 `git clone <url>`
    > clone 会复制仓库中的所有文件。
    >
    > clone 时更改本地仓库名`git clone <url> newname`

```
$ cd folder
$ git init

$ git clone https://github.com/jia-ben/hello-world mylib
```

### 工作目录

**工作目录中**的文件是仓库的副本。其中的文件分为 *已跟踪* 和 *未跟踪*，未跟踪文件不会被Git管控。已跟踪文件分为 未修改，已修改，已暂存。

当你在工作目录修改文件，会产生被Git标记为 *已修改* 的文件，你可以选择性地将修改过的文件放入暂存区，并 **提交** 所有已暂存的修改，循环往复。

使用 `git status` 检查工作目录中的文件状态。

```
$ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    nothing to commit, working directory clean
```

### add

`git add`指令可以：
- 跟踪新文件
- 将*已修改*文件加入*暂存区*。
- 将合并时有冲突的文件 标记为*已解决*。

```
$ git add new_file
$ git add modified_file
$ git add conflict_file
```
### 状态

status可以看工作目录的状态信息，使用`-s`简化该信息。
```
$ git status
$ git status -s
$ git status --short
```

### 忽略文件

有时候一些中间编译文件不需要被Git跟踪，通过创建一个名为`.gitignore`的文件，列出要忽略的文件的模式。
```
# 忽略所有的 .a 文件
*.a
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO
# 忽略任何目录下名为 build 的文件夹
build/
# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pd
```
> .gitnore规范:
> 
> 所有空行或者以 # 开头的行都会被 Git 忽略。
> 
> 可以使用标准的 glob 模式匹配(shell使用的简化正则表达式)，它会递归地应用在整个工作区中。
> 
> 匹配模式可以以（/）开头防止递归。
> 
> 匹配模式可以以（/）结尾指定目录。
> 
> 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反

> 根目录下的.ignore文件递归到整个仓库，子目录下的.ignore只对子目录有效。

### 查看修改

```
#查看工作目录的文件和暂存区的差异
$ git diff
#查看暂存区文件和最后一次提交的差异
$ git diff --staged
```

### 提交修改

```
#使用add暂存
    $ git add file
#使用status检查暂存状态
    $ git status
#使用commit提交,并启用编辑器编写说明
    $ git commit
```
