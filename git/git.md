# 基础

## 添加远程仓库

1. 生成 SSH KEY，生成目录在用户主目录的`.ssh`目录，把`id_rsa.pub`添加到GitHub的ssh

   ```
   ssh-keygen -t rsa -C "crazycs520@gmail.com"
   ```

2. 添加远程仓库

   ```
   git remote add origin git@github.com:crazycs520/util.git
   ```

   ​

