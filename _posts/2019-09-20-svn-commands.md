---
title: Useful Svn Commands
categories: Tools
tags: [svn, Tools, Tips]
---
# svn

##  revert working copy to previous version

* method1

  ```shell
  svn update -r ARG
  ```

* method2

  ```shell
  svn merge -r ARG1:ARG2 .
  ```

* method3

  ```shell
  svn diff -r ARG1:ARG2 > ../diff.patch
  patch -p0 < ../diff.patch
  ```

查看某个文件中代码的修改历史：

```shell
svn blame -v file_name
```

