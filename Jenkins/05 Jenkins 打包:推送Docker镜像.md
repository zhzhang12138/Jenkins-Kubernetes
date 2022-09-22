---
title: 'Jenkins打包/推送Docker镜像'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 17:17:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

### 一、新建任务

![image-20220901173453983](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901173453983.png)

![image-20220901173424816](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901173424816.png)

### 二、编写Pipeline script

![image-20220901173658456](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901173658456.png)

#### Pipeline script

```javascript
pipeline {
    agent {
        label 'tenxunyun-node'
    }
    
    environment {
        imagename = "flask_script"
        tag = "v${BUILD_NUMBER}"
        GIT_REPOSITORY_URL = "https://github.com/zhzhang12138/flask_script.git"
        GIT_CREDENTIALS_ID = "Github"
         
        DOCKER_REPOSITORY_URL = "ccr.ccs.tencentyun.com/flask_script"       
    }
    
    stages {
        stage('Pull code') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GIT_REPOSITORY_URL}"
            }
        }
        
        stage('Clean local history images') {
            steps {
                sh 'docker images | grep ${imagename} | awk \'{print $1":"$2}\' | xargs docker rmi || echo "DONE"'
            }
        }
        
         stage('Build docker image') {
            steps {
                timestamps{
                    sh 'docker build --tag=${imagename}:${tag} -q .'
                    sh 'docker tag ${imagename}:${tag} ${DOCKER_REPOSITORY_URL}/${imagename}:${tag}'
                    sh 'docker tag ${imagename}:${tag} ${DOCKER_REPOSITORY_URL}/${imagename}:latest'
                }
            }
        }
        
        stage('Push docker image') {
            steps {
                sh 'docker login ccr.ccs.tencentyun.com --username=100018328451 --password=Zt081998CJJ'
                sh 'docker push ${DOCKER_REPOSITORY_URL}/${imagename}:${tag}'
                sh 'docker push ${DOCKER_REPOSITORY_URL}/${imagename}:latest'
            }
        }
        
        
    }
}
```

### 三、查看构建结果

> Jenkins

![image-20220901173906385](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901173906385.png)

> tenxunyun-node

![image-20220901174118330](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901174118330.png)

> 远程镜像仓库

![image-20220901174258870](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901174258870.png)
