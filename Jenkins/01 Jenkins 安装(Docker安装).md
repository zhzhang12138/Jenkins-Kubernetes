---
title: 'Jenkins安装(Docker安装)'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 13:19:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

## 一、获取 `Jenkins` 镜像

#### **1、搜索 Jenkins 镜像**

```bash
docker search jenkins
---------------------
NAME                DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
jenkins             DEPRECATED; use "jenkins/jenkins:lts" instead   5441      [OK]
jenkins/jenkins     The leading open source automation server       2936
jenkinsci/jenkins   Jenkins Continuous Integration and Delivery …   396
```

这里，我们选择第二个版本的镜像，因为这个是 *Jenkins* 官网里面推荐的 *Docker* 镜像，同时第一个也提示了我们已经废弃了。

#### **2、拉取 Jenkins 镜像**

```bash
docker pull jenkins/jenkins
```

该命令直接拉取的最新版本（latest）的镜像，我们还可以选择下面几个推荐的版本：

* jenkins/jenkins:lts-jdk11：基于 JDK11 的最新 LTS 版本；
* jenkins/jenkins:alpine：Alpine 版本；
* jenkins/jenkins:latest-jdk8：基于 JDK8 的最新版本；

更多 *TAG* 版本的 *Jenkins* 可以查看 *Docker Hub* 官网：https://registry.hub.docker.com/r/jenkins/jenkins/tags

## 二、运行 `Jenkins` 容器

#### **1、创建 Jenkins 挂载目录**

```bash
mkdir -p /jenkins
chmod 777 /jenkins
```

**注意：** 创建挂载目录的同时要给该目录配置权限 777，如果权限不足的话，到时进行目录挂载的时候会失败导致无法启动 *Jenkins* 容器。

#### **2、创建并启动 Jenkins 容器**

```bash
docker run -d \
    -p 8888:8080 \
    -v /jenkins:/var/jenkins_home \
    -v /etc/localtime:/etc/localtime \
    --restart=always \
    --name=jenkins \
    jenkins/jenkins
```

* `-d`：后台运行容器；
* `-p 8888:8080`：将容器的 8080 端口映射到服务器的 8888 端口；
* `-p 50000:50000`：将容器的 50000 端口映射到服务器的 50000 端口；
* `-v /usr/local/jenkins:/var/jenkins_home`：将容器中 Jenkins 的工作目录挂载到服务器的 /usr/local/jenkins；
* `-v /etc/localtime:/etc/localtime`：让容器使用和服务器同样的时间设置；
* `--restart=always`：设置容器的重启策略为 Docker 重启时自动重启；
* `--name=jenkins`：给容器起别名；

#### **3、查看是否启动成功**

查看是否在运行：

```bash
docker ps -l
------------
CONTAINER ID   IMAGE             COMMAND                  CREATED       STATUS       PORTS                                              NAMES
9bbc86aa0358   jenkins/jenkins   "/sbin/tini -- /usr/…"   4 hours ago   Up 4 hours   0.0.0.0:8888->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
```

查看启动日志：

```bash
docker logs jenkins
-------------------
Running from: /usr/share/jenkins/jenkins.war
webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")
2022-03-16 03:49:09.190+0000 [id=1] [INFO] org.eclipse.jetty.util.log.Log
```

配置好后，访问 *Jenkins* 页面，地址为：**IP + 容器的8080端口所映射到服务器上的端口**

![image-20220901133025572](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901133025572.png)

管理员的初始密码在 *Jenkins* 的工作目录下：`/jenkins/secrets/initialAdminPassword`，我们可以进容器内部去查看，也可以在我们挂载的目录下查看：

```bash
cat /jenkins/secrets/initialAdminPassword
```

在下一个插件安装页面上，我们选择安装推荐的插件即可，下面是推荐的插件安装页面：

![image-20220901133102429](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901133102429.png)

安装完成后，会进入管理员创建页面，可以选择**使用admin账户继续**，也可以创建一个新的管理员用户（*建议创建新的管理员用户，方便管理账号密码*）：

![image-20220901133125593](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901133125593.png)

完成之后就是欢迎界面了：

![image-20220901133144266](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901133144266.png)