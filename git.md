# 基础

## 添加远程仓库

1. 生成 SSH KEY，生成目录在用户主目录的`.ssh`目录，把`id_rsa.pub`添加到GitHub的ssh

   ```
   ssh-keygen -t rsa -C "crazycs520@gmail.com"
   ```

2. 添加远程仓库

   ```
   git remote add origin git@github.com:crazycs520/util.git
   git remote add origin git@github.com:crazycs520/mit6.824-2017-golab.git
   ```

   ​


```Shell
git add
git status
git commit -m " "
git log
git log --pretty=oneline
git reset --hard HEAD^
git reset --hard ***** #commit number
git reflog 	#记录每次的命令
git checkout -- file	#丢弃工作区的修改,还没有add
git reset HEAD test.txt   #把暂存区的修改撤销掉（已经add了），重新放回工作区
```

```Shell
#创建一个`dev`分支, -b 表示创建并切换
git checkout -b dev
#相当于以下两个命令
git branch dev   # 创建dev分支
git checkout dev #切换分支
#查看当前分支
git branch
#合并分支的修改
git merge dev
#删除dev分支
git branch -d dev
#merge 冲突后中断merge
git merge --abort
#--no-ff参数就可以用普通模式合并表示禁用Fast forward,创建一个新的commit，所以加上-m参数
git merge --no-ff -m "merge with no-ff" dev
```

```shell
#
git stash

git rm -r --cached . && git add . && git commit -m 'update .gitignore'

```











```
[alias]
        co = checkout
        ci = commit
        st = status
        br = branch
        lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%C reset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --

        ls = log --stat
        lp = log -p
        la = log --pretty=\"format:%ad %h (%an): %s\" --date=short

        log-graph = log --graph --date=short --pretty=format:'%Cgreen%h %cd %Cblue%cn %Creset%s'
        log-all = log --graph --all --color --pretty='%x09%h %cn%x09%s %Cred%d%Creset'

        wc = whatchanged
        oneline = log --pretty=oneline
        ranking = shortlog -s -n --no-merges
        edit-unmerged = "!f() { git ls-files --unmerged | cut -f2 | sort -u ; }; vim `f`"
        add-unmerged = "!f() { git ls-files --unmerged | cut -f2 | sort -u ; }; git add `f`"

        gn = grep -n

[color]
    ui = auto
    branch = auto
    diff = auto
    status = auto
    interactive = auto
    grep = auto

[user]
  email = chenshuang@100tal.com
  name = chenshuang

[core]
    pager = less -FRSX
    excludesfile = /home/yuchao/.gitignore
    quotepath = false

[branch "master"]
    remote = origin
    merge = refs/heads/master

[branch "develop"]
    remote = origin
    merge = refs/heads/develop

```

