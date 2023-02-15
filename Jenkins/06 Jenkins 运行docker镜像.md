---
title: 'Jenkins运行docker镜像'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 17:17:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

### 一、新建任务

![image-20230215101422337](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20230215101422337.png)

### 二、编写Pipeline script

```bash
pipeline {
    agent {
        label '121.4.213.210'
    }
    
    environment {
        imagename = "flask-script-production"
        DOCKER_REPOSITORY_URL = "ccr.ccs.tencentyun.com/flask_script"   
        DOCKER_CONTAINER_RUN_ARGS = " -t --restart=always -p 9001:8080 -e AceEnv=production -e PYTHONIOENCODING=utf-8 -e PYTHONUNBUFFERED=0 "
    
    }
    
    stages {
    	 stage('Clean docker image') {
            steps {
                sh 'docker image prune -a -f'
            }
        }

    	stage('Pull docker image') {
            steps {
                sh 'docker login ccr.ccs.tencentyun.com --username=****** --password=******'
                sh 'docker pull ${DOCKER_REPOSITORY_URL}/${imagename}:latest'
            }
        }

         stage('Run') {
            steps {
                sh "docker stop ${imagename} || echo 'DONE'"
                sh "docker rm ${imagename} || echo 'DONE'"
                sh 'docker run -d --name ${imagename} ${DOCKER_CONTAINER_RUN_ARGS} ${DOCKER_REPOSITORY_URL}/${imagename}:latest'
            }
        }
    }
}
```

