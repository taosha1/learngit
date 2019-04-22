# demo1
1. git config --global user.name "%name%"
2. git config --global user.email "%email%"
3. a new repository -> git init 
4. git add filename  将当前目录下修改的所有代码从工作区添加到暂存区
5. git commit -m "%msg%"  将缓存区内容添加到本地仓库
6. git status  查看工作区代码相对于暂存区的差别 -> git diff 查看修改 
7. git log 显示提交日志
8. HEAD表示当前最新的提交版本 git reset --hard HEAD^ -> 回退到上一个版本 git reset --hard 版本号 回退到指定版本 git reflog 记录所有命令，查找commit id


