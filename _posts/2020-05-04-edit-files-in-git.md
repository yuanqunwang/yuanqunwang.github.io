---
title: 管理git中编辑修改文件的一些概念和命令
tags: [git]
---

## git用到的一些概念

![](assets/images/git_lifecycle.png)

* 存档区（stash）：对某个任务A进行中的修改，还没有完善到可以提交的程度时，需要切换到另一个任务B，此时可以将对任务A的修改保存到存档区，完成任务B的开发和提交，再将存档区中任务A的修改从存档区取出，继续任务A的工作。
* 工作区（workspace）：本地检出，所有修改都是针对工作区的文件。一般，git之外的工具都是修改工作区的文件。
* 暂存区/索引（index）：工作区的修改提交到本地版本库的中转站，在工作区中文件的修改，需要将这些修改提交到暂存区，再提交到本地仓库。这样做的目的是为了防止误添加一些不必要的文件到版本库中。
* 本地版本库（local repository）：隐藏子目录`.git`，该文件夹包含本地仓库的所有信息。
* 上游版本库（upstream repository）：托管在网络上的项目版本，用于与其他开发人员共享代码。默认名称为`origin`。

## 存档区

* 将当前工作区和暂存区的修改保存到存档区

  ```shell
  $ git stash
  ```

* 将当前工作区和暂存区的修改保存到存档区，同时添加信息

  ```shell
  $ git stash push -m 'msg about stash'
  ```

* 列出所有的存档列表

  ```shell
  $ git stash list
  ```

* 显示1号存档的修改内容

  ```shell 
  $ git stash show -p 0 //show接受git diff的所有参数
  ```

* 将某个存档区中的内容pop到当前工作区

  ```shell
  $ git stash pop //默认是 pop 0，可以通过pop + 序号指定stash
  ```

## 工作区

* 从暂存区或某个commit中恢复某个文件。

  * 语法：`git checkout [<tree-ish>] [--] <pathspec>...`
  
  * 从暂存区恢复文件：

    ```shell
    $ git checkout -- hello.c
    ```
  
    表示从暂存区中恢复文件`hello.c`，`--`的作用是，当有一个分支名称也为`hello.c`时，用于区分切换分支命令：`git checkout hello.c`。
  
  * 从某个提交恢复文件：
  
    ```shell
    $ git checkout master~2 hello.c
    ```
  ```
  
  ```

## 暂存区

* 将当前工作区中的内容提交到暂存区

  ```shell
  // add 后加文件名或目录，如果是目录则将目录下的所有文件加入到暂存区
  $ git add file_name
  ```

* 将提交到暂存区中的内容删除

  ```shell
  $ git restore file_name 
  ```

* 显示文件修改差异

  ```shell
  $ git status
  ```

  status显示的内容包含：

  * untracked files: 新增加的文件，从暂存区中删除的文件，没有被git追踪记录
  * tracked files：表示之前commit记录到的文件。

  通过`git status -s`命令可以显示文件修改的列表，关于文件的修改有两列，其中左侧表示暂存区的的修改情况，右侧表示工作区中的修改情况，如下是一个示例显示：

  ```shell
  $ git status -s
   M README
  MM Rakefile
  A  lib/git.rb
  M  lib/simplegit.rb
  ?? LICENSE.txt
  ```

  其中，`README`表示该文件已经在工作区中修改，但是没有提交到暂存区。`lib/simplegit.rb`表示tracked files修改后，提交到了暂存区。`lib/git.rb`表示新增文件，且提交到暂存区。`Rakefile`表示文件修改后提交到暂存区后，又做了修改，但是之后的修改没有提交到暂存区。`LICENSE.tst`表示新增的untracked files。

* 显示工作区与暂存区之间的差异

  ```shell
  $ git diff
  // difftool可以更好显示文件之间差异
  $ git difftool
  ```

* 显示暂存区与上一次提交之间的差异

  ```shell
  $ git diff --staged
  $ git diff --cached
  ```

* 删除文件

  ```shell
  $ git rm hello.c
  ```

  这个命令是将hello.c文件从暂存区和工作区中删除。下次提交时，就会将hello.c从本地仓库中删除。

  有些不需要的文件不小心提交到版本管理，这些文件如各种IDE的工程文件，但与项目无关。我们希望将这些文件从版本库中删除，但是希望保存在本地工作区中，可以通过如下命令将文件从暂存区中删除，同时添加到`.gitingore`文件中：

  ```shell
  $ git rm --cached .idea
  ```

## 本地仓库

* 提交暂存区中的修改到本地仓库

  ```shell
  $ git commit -m 'commit from index area'
  ```

* 直接将工作区中的修改提交到本地仓库

  ```shell
  $ git commit -a -m 'commit from workspace'
  ```

* 修改上次提交

  ```shell
  $ git commit --amend
  ```

  







