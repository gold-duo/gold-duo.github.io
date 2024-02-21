---
title: Git 常用命令速查表

date: 2021-8-31 22:24
categories: [other,git]
tags: [git,manual,手册 ] 
---

## 一些默认约定

词|描述
---|---
master |默认开发分支 
Head|默认开发分支
origin |默认远程版本库
Head^ |Head 的父提交

## 命令

作用|命令|描述
---|---|---
创建版本库|1.git clone \<url\> <br/>2.git init| 1.克隆远程版本库<br/>2.初始化本地版本库
修改和提交|1.git status <br/>2.git diff <br/>3.git add<br/>4.git add \<file\> <br/><br/>5.mv \<old\> \<new\><br/>6.rm\<file\><br/>7.rm --cached \<file\><br/>8.commit -m "commit message"<br/>9.git commit --amend|1.查看状态<br/>2.查看变更内容<br/>3.跟踪所有改动<br/>4.跟踪指定的文件过的文件<br/><br/>5.文件改名<br/>6.删除文件<br/>7.停止跟踪文件但不删除<br/>8.提交所有更新过的文件 <br/>9.修改最后一次提交
 查看提交历史|1.git log<br/>2.git log -p \<fite\><br/>3.git btame \<file\> |1.查看提交历史<br/>2.查看指定文件的提交历史<br/>3.以列表方式查看指定文件的提交历史
撤消|1.git reset --hard HEAD<br/>2.git checkout HEAD \<file\><br/>3.git revert\<commit\>|1.撤消工作自录中所有未提交文件的修改内容<br/>2.撤消指定的未提交文件的修改内容<br/>3.撤消指定的提交
分支与标签|1.git branch<br/>2.git checkout \<branch/tag\><br/>3.git branch \<new-branch\><br/>4.git branch -d \<branch\><br/><br/>5.git tag<br/>6.git tag \<tagname\><br/>7.git tag -d \<tagname\> | 1.显示所有本地分支<br/>2.切换到指定分支或标签<br/>3.创建新分支<br/>4.删除本地分支<br/><br/>5.列出所有本地标签<br/>6.基于最新提交创建标签<br/>7.删除标签
合并与衍合|1.git merge \<branch\><br/>2.git rebase \<branch\>|1合并指定分支到当前分支<br/>2.衍合指定分支到当前分支
远程操作|1.git remote -v<br/>2.git remote show \<remote\><br/>3.git remote add \<remote\> \<urt><br/>4.git fetch \<remote\><br/><br/>5.git pull \<remote\>  \<branch\><br/>6.git push \<remote\>  \<branch\><br/>7.git push \<remote\> :\<branch/tag-name\><br/>8.git push --tags: |1.查看远程版本库信惠<br/>2.查看指定远程版本库信息<br/>3.添加远程版本库<br/>4.从远程库获取代码<br/><br/>5.下载代码及快速合<br/>6.上传代码及快速合井<br/>7.删除远程分支或标签<br/>8.上传所有标签


