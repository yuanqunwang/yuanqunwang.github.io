---
title: Useful git commands
categories: tools
tags: [git, Tools, Tips]
---
# Git Commands

## Concepts

* HEAD: 指向一个命名的分支，分支则指向最新的commit，当该分支有新的提交时，分支会更新到最新的commit，同时HEAD也会指向最新的commit。

* detached HEAD：HEAD不是指向一个命名的分支，而是指向某个commit，可以在这种状态下进行编辑、提交，但是这些提交会在git的垃圾处理过程中丢失，如果要保留这些提交，需要创建一个新的分支。

  * 进入detached HEAD模式：

    ```shell
    $ git checkout 43cbcc7        # 43cbcc7代表某个commit
    ```

  * 退出detached HEAD模式：

    ```shell
    $ git checkout experimental   # experimental代表分支名称
    ```

## help

* 获得log命令的帮助

  ```shell
  $ git help log
  ```

* 显示当前项目的所有配置信息

  ```shell
  $ git config --list
  ```

* 配置个人信息

  ```shell
  $ git config --global user.name "my name"
  $ git config --global user.email "my email address"
  $ git config -e [--global] #通过编辑器配置
  ```

## 创建、提交

* 初始化git新项目

  ```shell
  $ cd myapp
  $ git init
  ```

* 在myapp编辑文件后，将其提交到index区

  ```shell
  $ git add .                # 将工作目录所有的修改文件提交到index中
  $ git add file1 file2      # 将工作目录所有文件提交到index中
  ```

* 提交commit

  ```shell
  $ git commit -m 'commit msg'
  $ git commit -a            #可以省略前面的add步骤
  ```



## Exploring history

* 显示commit历史

  ```shell
  $ git log
  ```

* 显示历史，同时输出完整的修改记录

  ```shell
  $ git log -p
  ```

* 显示commit的大体历史信息

  ```shell
  $ git log --stat --summary
  ```

* 版本、分支提交、合并信息

  ```shell
  $ git log --all --graph --oneline --decorate
  ```

* 单行显示最近3次提交

  ```shell
  $ git log -3 --pretty --oneline
  ```

* 显示HEAD~3 到HEAD之间修改过文件

  ```shell
  $ git log --name-only HEAD~3 HEAD
  ```

* 显示tag1到tag2之间的commits

  ```shell
  $ git log tag1..tag2
  ```

* 显示某个commit修改过的文件名称

  ```shell
  # 显示2ea5dcb修改文件列表
  $ git show --name-only 2ea5dcb 
  
  # 显示2ea5dcb修改文件及其状态列表
  $ git show --name-status 2ea5dcb
  ```

  

## manage branches

* 显示分支

  ```shell
  $ git branch       #显示本地分支
  $ git branch -r    #显示远程分支
  $ git branch -a    #显示本地+远程分支
  ```

* 创建分支experimental

  ```shell
  $ git branch experimental
  ```

* 切换到experimental分支

  ```shell
  $ git branch switch experimental
  ```

* 删除experimental分支

  ```shell
  $ git branch -d experimental
  ```

## tag

* 显示所有tag

  ```shell
  $ git tag
  ```

* 创建tag

  ```shell
  $ git tag tag_name commit
  ```

* 删除tag

  ```shell
  $ git tag -d tag_name
  ```

## 协作开发

* 克隆项目

  ```shell
  $ git clone https://github.com/spring-projects/spring-framework.git
  ```

  













