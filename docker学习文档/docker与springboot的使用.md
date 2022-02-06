# docker与springboot的使用

## 1: docker-maven-plugin配置自动打包docker镜像

   使用的是docker-maven-plugin的打包插件，当对程序进行打包时，可以自动在远程的docker 服务器打包出相关镜像，具体的配置信息如下图所示:  **（注意：需要使用1.0以上的版本）**

### 1.1  pom文件上相关参数添加

   相关参数的解释：

  (1) executions 里面是固定写法，表示是在 package之后运行 build，来自动创建镜像

  (2) imageName:是镜像的名称，imageTag：镜像的标签值，可以写多个  forceTags：存在相同镜像是否覆盖   dockerDirectory: dockerfile文件的存放目录，如果不使用dockerfile，则可以在pom文件上配置    dockerHost:docker主机的地址,需要先打开了2375端口   

  (3) resources: 复制 jar包/zip包 到docker容器指定的目录

```
<plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.2.2</version>
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <imageName>wdw/${project.artifactId}</imageName>
                    <imageTags>
                        <imageTag>${project.dockerVersion}</imageTag>
                    </imageTags>
                    <forceTags>true</forceTags>
                    <dockerDirectory>${project.basedir}</dockerDirectory>
                    <dockerHost>http://1.1.1.29:2375</dockerHost>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.artifactId}-${project.version}.zip</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
```

### 1.2 dockerfile文件内容的编写

   因为程序的特殊性，需要把config目录下的配置文件和log日志的相关目录挂载到服务器的本地目录

```
# Dockerfile example
FROM java:8
MAINTAINER wangdongwei <wangdongwei@synway.com>
VOLUME /tmp
ADD  datastandardmanager-V1.7.0.20210129.zip /
EXPOSE 8123
ENTRYPOINT ["java", "-jar", "/datastandardmanager/datastandardmanager-V1.7.0.20210129.jar"]
```





## 2: dockerfile-maven-plugin配置自动打包docker镜像

```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.10</version>
    <executions>
        <execution>
            <id>build-image</id>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <dockerfile>${project.basedir}/Dockerfile</dockerfile>
        <repository>wdw/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>${project.artifactId}-${project.version}.jar</JAR_FILE>
            <ZIP_FILE>target/${project.artifactId}-${project.version}.zip</ZIP_FILE>
        </buildArgs>
    </configuration>
</plugin>
```



```
FROM java:8
MAINTAINER wangdongwei <wangdongwei@synway.com>
VOLUME /tmp
ADD  ${ZIP_FILE} /
WORKDIR datastandardmanager
EXPOSE 8123
ENTRYPOINT ["java", "-jar", "${JAR_FILE}"]
```

 **注意：如果再服务器上使用 docker build命令，则不能使用环境变量，必须指定好文件名。**



