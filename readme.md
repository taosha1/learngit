# demo1
1. git config --global user.name "%name%"
2. git config --global user.email "%email%"
3. a new repository -> git init 
4. git add filename  将当前目录下修改的所有代码从工作区添加到暂存区
5. git commit -m "%msg%"  将缓存区内容添加到本地仓库
6. git status  查看工作区代码相对于暂存区的差别 -> git diff 查看修改 
7. git log 显示提交日志
8. HEAD表示当前最新的提交版本 git reset --hard HEAD^ -> 回退到上一个版本 git reset --hard 版本号 回退到指定版本 git reflog 记录所有命令，查找commit id
9. git checkout -- filename 撤销工作区的修改 回到最近一次的add或commit时的状态，已经添加到暂存区，再修改，撤销后回到暂存区的状态。在工作区修改了，但未放到暂存区，撤销后回到上次commit时版本库的状态.
10. 修改只是添加到暂存区，还未commit，使用 git reset HEAD <file>  将暂存区的修改撤销掉。在使用 git checkout -- filename 清除工作区的修改. 如果已经commit，则需要回退到以前的版本，才能撤销修改。
11. 使用rm删除工作区文件后，如果不想保留本地版本仓库的该文件 使用git rm filename&& git commit. 如果误删想要恢复,使用git checkout -- filename. git checkout 作用是用版本库里的版本替换工作区的版本。即无论修改还是删除，都能还原。
