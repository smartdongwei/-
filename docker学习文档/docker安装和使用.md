## docker安装和使用

###  1：离线包安装docker

  下载好docker所需要的3个deb文件，containerd.io_1.4.3-1_arm64.deb   docker-ce_18.09.0~3-0~debian-stretch_arm64.deb   docker-ce-cli_18.09.0~3-0~debian-stretch_arm64.deb 

  然后使用 以下命令来安装docker-ce的内容

```
sudo dpkg -i *.deb
```

2: docker的相关操作

修改配置后启动docker

systemctl daemon-reload

systemctl start docker



systemctl restart docker

#### 1.2私有仓库的创建

```
docker run -d -v /home/synway/registry:/var/lib/registry -p 5000:5000 --restart=always --name=registry arm64v8/registry:latest
```

   信任私有仓库

#### 1.3 :arm架构中没有java8的镜像，如何办？





### 2：镜像相关操作

1：导入镜像

```
docker load -i zookeeper.tar
```

2：列出镜像

```
docker image ls
```

3:删除镜像

```
docker images rm  镜像id
```

4:from指定基础镜像

  所谓定制镜像，是以一个镜像为基础，在其上进行定制。因此一个 dockerfile中from是必备的指令，且必须是第一条指令。

from是用来指定基础镜像，run指定是用来执行命令行命令的。

其格式有两种：

- shell格式： **RUN	<命令>**，就像直接在命令行中输入的命令一样。刚才写的

  ​    Dockerfile中的 RUN指令就是这种格式。

```
RUN	echo	'<h1>Hello,	Docker!</h1>'	>	/usr/share/nginx/html/index
.html
```

- exec	格式： 	RUN   ["可执行文件",	"参数1",	"参数2"]	，这更像是函数调用中

  的格式。

```
FROM nginx
RUN echo '<h1>Hello,Docker!</h1>'	>	/usr/share/nginx/html/index
.html
```

创建的时候有层数限制，不得超过127层。以下为正确的命令例子：

```
FROM	debian:jessie
RUN	buildDeps='gcc	libc6-dev	make'	\
&&	apt-get	update	\
&&	apt-get	install	-y	$buildDeps	\
&&	wget	-O	redis.tar.gz	"http://download.redis.io/releases/r
edis-3.2.5.tar.gz"	\
&&	mkdir	-p	/usr/src/redis	\
&&	tar	-xzf	redis.tar.gz	-C	/usr/src/redis	--strip-component
s=1	\
&&	make	-C	/usr/src/redis	\
&&	make	-C	/usr/src/redis	install	\
&&	rm	-rf	/var/lib/apt/lists/*	\
&&	rm	redis.tar.gz	\
&&	rm	-r	/usr/src/redis	\
&&	apt-get	purge	-y	--auto-remove	$buildDeps
```



### 3：构建镜像

  先创建dockerfile这个文件，然后在这个文件所在目录执行以下命令

#### 3.1 copy指令

适用场景：所有文件复制需要使用的命令



#### 3.2 add指令

​    如果<源路径>为一个 tar压缩文件的话，压缩格式为gzip,bzip2以及xz的情况下， ADD指令将会自动解压缩这个压缩文件到<目标路径>去。适用场景：需要自动解压缩的地方。

```
  FROM	scratch
    ADD	 ubuntu-xenial-core-cloudimg-amd64-root.tar.g
```



#### 3.3 CMD容器启动命令

  **CMD**指令的格式和**RUN**相似，也是两种格式：

- **shell格式**： **CMD**	<命令>
  - **exec格式**： **CMD**["可执行文件","参数1",	"参数2"...]
  - **参数列表格式**： CMD["参数1","参数2"...]。在指定了 ENTRYPOINT指令后，用CMD指定具体的参数。

   容器就是进程，在启动容器的时候，需要指定所运行的程序及参数，cmd指令就是用于指定默认的容器主进程的启动命令。



### 4：操作容器

#### 4.1 启动

   启动容器有两种方式，一种是基于镜像新建一个容器并启动，另一个是在终止状态的容器重新启动。

新建并启动    docker run ,在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从共有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
  - 从地址池配置一个ip	地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

##### 4.1.1 启动终止容器

   可以使用以下命令，将一个已经终止

```
 docker container start
 docker container restart
```

  后台运行，可以通过 添加  **-d** 参数来实现。 输出结果可以用 docker logs查看

  使用  -d	 参数启动后会返回一个唯一的id，也可以通过docker  container  ls 命令来查看容器信息。



  要获取容器的输出信息，可以通过与以下命令获取：

```
  docker container logs [container ID or NAMES]
```



  终止容器：

```
   docker container stop zk1
```



##### 4.1.2 进入容器

  在使用 **-d** 参数时，容器启动后就会进入后台

  某些时候需要进入容器进行操作，包括使用 **docker  attach** 命令或 **docker  exec** 命令。

**1：exec 命令   -i -t 参数**

   docker	 exec 后边可以跟多个参数，这里主要说明	  **-i	 -t** 参数。
只用  -i 参数时，由于没有分配伪终端，界面没有我们熟悉的	Linux命令提示符，但命令执行结果仍然可以返回。
当  -i   -t	参数一起使用时，则可以看到我们熟悉的	Linux 命令提示符。
   如果输入exit，则会退出，但是不会导致容器的停止。



##### 4.1.3 导出和导入容器

  **1：导出容器**

  如果要导出本地某个容器，可以使用 docker export 命令，将导出容器快照到本地文件。

```
docker	container	ls	-a


docker	export	7691a814370e	>	ubuntu.tar

```



**2：导入容器快照**

可以使用	 	 docker	import		从容器快照文件中再导入为镜像，例如

```
cat	ubuntu.tar	|	docker	import	-	test/ubuntu:v1.0
```

 

 删除容器

```
docker	container	rm		trusting_newton
```



##### 4.1.4  zookeeper部署

```
docker run -d -p 2181:2181 -v /home/synway/zk_docker_cluster/:/data/ --name=zk3 --privileged arm64v8/zookeeper
```

```
docker exec -it zk3 /bin/bash

cd bin

./zkCli.sh
```



### 5：数据卷

  数据卷是一个可供一个或多个容器使用的特殊目录，有以下有用的特性：

- **数据卷**可以在容器之间共享和重用
- 对**数据卷**的修改会立马生效
  - 对**数据卷**的更新，不会影响镜像
- **数据卷**默认会一直存在，即使容器被删除

   **注意：  数据卷的使用，类似于Linux下对目录或文件进行mount，镜像中的**
   **被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷。**



#### 5.1 创建一个数据卷

```
docker	volume	create	my-vol
```

1： 查看所有的***数据卷***

```
docker	volume	ls
```

2： 在主机里使用以下命令可以查看指定 *数据卷*的信息

```
 docker	volume	inspect	my-vol
```

3： 启动一个挂载数据卷的容器

   在用 *docker	 run*	命令的时候，使用	*--mount* 标记来将 *数据卷*挂载到容器里。在一次 **docker	run**	中可以挂载多个数据卷	。
  下面创建一个名为***web***的容器，并加载一个***数据卷***到容器的/webapp目录。

```
docker	run	-d	-P	\
--name	web	\
# -v	my-vol:/wepapp	\
--mount	source=my-vol,target=/webapp	\
training/webapp	\
python	app.py
```

4：查看数据卷的具体信息

```
docker	inspect	web
```

5：删除数据卷

```
docker	volume	rm	my-vol
```

​    ***数据卷***是被设计用来持久化数据的，它的生命周期独立于容器，Docker不会在容器被删除后自动删除***数据卷***，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的***数据卷***。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用  ***docker  rm   -v***	这个命令。

  无主的数据卷可能会占据很多空间，要清理请使用以下命令

```
docker	volume	prune
```

#### 5.2 挂载主机目录

  挂载一个主机目录作为数据卷，使用 ***--mount***	标记可以指定挂载一个本地主机的目录到容器中去。

```
docker	run	-d	-P	\
--name	web	\
-v	/src/webapp:/opt/webapp	\
--mount	type=bind,source=/src/webapp,target=/opt/webapp	\
training/webapp	\
python	app.py
```

​    上面的命令加载主机的 ***/src/webapp*** 目录到容器的 ***/opt/webapp***目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用	 	 ***-v***	参数时如果本地目录不存在	Docker	会自动为你创建一个文件夹，现在使用  ***--mount***参数时如果本地目录不存在，**Docker**会报错。



### 6：使用网络

***Docker***允许通过外部访问容器或容器互联的方式来提供网络服务。

####   6.1 外部访问容器

   容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过  **-P** 或**-p**参数来指定端口映射。
当使用 **-P**标记时，**Docker**会随机映射一个 **49000~49900** 的端口到内部容器开放的网络端口。

####   6.2 映射所有接口地址

使用  **hostPort:containerPort** 格式本地的	5000端口映射到容器的5000端口，可以执行

```
docker	run	-d	-p	5000:5000	training/webapp	python	app.py
```

####   6.3 映射到指定地址的指定端口

可以使用	**ip:hostPort:containerPort**格式指定映射使用一个特定地址，比如***localhost***地址***127.0.0.1***

```
docker	run	-d	-p	127.0.0.1:5000:5000	training/webapp	python	app.py
```

####   6.4 查看映射端口配置

  使用 ***docker port*** 来查看当前映射的端口配置，也可以查看到绑定的地址。

```
docker	port  nostalgic_morse	5000
127.0.0.1:49155.
```







### 7: 进入正在运行的容器中

  容器没有创建

```
docker run -i -t arm64_my:8 /bin/bash
```

  容器已经创建

```
docker exec -it  ecdefac8d55a /bin/bash 
```

 当exit退出容器之后如何再次进入已经停止的容器

```
docker start -ia 24f01b607a06
```



### 8: 启动镜像

  (1) 只能启动一个镜像    --network=host 这个的意思是docker里面使用是宿主机的ip地址 

```shell
docker run -di -p 8123:8123 --network=host  --name=datastandardmanager datastandardmanager:V1.7.0.20210129
```



### 9: 自定义生成java镜像

```shell
vi ~/.bashrc

# 然后把java路径写入到文件中
source ~/.bashrc
```

