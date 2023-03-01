---
title: git单次操作revert分支所有修改内容
date:  2023-01-02
---

> 假设分支名称：Feature01 基础分支：master

### 文件还原&删除

将Feature分支上所有文件修改还原成master分支一样

``` bash
git checkout <master latest commitID> .
```

通过比较与master版本的文件差异，提取新增的文件，然后删除

``` bash
git diff --name-status origin/master..HEAD|awk '{if($1=="A") print $2}'|xargs rm
```
### 提交修改

将修改内容提交，产生一个新的commitID,就相当于单次revert掉了之前分支上所有的commit.
``` bash
git add .
git commit -m "revert commit"
```
