## git常用命令
```
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
12. 创建切换分支 git checkout -b 分支名。 禁用 fast forward模式 git merge --no-ff -m "msg" 分支名
13. bug分支 使用 git stash储存当前工作现场 git stash list查看。恢复：git stash pop，git stash apply（恢复后stash内容不删除）需要 git stash drop
14. 删除分支 git branch -d 分支名
15. git push origin 分支名 推送分支。多人协作：git checkout -b dev origin/dev 创建远程的origin的dev分支到本地。 多人同时推送冲突： git pull 把最新的提交抓下来， 在本地合并。失败原因：没有指定本地dev分支与远程origin/dev分支的链接. git branch --set-upstream-to dev origin/dev
16. 打标签 git tag v1.0 默认打在最新的commit上。 打历史标签 git tag Vx.x commit id 查看标签 git show tagname  git push origin --tags 推送所有本地标签
17. 添加远程库git remote add origin git@github.com:taosha1/learngit.git 推送git push -u origin master  第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来
18. .gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：
--cached 后面 有个点
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
19. git飞行规则 https://github.com/k88hudson/git-flight-rules/blob/master/README_zh-CN.md
```



