## 一 Flink集群搭建

### 1.1 镜像下载

​    首先下载Flink镜像

```shell
docker pull flink   # 获取镜像
docker images       # 查看下载的镜像
```

### 1.2 集群搭建

  在本文中使用`Docker Compose`的方式运行一个集群：

  首先新建一个文件夹用于存放yml文件。这里我在D:\docker\flink_yml 新建一个 docker-flink 文件夹，并在该文件夹中新建一个 docker-compose.yml 文件，内容如下：

```xml
version: "2.1"
services:
  jobmanager:
    image: flink
    expose:
      - "6123"
    ports:
      - "8081:8081"
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
  taskmanager:
    image: flink
    expose:
      - "6121"
      - "6122"
    depends_on:
      - jobmanager
    command: taskmanager
    links:
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
```

创建完成，直接在该目录运行如下命令启动docker compose 即可：

```shell
docker-compose up -d
```

使用浏览器打开 `localhost:8081` 即可：



如果想要扩展可以通过如下命令：

```powershell
docker-compose scale taskmanager=<N>
```