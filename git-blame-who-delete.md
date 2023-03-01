---
title: Git定位哪次commit删除某行（谁删除了我的代码）
date:  2023-02-11 15:30:56
---

首先要说的是git blame在定位一个文件中某些行是由删除时起不到任何作用。如官方文档所述:

The report does not tell you anything about lines which have been deleted or replaced; you need to use a tool such as git diff or the "pickaxe" interface briefly mentioned in the following paragraph.  from : https://git-scm.com/docs/git-blame

通过google、stackoverflow找到如下两种方法，其实是一种思想的两种实现变体，即都通过git log搜索历史提交中包含的删除行的内容，来定位目标行是被哪次commit删除了。

首先假设被删除的行中包含内容："XXXXX" 

``` bash
git log -S  "XXXXX"  -p  [--]  <fileName> 
````

意思是在所有的提交内容中查询包含"XXXXX"的提交记录，至少查询到两个一个是新增时commitId,另一个是删除时的commitId 。通过查询出来的commitId，再使用git show commitId 查看对应某次commit的具体内容，来确定是否是这次删除了目标行。

``` bash
git log  -p  [--] <fileName> | less
```
意思是先通过-p参数查询出这个文件所有的commit的内容，再通过less命令 /XXXXX 来搜索所有的commit的内容中包含目的行的内容，通过对比就能定位到是哪次删除了目标行。