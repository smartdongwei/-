---
title: 如何搭建博客
tags: [文档基础]
date: 2019-10-10 11:00:00
---



# [搭建好看的静态博客（使用Hexo进行搭建）](https://www.cnblogs.com/NinWoo/p/9649162.html)

经常看到大牛的博客非常的高大帅气，虽然我很渣，但是逼格不能输，所以有了以下的搭建记录。

我的成果[smartdongwei](https://smartdongwei.github.io/),喜欢的可以参考下面的记录一起来动手搞起来。

## 安装Git Bash

访问[git]( https://npm.taobao.org/mirrors/git-for-windows/v2.30.1.windows.1/Git-2.30.1-64-bit.exe)下载最新版本的git bash

安装完成之后，右键打开git bash,设置用户名和邮件信息

```
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

生成ssh秘钥文件

```
ssh-keygen -t rsa -C "GitHub注册邮箱"
```

直接三个回车即可，默认不需要设置密码。

查看生成的公钥

```
cat ~/.ssh/id_rsa.pub
```

复制内容，打开[github setting keys](https://github.com/settings/keys)界面，创建新的SSH key,并粘贴公钥到Key输入框中。

在git bash中测试是否配置成功

```
ssh git@github.com
```

出现：

```
PTY allocation request failed on channel 0
Hi Ninwoo! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

证明设置成功。

## 创建GitHub仓库

创建新的仓库，仓库名为[用户名].github.io,用户名用Github名称代替。

这样做的主要目的就是为了以后可以通过该网址进行访问你的博客主页，所以要仔细设置。在这里，我配置成smartdongwei.github.io
以下的配置中，也需要对应修改为你自己的仓库名。

## 安装Node.js

访问[Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download/)官网下载安装包。

下载完成后，点击安装。打开CMD查看是否安装成功。

```
C:\Users\ljo04>node -v
v8.12.0

C:\Users\ljo04>npm -v
6.4.1
```

如果上述命令均正常通过，则安装完成。

## 

## 安装Hexo

1.创建一个新的文件夹作为Hexo的开发目录，这里我命名为Hexo_project

```
E:\project>mkdir Hexo_project
```

2.使用npm安装Hexo

为了提高安装速度，先配置淘宝npm镜像

```
E:\project\Hexo_project>npm config set registry https://registry.npm.taobao.org
```

3.安装Hexo

```
E:\project\Hexo_project>npm install -g hexo-cli
```

4.初始化hexo

```
E:\project\Hexo_project>hexo init blog
INFO  Cloning hexo-starter to E:\project\Hexo_project\blog
Cloning into 'E:\project\Hexo_project\blog'...
remote: Counting objects: 68, done.
remote: Total 68 (delta 0), reused 0 (delta 0), pack-reused 67
Unpacking objects: 100% (68/68), done.
Submodule 'themes/landscape' (https://github.com/hexojs/hexo-theme-landscape.git) registered for path 'themes/landscape'
Cloning into 'E:/project/Hexo_project/blog/themes/landscape'...
remote: Counting objects: 838, done.
remote: Compressing objects: 100% (6/6), done.
Receiving objects:  26% (222/838), 292.01 KiB | 88.00 KiB/s
```

这个过程可能需要等待一阵时间

5.测试站点是否创建成功

```
# 创建一篇博客test
E:\project\Hexo_project\blog>hexo n test

# 生成博客
E:\project\Hexo_project\blog>hexo g

# 启动服务器预览
E:\project\Hexo_project\blog>hexo s
```

这时，可以打开浏览器访问http://localhost:4000/，查看blog界面，发现已经创建好新的文章test。

## 推送至网站

1.修改blog配置文件`E:\project\Hexo_project\blog\_config.yml`

```
deploy:
  type: git
  repo: git@github.com:smartdongwei/smartdongwei.github.io.git
  branch: master
```

> 

注意：这里repo要选择ssh的git库链接，否则会在部署的时候报错

2.安装Git部署插件

```
E:\project\Hexo_project\blog>npm install hexo-deployer-git --save
```

3.部署博客

```
E:\project\Hexo_project\blog>hexo clean
E:\project\Hexo_project\blog>hexo g
E:\project\Hexo_project\blog>hexo d
```

4.测试是否部署成功

现在访问https://smartdongwei.github.io/，如果出现blog界面这证明部署成功。

## 更换主题

如果觉得默认主题实在是太丑，可以更换其他[主题](https://hexo.io/themes/),下面的教程中，我选择Next主题。

1.下载主题

```
E:\project\Hexo_project\blog> git clone https://github.com/theme-next/hexo-theme-next themes/next
```

2.打开配置文件`E:\project\Hexo_project\blog\_config.yml`更换主题

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
plugins: hexo-generate-feed
```

3.重新部署blog

```
E:\project\Hexo_project\blog>hexo clean
E:\project\Hexo_project\blog>hexo g
E:\project\Hexo_project\blog>hexo d
```



## 相关插件的url地址

1：[hexo的中文文档](https://hexo.io/zh-cn/docs/)

2:   [Fluid](https://hexo.fluid-dev.com/docs/)









