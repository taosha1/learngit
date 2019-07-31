# docker
## 镜像image相关指令

- 获取镜像: docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。
仓库名：这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。
- 运行镜像: docker run -it --rm ubuntu:18.04  bash
it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。
-  列出镜像： docker image ls
列表包含了 仓库名、标签、镜像 ID、创建时间 以及 所占用的空间。
镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个 标签。
- 通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间:docker system df
- 删除本地镜像:docker image rm [选项] <镜像1> [<镜像2> ...]
- 定制镜像：尽量使用dockerfile，而不是commit
- 使用 docker exec 命令进入容器（docker exec -it webserver bash）
- 修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动
- docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
- Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。
- 用 docker history 具体查看镜像内的历史记录
- 使用 docker commit 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 黑箱镜像，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。黑箱镜像的维护工作是非常痛苦的。
### 使用 Dockerfile 定制镜像
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
 [参见](https://yeasy.gitbooks.io/docker_practice/introduction/)

