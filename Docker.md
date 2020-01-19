# Docker

## 镜像命令

- docker search 镜像名
  
  - 作用：
    - 从仓库上搜索相关的镜像
  - 参数
    - -s Number 列出收藏数不少于指定值(Number)的镜像
    - --no-trunc 显示完整信息
    - --automated 只列出automated build类型的镜像
- docker images
  - 作用：
    - 查看本地镜像
  - 参数：
    - -a 列出本地所有镜像(含中间映像层)
    - -q 只显示镜像ID
    - --digests 显示镜像的摘要信息
    - --no-trunc 显示完整的镜像信息
- docker pull 镜像名[:Tag]
  - 作用:
    - 从仓库上拉取某个镜像，如果不写Tag默认为latest(最新版)
- docker rmi 某个镜像名ID / 镜像名:TAG
  
  - 作用：
    - 删除某个镜像
  
  - 参数：
    - -f 强制删除
    - $(docker images -qa) 全部删除，相当于子命令
- docker build [options] Path|Url|-

  - 作用：
    - 根据Dockerfile构建镜像
  - 注：
    - Path|Url|- 为构建镜像的上下文路径
  - 参数：
    - -f 指明Dockerfile文件的位置，默认为当前根目录
    - -t imageName:Tag 指明镜像名和Tag



## 容器命令

- docker run [OPTIONS] 镜像名 [COMMAND] [ARG...]
  - 作用：
    - 新建并启动容器
  - 参数：
    - --name new name 为容器指定一个名称
    - -d 后台运行容器，并返回容器ID，也即启动守护式容器
    - -i 以交互模式运行容器，通常与-t同时使用
    - -t 为容器重新分配一个伪输入终端，通常与-i同时使用
    - -P 随机端口映射 宿主机port:容器port
    - -p 指定端口映射
- docker ps
  - 作用：
    - 查看当前运行的容器
  - 参数：
    - -a 列出当前所有正在运行的容器+容器上运行过的
    - -l 显示最近创建的容器
    - -n 显示最近n个创建的容器
    - -q 静默模式，只显示容器编号
    - --no-trunc 不裁断输出
- exit
  - 作用：
    - 容器停止并退出
  - 注：
    - 在容器伪终端内输入
- ctrl + p + Q
  - 作用：
    - 容器不停止退出
  - 注：
    - 在容器伪终端内操作
- docker start 容器ID/容器名
  - 作用：
    - 启动一个容器
- docker restart 容器ID/容器名 
  - 作用：
    - 重启一个容器

- docker stop 容器ID/容器名
  - 作用：
    - 停止一个容器(相当于走正常关机流程)
- docker kill 容器ID/容器名
  - 作用：
    - 强制停止一个容器(相当于拔掉电源)
- docker rm 容器ID/容器名
  - 作用：
    - 删除容器
  - 注：
    - 虽然容器已停止，但是docker的缓存机制会保存着运行过的容器信息
  - 参数：
    - -f 强制删除，即先关再停
    - $(docker ps -qa) 及联删除全部容器
- docker logs 容器ID
  - 作用：
    - 查看容器日志
  - 参数：
    - -t 加入时间戳
    - -f 跟随最新的日志打印
    - --tail num 显示最后多少条
- docker top 容器ID
  - 作用：
    - 查看容器内运行的进程
- docker inspect 容器ID
  - 作用：
    - 查看容器内部细节
- docker exec 容器ID
  - 作用：
    - 在容器中打开终端，并且可以启动新的进程
  - 参数：
    - i -t 略
- docker attach 容器ID 命令
  - 作用：
    - 直接进入容器启动命令的终端，不会启动新的进程
- docker cp 容器ID:容器内路径 目的主机路径
  - 作用：
    - 从容器内拷贝文件到主机上
- docker commit 容器ID 要创建的目标镜像名:[标签名] 
  - 作用：
    - 提交容器副本使之成为一个新的镜像
  - 参数：
    - -m=提交的描述信息
    - -a=作者
  - 注：
    - 参数有 ‘=’



## 容器数据卷

**概念**

1. 将应用与运行中的环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
2. 容器之前希望有可能共享数据

Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了，为了能保存数据在docker中，我们使用卷

**特点**

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止

**启动方法**

1. 直接命令
   - docker run -it -v /宿主机绝对路径:/容器内目录	镜像名 (可读可写)
   - docker run -it -v /宿主机绝对路径:/容器内目录:ro	镜像名 (只可读不可写) 注：容器只可读
     - Dockerfile
       VOLUME ['/dataVolumeContainer1', '/dataVolumeContainer2']

**容器间传递共享**

1. -volumes-from
2. docker run -it --name doc2 -volumes-from doc1 镜像名
3. ***结论***：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止



## Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionsFS

1. bootfs(boot file system)主要包含bootloader和kernel，bootloader主要是引导加载kernel，linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs，这一层与我们典型的linux/unix系统是一样的，包含boot加载器和内核，当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs
2. rootfs(root file system)，在bootfs之上，包含的就是典型linux系统中的/dev /proc /bin /etc等标准目录和文件，rootfs就是各种不同的操作系统发行版，比如ubuntu、centos

## Dockerfile

## 基础知识 

1. 每条保留字指令必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. \# 表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

## docker执行Dockerfile的大致流程

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行Dockerfile中的下一条指令直到所有指令都执行完成

## 指令

- FROM <image name>(最基本的镜像是scratch)

  新的镜像基于哪个镜像来构建

- MAINTAINER <author name>
  设置该镜像的作者

- WORKDIR path
  指定RUN、CMD、ENTRYPOINT命令的工作目录

- RUN <command>
  在shell或者exec的环境下执行的命令，RUN指令会在新创建的镜像上添加新的层面

- ADD <src> <destination>
  将宿主机下的文件拷贝进镜像且自动处理URL和解压tar压缩包

- COPY <src> <destination>
  复制文件和目录到镜像中

- EXPOSE<port>
  指定容器在运行时监听的端口

- CMD
  指定容器启动时要运行的命令，Dockerfile只允许使用一次CMD指令。使用多个CMD会抵消之前所有的指令，只有最后一个指令生效。CMD会被docker run之后的参数替换。CMD有三种形式：
  CMD[“executable”， ”param1“， ”param2“]
  CMD[‘‘param1“， ”param2“]
  CMD command param1 param2

- ENTRYPOINT
  指定容器启动时要运行的命令，不会被run命令的参数替换，只会追加
ENTRYPOINT [“executable”， ”param1“， ”param2“]
  ENTRYPOINT command param1 param2
  
- ENV <key> <value>
  设置环境变量，使用键值对

- USER
  镜像运行时设置一个UID

- VOLUME [path]
  授权访问从容器内到主机上的目录
  
- ONBUILD
  当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

## 小总结

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段

* Dockerfile是软件的原材料
* docker镜像是软件的交付品
* docker容器则可以认为是软件的运行状态

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石

