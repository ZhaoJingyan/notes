# git命令速记 #

```sh
git clone https://github.com/ZhaoJingyan/notes # 从服务器克隆的本地
git init # 在当前目录下新建仓库
git init test # 创建test仓库，相当于mkdir test;cd test; git init
git add file.txt # 将file.txt加入暂存区
git commit -m "提交信息" # 将暂存区的文件提交到本地版本库
git commit -a -m "提交信息" # 将全部追踪文件提交到本地版本库，相当于先执行git add，在执行git commit
git remote # 查看远程分支，其中origin是默认远程分支
git remote -v # 列出远程分支url
```
