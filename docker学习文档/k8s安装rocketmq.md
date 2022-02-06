# 单机版本rocketmq安装

## 1: docker上运行

###  1.1 前期准备

  	使用docker-compose来运行docker的namesrv、brokersrv、console信息，需要先离线安装docker-compose。

1.   **下载docker-compose**

​       该程序的地址为 https://github.com/docker/compose/releases, 根据自己的平台下载对应的二进制包，之后将二进制包上传到linux服务器上。之后将命令导入到/usr/local/bin/目录下

   ```shell
   # mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
   # chmod +x /usr/local/bin/docker-compose
   ```

   测试一下，如果出现以下内容，则表示安装成功2

   ```shell
   # docker-compose -v
   docker-compose version 1.29.2, build 5becea4c
   ```



### 1.2 使用docker-compose启用rocketmq

#### 1.2.1 需要创建的目录结构

​    因为需要把容器内部的日志和存储数据等内容挂载到宿主机，所以容器内部这个目录所属的用户必须和宿主机上挂载目录所属用户名相同，具体的操作方式 [用户不同的修改方式](#1.3.1 容器内部的目录所属用户与宿主机所属用户不同)

​     先创建broker目录，然后在broker目录里面

```
drwxrwxrwx. 5 rocketmq rocketmq        43 May 17 20:59 broker
-rw-r--r--. 1 root     root          1104 May 17 21:45 docker-compose.yml
drwxr-xr-x. 5 rocketmq rocketmq         6 May 17 20:32 namesrv_logs
```



#### 1.2.2 docker-compose文件内容

```dockerfile
version: '2'
services:
  #Service for nameserver
  namesrv:
    image: rocketmq:4.3.2
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./namesrv_logs/:/home/rocketmq/logs/
    command: sh mqnamesrv
  #如果想要多个 broker ，则换个服务名，容器名，端口信息，数据卷信息即可
  broker:
    image: rocketmq:4.3.2
    container_name: rmqbroker-a
    links:
      - namesrv 
    depends_on:
      - namesrv
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    environment:
      - NAMESRV_ADDR=namesrv:9876
    volumes:
      - ./broker/logs:/home/rocketmq/logs
      - ./broker/store:/home/rocketmq/store
      - ./broker/conf/broker.conf:/opt/rocketmq-4.3.2/conf/broker.conf
    command: sh mqbroker -c /opt/rocketmq-4.3.2/conf/broker.conf
    #管控平台
  console:
    image: styletang/rocketmq-console-ng
    container_name: rocketmq-console-ng
    ports:
      - 9999:8080
    links:
      - namesrv
    depends_on:
      - namesrv
      - broker
    environment:
      - JAVA_OPTS= -Dlogging.level.root=info  -Drocketmq.namesrv.addr=namesrv:9876
      - Dcom.rocketmq.sendMessageWithVIPChannel=false
```



#### 1.2.3 相关的命令

```shell
# 查看容器的运行情况
docker-compose ps 

#创建启动容器
docker-compose up -d

#停止容器，删除挂载等内容
docker-compose down --volumes

删除所有的空闲挂载卷
# docker volume prune
```



#### 1.2.4 如何验证是否启动成功

  运行 docker-compose ps命令，出现以下情况表示程序运行成功

```shell
       Name                      Command               State                                           Ports                                         
-----------------------------------------------------------------------------------------------------------------------------------------------------
rmqbroker-a           sh mqbroker -c /opt/rocket ...   Up      0.0.0.0:10909->10909/tcp, 0.0.0.0:10911->10911/tcp, 0.0.0.0:10912->10912/tcp, 9876/tcp
rmqnamesrv            sh mqnamesrv                     Up      10909/tcp, 10911/tcp, 0.0.0.0:9876->9876/tcp                                          
rocketmq-console-ng   sh -c java $JAVA_OPTS -jar ...   Up      0.0.0.0:9999->8080/tcp    
```



### 1.3 遇到的坑

#### 1.3.1 容器内部的目录所属用户与宿主机所属用户不同

​     由于相关镜像为网上下载，容器内部创建的日志文件是rocketmq用户，宿主机里面目录权限是root用户，造成挂载一直报错。

  **解决方法：**

  1: 进去到镜像中，找到rocketmq所属的用户id值等信息，在宿主机中使用该id创建该用户，相关命令如下，之后即可将容器内部的文件挂载到宿主机上。

```shell
# useradd -u 3000 rocketmq
# chown -R rocketmq:rocketmq broker/
# chmod -R 777 broker/
```

 2：在Dockerfile中指定具体的用户信息，如何做暂定





## 2: k8s上运行

  因为是部署的rocketmq，需要配置数据卷来做持久化。这是个大问题

   

   本次部署的是一套低配置版本的rocketMq，仅启动一个nameservice和1个broker。首先需要在宿主机上创建目录，用于volumeMounts挂载目录。以基础目录 /home/ckw/rocketmq 为例子，需要在这个下面创建数据卷

```shell
cd  /home/ckw/rocketmq 
mkdir -p broker_a/logs broker_a/store  namesrv_a/logs  namesrv_a/store
useradd -u 3000 rocketmq
cd ..
chown -R rocketmq:rocketmq rocketmq
```

### 2.1 相关的deployment文件

​    我们使用在一个pod来完成rocketmq的部署，这个pod中将包含1个name server的dockercontainer 和1个name service的docker container。

**1: nameSrv的相关配置文件**

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: mqnamesrv
  namespace: datagovernance
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mqnamesrv
      project: datagovernance
  template:
    metadata:
     labels:
      app: mqnamesrv
      project: datagovernance
    spec:
      containers:
      - name: mqnamesrv
        image: 10.1.7.121/datagovernance/rocketmq:4.3.2
        command: ["sh","mqnamesrv"]
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9876
          name: web
          protocol: TCP
          hostPort: 9876
        resources:
          limits:
            cpu: 1024m
            memory: 2Gi
          requests:
            cpu: 600m
            memory: 400Mi
        volumeMounts:
          - mountPath: /home/rocketmq/logs
            name: namesrvlogs
      volumes:
      - name: namesrvlogs
        hostPath:
          path: /home/ckw/rocketmq/namesrv_a/logs
      - name: namesrvstore 
        hostPath:
          path: /home/ckw/rocketmq/namesrv_a/store
---
apiVersion: v1
kind: Service
metadata:
  name: mqnamesrv
  namespace: datagovernance
  labels:
    project: datagovernance
    app: mqnamesrv
    svc: mqnamesrv-svc
spec:
  selector:
    project: datagovernance
    app: mqnamesrv
  ports:
  - name: web
    port: 9876
    targetPort: 9876
  type: NodePort
```

**2: broker的相关配置文件**

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: mqbroker
  namespace: datagovernance  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mqbroker
      name: mqbroker
  template:
    metadata:
     labels:
      app: mqbroker
      name: mqbroker
      project: datagovernance
    spec:
      containers:
      - name: mqbroker
        image: 10.1.7.121/datagovernance/rocketmq:4.3.2
        command: ["sh","mqbroker", "-n","mqnamesrv:9876"]
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 10909
          - containerPort: 10911
        resources:
          limits:
            cpu: 1024m
            memory: 2Gi
          requests:
            cpu: 600m
            memory: 400Mi 
        volumeMounts:
          - mountPath: /home/rocketmq/logs
            name: brokerlogs
          - mountPath: /home/rocketmq/store
            name: brokerstore
      volumes:
      - name: brokerlogs
        hostPath:
          path: /home/ckw/rocketmq/broker_a/logs
      - name: brokerstore
        hostPath:
          path: /home/ckw/rocketmq/broker_a/store 
```

**3: 控制程序**

```
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: mqconsole
  namespace: datagovernance  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mqconsole
      name: mqconsole
  template:
    metadata:
     labels:
      app: mqconsole
      name: mqconsole
      project: datagovernance
    spec:
      containers:
      - name: mqconsole
        image: 10.1.7.121/datagovernance/rocketmq-console-ng
        env:
        - name: JAVA_OPTS
          value: -Dlogging.level.root=info  -Drocketmq.namesrv.addr=mqnamesrv:9876
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: web
          protocol: TCP
          hostPort: 9999 
---
apiVersion: v1
kind: Service
metadata:
  name: mqconsole
  namespace: datagovernance
  labels:
    project: datagovernance
    app: mqconsole
    svc: mqconsole-svc
spec:
  selector:
    project: datagovernance
    app: mqconsole
  ports:
  - name: web
    port: 8080
    targetPort: 9999
  type: NodePort
```

### 2.2 k8s上的运行命令

​    将已经打好的镜像包导入到k8s上，之后再改名，需要加上私有仓库的地址,其中10.1.7.121是私有仓库的地址，datagovernance是命名空间(该规范用于区分),然后再提交到私有仓库中.

```
docker load -i configserver-V1.7.0.tar

docker tag 81be6590c05a 10.1.7.121/datagovernance/configserver

docker push 10.1.7.121/datagovernance/configserver
```

 编写相关的yaml文件，用于 k8s启动镜像

```
kubectl delete -f .
kubectl apply -f .
kubectl apply -f rocketmq-broker-deployment.yaml 
```

  3: 其它需要用到的 docker命令

```
# 根据 Dockerfile 打包镜像
docker build -t gateway:V1.7.0 .
# 将镜像导出到本地
docker save -o  gateway-V1.7.0.tar gateway:V1.7.0 
# 压缩
tar -zcvf gateway-V1.7.0-20210412.tar.gz gateway

```

