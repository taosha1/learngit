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
- 创建并运行容器：
```
 docker run [optionｓ] <image:tag> <cmd>
    options:
        -i：交互式操作
        -t：终端
        -d：后台运行
        --env k=v：指定环境变量 # 多个 env 参数指定多个环境变量
        --rm：退出后删除容器
        --name：指定容器名
        -p <host_port>:<container_port>：映射端口 # 多个 p 参数指定多个端口映射
        --mount source=<volume_name>,target=<mount_point>：挂载指定数据卷到内部挂载点
                --mount type=bind,source=<host_dir>,target=<mount_point>[,readonly]：挂载指定主机目录到内部挂载点
        --network <net_name>：指定所属内部网络，网络名为 --name 指定
```
- docker run -it -v /home/haha/下载:/share microsoft/dotnet:latest /bin/bash
- 上面命令表示： 
把宿主机的/home/haha/下载目录挂载到microsoft/dotnet:latest容器的/share目录下。 
执行完上面命令进入Docker容器后，进入/share文件夹下，ls后就会看到原来宿主机下目录“/home/haha/下载”的文件。

### 使用 Dockerfile 定制镜像
1. FROM 指定基础镜像
所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定 基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。
scratch这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。
2. RUN 执行命令
RUN 指令是用来执行命令行命令的。其格式有两种：
shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。
1. 构建镜像
docker build [选项] <上下文路径/URL/->
docker build 还支持从 URL 构建，比如可以直接从 Git repo 中构建：
docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
```
Dockerfile 制作镜像:
- 命令
    FROM 指定基础镜像：
    # FROM scratch 表示不以任何系统为基础，适用于静态可执行文件

    RUN 执行命令：
    # 每一次 RUN 都会创建一层，注意尽量合并操作到一个 RUN；注意清理无用文件、缓存等，防止镜像臃肿
    
    COPY 复制文件：
    # eg. COPY <src> <dst>
    # <dst> 可以是容器内的绝对路径,也可以是相对于工作目录的相对路径(工作目录可以用 WORKDIR 指令来指定)。目标路径不需要事先创建,如果目录不存在会在复制文件前先行创建缺失目录。
 
    ADD 高级文件复制：
    # 不推荐使用，请用 COPY + RUN
    
    CMD 容器启动命令：
    # CMD <cmd>
    # 指定默认的容器主进程的启动命令
    
    ENTRYPOINT 入口点：
    # 指定 ENTRYPOINT 之后 CMD 会变为 ENTRYPOINT 的参数
    
    ENV 环境变量：
    # ENV k1=v1 k2=v2 ...
    
    ARG 构建参数：
    # ARG k1=v1 ...
    # 构建时环境变量，成镜像后不会保存
    
    VOLUME 定义匿名卷：
    # VOLUME ["<path1>", "<path2>"...]
    # VOLUME <path>
 
        EXPOSE 声明端口：
    # EXPOSE <port1> <port2> ...
    # 仅为声明，不会真正映射端口
    
- 构建镜像：
        # 在 Dockerfile 所在目录执行
        docker build [options] <上下文路径/URL>
    options:
        -t：指定镜像名称
        
    # 可用 .dockerignore 文件指定不希望打包的文件
    
    # 使用 git repo 构建
    docker build <git_url>
    
    # 使用 tar 压缩包构建
    docker build <tar_url>
 
数据卷
- 创建数据卷
        docker volume create <volume_name>
- 删除数据卷
        docker volume rm <volume_name>
- 清理无主数据卷
        docker volume prune
- 查看所有的数据卷
        docker volume ls
- 查看数据卷信息
        docker volume inspect <volume_name>
    
容器互联网络
- 创建新网络
        docker network create -d bridge <net_name>
- 容器动态加入网络
        docker network connect <net_name> <container_name>
- 容器动态断开网络
    docker network disconnect <net_name> <container_name>
    
技巧：
- 新建/替换容器的挂载
        先 commit 旧容器为新镜像，再通过新镜像创建新容器，期间可设置挂载，删除旧容器后将新容器 rename 为旧名字。
```
 [参见](https://yeasy.gitbooks.io/docker_practice/introduction/)

