---
title: 'Jenkins拉取代码'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 16:29:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

### 前提

> 需要下载插件：Git	GitHub

![image-20220901171435157](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901171435157.png)

### 一、添加凭据

![image-20220901164225342](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901164225342.png)

>  创建 GitHub 个人访问令牌 https://zhuanlan.zhihu.com/p/401978754

### 二、新建一个任务

![image-20220901164357522](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901164357522.png)

![image-20220901164445468](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901164445468.png)

### 三、编写Pipeline script

> 可以在流水线语法中，快速生成流水线脚本

![image-20220901165019506](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901165019506.png)

![image-20220901165208796](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901165208796.png)

> 将上述生成好的流水线脚本，按照Pipeline script格式继续编写，完整版如下

![image-20220901170300400](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901170300400.png)

#### Pipeline script

```javascript
pipeline {
    agent {     // 告诉Jenkins，选择哪台节点机器去执行Pipeline代码
        label 'tenxunyun-node'
    }
    stages {
        stage('Example stage 1') {
            steps {
                git branch: 'master', credentialsId: 'Github', url: 'https://github.com/zhzhang12138/flask_script.git'
            }
        }
    }
}
```

### 四、构建

![image-20220901165810287](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901165810287.png)

![image-20220901170551381](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901170551381.png)

构建成功之后，可以去对应的`tenxunyun-node`的`/jenkins/workspace/PullCode`目录下查看代码是否成功拉取。

![image-20220901170741043](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901170741043.png)











