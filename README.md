#1git常用命令

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
```
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
## git飞行规则 https://github.com/k88hudson/git-flight-rules/blob/master/README_zh-CN.md
```
#2 linux常用指令
netstat -an   查看本机所有网络连接 
lsof -i:8080 查看那个进程占用了 8080 端口
ipconfig   ➜ 本机IP 掩码 网关 
ping       ➜ 测试连通性 
nslookup   ➜ 测试是否能解析某个网址. 用来判断 DNS 是否可用!
traceroute ➜ 路由追踪 , 可用MTR, 用来判断那个节点出错了.
arp -a     ➜ 查看 arp 表是否正常.  有没有被 ARP 攻击! 
#3 docker指令

##镜像image相关指令
```
-获取镜像: docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。
仓库名：这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。
-运行镜像: docker run -it --rm ubuntu:18.04  bash
it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。
-列出镜像： docker image ls
列表包含了 仓库名、标签、镜像 ID、创建时间 以及 所占用的空间。
镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个 标签。
-通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间:docker system df
-删除本地镜像:docker image rm [选项] <镜像1> [<镜像2> ...]
-定制镜像：尽量使用dockerfile，而不是commit
-使用 docker exec 命令进入容器（docker exec -it webserver bash）
-修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动
-docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。
-用 docker history 具体查看镜像内的历史记录
使用 docker commit 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 黑箱镜像，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。黑箱镜像的维护工作是非常痛苦的。
###使用 Dockerfile 定制镜像
1. FROM 指定基础镜像
所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定 基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。
scratch这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。
2. RUN 执行命令
RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：
shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。
3. 构建镜像
docker build [选项] <上下文路径/URL/->
docker build 还支持从 URL 构建，比如可以直接从 Git repo 中构建：
docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1









```

 
