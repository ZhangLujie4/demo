---
title: Git学习笔记
date: 2018-04-10 22:04:00
tags: git
categories: git
---

part 1

+ mkdir demo
+ cd mkdir
+ git init
+ git status （查看修改状态） 
+ git checkout -- file (没有add 后丢弃缓存区修改)
+ git reset HEAD file   >   git checkout -- file (add 后丢弃缓存区修改)
+ git add file （会暂存修改）
+ git commit -m "123456"
+ git config user.name "XXX"
+ git config user.email "YYY"
+ git log
+ git show-branch
+ git diff file（在git add之前查看修改情况）
+ git log --pretty=oneline
+ git reset --hard HEAD^ / HEAD^^ / HEAD~100 / (commit id 只要前几位就行)
+ git rm (提交到版本库就不用担心误删，可以用git checkout -- file恢复)

github相关

+ ssh-keygen -t rsa -C "xxxxxxx@example.com"(> ~/.ssh/id_rsa.pub)

+ git remote rm origin
+ git remote add origin git@github.com:ZhangLujie4/demo.git
+ git push -u origin master(如果push不上去直接把-u改成-f， 强制上传;第一次加-u，之后就不用了)
+ 通过`ssh`支持的原生`git`协议速度最快
+ git checkout -b dev
+ git branch dev  +  git checkout dev
+ git branch    查看当前branch
+ git checkout dev/master    切换分支
+ git merge dev  合并指定分支到当前分支
+ git branch -d dev  删除分支(-D强行删除)

part 2

+ git log --graph
+ git merge --no-ff -m "message" dev
+ git log --graph --pretty=oneline --abbrev-commit
+ git stash(储存当前工作现场)
+ git stash list （查看工作现场记录）
+ git stash apply + git stash drop (恢复工作区 + 删除)
+ git stash pop （恢复&删除）

part 3

+ git remote (-v)   远程仓库信息
+ git push origin dev（branch id）
+ git pull
+ git branch --set-upstream dev origin/dev

划重点划重点划重点！！！

重要事情说三遍

如果要在同一台电脑上配置两个github账号

ssh-keygen -t rsa -C "xxxxxxx@163.com"  +  id_rsa_163

在~/.ssh下面创建config

```
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
Host my.github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_163
```

然后如果要用hexo把_config.yml的git@github.com 改成 git@my.github.com

如果要在git时忽略某个文件,例如node_modules文件夹,则先用`touch .gitignore`,然后在创建的文件里写上node_modules/

还有一些操作,看git提示或者github提示自然就明白了，有友好的提示

part **patch的应用**
```
1. 两个commit间的修改（包含两个commit）
git format-patch <r1>..<r2>

2. 单个commit
git format-patch -1 <r1>

3. 从某commit以来的修改（不包含该commit）
git format-patch <r1>
```
