# DockerFile 

## 目录

[1.docker典型结构 ](1.docker典型结构 )

[2.指令介绍](2.指令)

[3.创建镜像 ](3.创建镜像 )

[4.创建redis镜像实例](4.创建redis镜像实例)

## 1.docker典型结构 
- DockerFile分为四部分组成：基础镜像信、维护者信息、镜像操作指令和容器启动时执行指令

````
#第一行必须指令基于的基础镜像
From ubutu

#维护者信息
MAINTAINER docker_user  docker_user@mail.com

#镜像的操作指令
apt/sourcelist.list

RUN apt-get update && apt-get install -y ngnix 
RUN echo "\ndaemon off;">>/etc/ngnix/nignix.conf

#容器启动时执行指令
CMD /usr/sbin/ngnix

````

## 2.指令
### 1、From指令 
From 或者From :

DockerFile第一条必须为From指令。如果同一个DockerFile创建多个镜像时，可使用多个From指令（每个镜像一次）

### 2、MAINTAINER 
格式为maintainer ，指定维护者的信息

### 3、RUN 
格式为Run 或者Run [“executable” ,”Param1”, “param2”] 
前者在shell终端上运行，即/bin/sh -C，后者使用exec运行。例如：RUN [“/bin/bash”, “-c”,”echo hello”] 
每条run指令在当前基础镜像执行，并且提交新镜像。当命令比较长时，可以使用“/”换行。

### 4、CMD指令 
支持三种格式： 
CMD [“executable” ,”Param1”, “param2”]使用exec执行，推荐 
CMD command param1 param2，在/bin/sh上执行 
CMD [“Param1”, “param2”] 提供给ENTRYPOINT做默认参数。

每个容器只能执行一条CMD命令，多个CMD命令时，只最后一条被执行。

### 5、EXPOSE

格式为 EXPOSE […] 。

告诉Docker服务端容器暴露的端口号，供互联系统使用。在启动Docker时，可以通过-P,主机会自动分配一个端口号转发到指定的端口。使用-P，则可以具体指定哪个本地端口映射过来

例如： 
EXPOSE 22 80 8443

### 6、ENV

格式为 ENV 。 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。
例如：

```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

### 7、ADD 
格式为 ADD 。

该命令将复制指定的 到容器中的 。 其中 可以是Dockerfile所在目录的一个相对路径；也可以是一个URL；还可以是一个tar文件（自动解压为目录）。则。

### 8、COPY

格式为 COPY 。

复制本地主机的 （为Dockerfile所在目录的相对路径）到容器中的 。

当使用本地目录为源目录时，推荐使用 COPY 。

### 9、ENTRYPOINT

两种格式：

ENTRYPOINT [“executable”, “param1”, “param2”] 
ENTRYPOINT command param1 param2 （shell中执行）。 
配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

每个Dockerfile中只能有一个 ENTRYPOINT ，当指定多个时，只有最后一个起效。

### 10、VOLUME

格式为 VOLUME [“/data”] 。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

### 11、USER

格式为 USER daemon 。

指定运行容器时的用户名或UID，后续的 RUN 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如： RUN groupadd -r postgres && useradd -r -g postgres postgres 。要临时获取管理员权限可以使用 gosu ，而不推荐 sudo 。

### 12、WORKDIR

格式为 WORKDIR /path/to/workdir 。

为后续的 RUN 、 CMD 、 ENTRYPOINT 指令配置工作目录。

可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如

WORKDIR /a 
WORKDIR b 
WORKDIR c 
RUN pwd 
则最终路径为 /a/b/c 。

### 13、ONBUILD

格式为 ONBUILD [INSTRUCTION] 。

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

例如，Dockerfile使用如下的内容创建了镜像 image-A 。

[…] 
ONBUILD ADD . /app/src 
ONBUILD RUN /usr/local/bin/python-build –dir /app/src 
[…] 
如果基于A创建新的镜像时，新的Dockerfile中使用 FROM image-A 指定基础镜像时，会自动执行 ONBUILD 指令内容，等价于在后面添加了两条指令。

```
FROM image-A

#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild 。
```

## 3、创建镜像 
通过Docker Build 创建镜像。 
命令读取指定路径下（包括子目录）所有的Dockefile，并且把目录下所有内容发送到服务端，由服务端创建镜像。另外可以通过创建.dockerignore文件（每一行添加一个匹配模式）让docker忽略指定目录或者文件

格式为Docker Build [选项] 路径 
需要制定标签信息，可以使用-t选项 
例如：Dockerfile路径为 /tmp/docker_build/，生成镜像的标签为build_repo/my_images 

- 指定路径`$dudo docker build -t build_repo/my_images /tmp/docker_build/`
- 当前路径 `$ docker build -t build_repo/my_images .`
- 指定文件(/u01是路径): `$ docker build -t build_repo/my_images -f /u01/app/dockerFile/df /u01`

## 4、创建redis镜像实例
```
# 来源于哪个镜像
FROM centos
# 作者信息
MAINTAINER xiaofeng
# 将宿主机文件添加到容器中，如果是tar包,自动解压
ADD redis-4.0.9.tar /redis
# 创建镜像时执行的命令
RUN touch /var/log/1.txt
# 从镜像中创建容器时执行的命令
CMD /redis/redis-4.0.9/src/redis-server /redis/redis-4.0.9/redis.conf & /bin/bash
```
 
   
