---
title: 'Kubernetes 之命令集锦'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-09-02 13:32:00
top_img: transparent
cover: https://img2.baidu.com/it/u=2608987192,3553019956&fm=253&fmt=auto&app=138&f=JPEG?w=889&h=500
---

### Node

| 功能说明                 | 命令                             |
| ------------------------ | -------------------------------- |
| 查看服务器节点           | kubectl get nodes                |
| 查看服务器节点详情       | kubectl get nodes -o wide        |
| 查看服务器节点的运行信息 | kubectl describe node <节点名称> |

### Pod

| 功能说明                        | 命令                                                   |
| ------------------------------- | ------------------------------------------------------ |
| 根据yaml文件创建pod             | kubectl apply -f <文件名称>                            |
| 根据yaml文件删除pod             | kubectl delete -f <文件名称>                           |
| 删除pod节点                     | delete pod <pod名称> -n <名称空间>                     |
|                                 |                                                        |
| 查看pod节点                     | kubectl get pod                                        |
| 查看所有pod节点                 | kubectl get pods -A                                    |
| 查看pod节点的日志               | kubectl logs <pod名称>                                 |
| 查看所有pod节点详情             | kubectl get pod -o wide                                |
| 查看单个pod节点启动详情         | kubectl describe pod <pod名称>                         |
| 查看所有名称空间下的pod         | kubectl get pod --all-namespaces                       |
| 查看 Pod 的配置是否正确         | kubectl get pod <pod名称> -o yaml                      |
|                                 |                                                        |
| 进入默认命名空间的pod节点       | kubectl exec -it <pod名称> -- /bin/bash                |
| 进入某个特定命名空间下的pod节点 | kubectl exec -it <pod名称>  -n <命名空间> -- /bin/bash |
|                                 |                                                        |
| 监控pod(一秒钟更新一次命令)     | watch -n 1 kubectl get pod  <pod名称>                  |

### Deployment

| 功能说明                                      | 命令                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| deployment部署pod(具有自愈能力，宕机自动拉起) | kubectl apply -f <文件名称>                                  |
|                                               |                                                              |
| 查看deployment部署                            | kubectl get deployments.apps                                 |
| 查看deployment状态                            | kubectl rollout status  deployment/<dep名称>                 |
| 查询deployment发布历史                        | kubectl rollout history  deployment/<dep名称>                |
| 查询deployment发布历史详情                    | kubectl rollout history  deployment/<dep名称> --revision=<版本号> |
|                                               |                                                              |
| 伸缩扩展副本                                  | kubectl scale deployment <dep名称> --replicas=<副本数量>     |
| 回滚deployment                                | kubectl rollout undo deployment/<dep名称> --to-revision=<版本号> |
| deployment更新image版本                       | kubectl  set image  deployment/<dep名称>  <容器名称>=<容器镜像地址>:<版本号><br />例：kubectl  set image  deployment/simpleweb  simpleweb=zhangxiaotao/johndoe:v1.0.1 |
| 删除deployment部署                            | kubectl delete deployment <dep名称>                          |

### Service

| 功能说明                 | 命令                                           |
| ------------------------ | ---------------------------------------------- |
| 根据yaml文件创建Service  | kubectl apply -f  <文件名称>                   |
|                          |                                                |
| 查看服务                 | kubectl get svc                                |
| 查看服务详情             | kubectl get svc -o wide                        |
| 查看所有名称空间下的服务 | kubectl get svc --all-namespaces               |
|                          |                                                |
| 删除svc                  | kubectl delete svc <Service名称> -n <命名空间> |
