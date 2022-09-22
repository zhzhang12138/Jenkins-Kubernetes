---
title: '安装k8s (基础配置)'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-09-02 09:44:00
top_img: transparent
cover: https://img2.baidu.com/it/u=2608987192,3553019956&fm=253&fmt=auto&app=138&f=JPEG?w=889&h=500
---

### 安装k8s

准备N台服务器, 内网互通。

> 1、同账号下同一VPC下云服务器CVM之间默认是内网互通的。
> 2、如果是不同账号下的云服务器CVM。 可以通过云联网实现内网互通。
>
>  参考文档：https://cloud.tencent.com/document/product/877/308053，
>
> 如果是同账号不同VPC下云服务器CVM， 也可以通过云联网实现内网互通。 
>
> 参考文档：https://cloud.tencent.com/document/product/877/30805

#### 机器环境准备

![image-20220902131105195](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902131105195.png)

![image-20220902131123172](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902131123172.png)

> 机器配置

| 主机名              | 节点ip        | 角色     | 部署组件                                                     |
| ------------------- | ------------- | -------- | ------------------------------------------------------------ |
| 腾讯云 k8s-master01 | 121.4.213.210 | Master01 | etcd, kube-apiserver, kube-controller-manager, kubectl, kubeadm,kubelet, kube-proxy, flannel |
| 腾讯云 k8s-node01   | 1.117.216.188 | Node01   | kubectl, kubelet, kube-proxy, flannel                        |

> 组件版本

| 组件       | 版本                        | 说明                                     |
| ---------- | --------------------------- | ---------------------------------------- |
| CentOS     | 7.6.1810                    |                                          |
| Kernel     | 3.10.0-1160.53.1.el7.x86_64 |                                          |
| etcd       | 3.3.15                      | 使用容器方式部署, 默认数据挂载到本地路径 |
| coredns    | 1.6.2                       |                                          |
| kubeadm    | v1.16.2                     |                                          |
| kubectl    | v1.16.2                     |                                          |
| kubelet    | v1.16.2                     |                                          |
| Kube-proxy | v1.16.2                     |                                          |
| flannel    | v0.11.0                     |                                          |

#### 安装前环境配置

> **注意, 有的配置是全部机器执行, 有的是部分节点执行**

> 修改主机名(每个主机挨个修改)

```shell
hostnamectl set-hostname k8s-master01
hostnamectl set-hostname k8s-node01
```

![image-20220902131550933](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902131550933.png)



> 添加hosts解析(统一修改)

```bash
cat >> /etc/hosts <<EOF
121.4.213.210  k8s-master01
1.117.216.188 k8s-node01
EOF
```

> ping检查一下

```shell
ping k8s-master01
ping k8s-node01
```

![image-20220902131646287](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220902131646287.png)

> 调整系统, 都是所有机器执行

```shell
iptables -P FORWARD ACCEPT  # 防火墙规则


swapoff -a
# 防止开机自动挂载 swap 分区
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


# 关闭防火墙以及selinux
sed -ri 's#(SELINUX=).*#\1disabled#' /etc/selinux/config
setenforce 0
systemctl disable firewalld && systemctl stop firewalld


# 开启内核对流量的转发
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
vm.max_map_count=262144
EOF

# 让上面的内核生效一下
modprobe br_netfilter

sysctl -p /etc/sysctl.d/k8s.conf


# 配置yum基础源和docker源
curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo

curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo



# 使用cat生成yum的kubernetes源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
				http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF



yum clean all && yum makecache

```

#### 安装docker

```shell
yum list docker-ce --showduplicates | sort -r

yum install docker-ce -y

mkdir -p /etc/docker

# 配置docker镜像加速地址
vi /etc/docker/deamon.json
{
  "registry-mirrors": ["https://h0ulrp80.mirror.aliyuncs.com"]
}

# 启动docker
systemctl enable docker && systemctl start docker

# 验证docker
docker version

```

#### 安装k8s-master01 (master01机器执行)

**k8s命令解释**

> 安装 kubeadm, kubelet, kubectl
>
> kubeadm: 用来初始化集群的指令, 安装集群的;
> kubelet: 在集群中的每个节点上用来启动Pod 和 容器等; 管理docker的;
> kubectl: 用来与集群通信的命令行工具, 就好比你用 docker ps , docker images类似;

操作节点: 所有的master和slave节点 ( `k8s-master`, `k8s-slave`) 需要执行

补充: yaml文件的语法; 配置文件常见的还有: ini, json, xml, yaml

```shell
yum install -y kubelet-1.16.2 kubeadm-1.16.2 kubectl-1.16.2 --disableexcludes=kubernetes

## 查看kudeadm版本
kubeadm version

## 设置kubelet开机启动, 作用是管理容器的, 下载镜像, 创建容器, 启停容器
# 确保机器一开机, kubelet服务启动了, 就会自动帮你管理pod(容器)
systemctl enable kubelet
```

初始化配置文件 (只在k8s-master01执行)

```shell
mkdir ~/k8s-install && cd ~/k8s-install

# 生成配置文件
kubeadm config print init-defaults > kubeadm.yaml

# 修改配置文件
vim kubeadm.yaml

```

只修改有标号1,2,3,4的部分, 其他请忽略

```shell
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.211.55.22  # 4.修改k8s-master01内网ip
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: k8s-m1 # 修改为第一台执行节点的hostname
  taints: null
---
controlPlaneEndpoint: 192.168.122.100:8443 # 新增控制平台地址
apiServer:
  timeoutForControlPlane: 4m0s
  extraArgs:
    authorization-mode: "Node,RBAC"
    # nfs/rbd pvc 需要用到
    feature-gates: RemoveSelfLink=false
    enable-admission-plugins: "NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeClaimResize,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,Priority"
    etcd-servers: https://k8s-m1:2379,https://k8s-m2:2379,https://k8s-m3:2379 # etcd 节点列表
  certSANs:
  - 192.168.122.100 # VIP 地址
  - 10.96.0.1  # service cidr的第一个ip
  - 127.0.0.1 # 多个master的时候负载均衡出问题了能够快速使用localhost调试
  - k8s-m1
  - k8s-m2
  - k8s-m3
  - kubernetes
  - kubernetes.default
  - kubernetes.default.svc
  - kubernetes.default.svc.cluster.local
  extraVolumes:
  - hostPath: /etc/localtime
    mountPath: /etc/localtime
    name: timezone
    readOnly: true
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager:
  extraVolumes:
  - hostPath: /etc/localtime
    mountPath: /etc/localtime
    name: timezone
    readOnly: true
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
    # etcd 高可用，需要配置多个节点
    serverCertSANs:
    - k8s-m1
    - k8s-m2
    - k8s-m3
    peerCertSANs:
    - k8s-m1
    - k8s-m2
    - k8s-m3
imageRepository: registry.aliyuncs.com/google_containers # 1.修改阿里镜像源
kind: ClusterConfiguration
kubernetesVersion: v1.16.2 # 2.修改你安装的k8s版本
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16 # 3.添加pod网段, 设置容器内网络
scheduler:
  extraVolumes:
  - hostPath: /etc/localtime
    mountPath: /etc/localtime
    name: timezone
    readOnly: true
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
bindAddressHardFail: false
clientConnection:
  acceptContentTypes: ""
  burst: 0
  contentType: ""
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
  qps: 0
clusterCIDR: ""
configSyncPeriod: 0s
conntrack:
  maxPerCore: null
  min: null
  tcpCloseWaitTimeout: null
  tcpEstablishedTimeout: null
detectLocalMode: ""
enableProfiling: false
healthzBindAddress: ""
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: null
  minSyncPeriod: 0s
  syncPeriod: 0s
ipvs:
  excludeCIDRs: null
  minSyncPeriod: 0s
  scheduler: ""
  strictARP: false
  syncPeriod: 0s
  tcpFinTimeout: 0s
  tcpTimeout: 0s
  udpTimeout: 0s
kind: KubeProxyConfiguration
metricsBindAddress: ""
mode: "ipvs" # ipvs 模式
nodePortAddresses: null
oomScoreAdj: null
portRange: ""
showHiddenMetricsForVersion: ""
udpIdleTimeout: 0s
winkernel:
  enableDSR: false
  networkName: ""
  sourceVip: ""
---
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging: {}
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

```

修改完成如下:

```shell
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 172.17.0.5  # 4.修改k8s-master01内网ip, 根据自己服务器去填写
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers  # 1.修改阿里镜像源
kind: ClusterConfiguration
kubernetesVersion: v1.16.2  # 2.修改你安装的k8s版本
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16  # 3.添加pod网段, 设置容器内网络
scheduler: {}

```

对上面的资源清单的文档比较复杂, 想要完整了解上面的资源对象对应的属性, 可以查看对应的godoc文档, 地址:

https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2

提前下载镜像 (只在k8s-master01执行)

```shell
# 检查镜像列表
[root@k8s-master01 k8s-install]# kubeadm config images list --config kubeadm.yaml
registry.aliyuncs.com/google_containers/kube-apiserver:v1.16.2
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.16.2
registry.aliyuncs.com/google_containers/kube-scheduler:v1.16.2
registry.aliyuncs.com/google_containers/kube-proxy:v1.16.2
registry.aliyuncs.com/google_containers/pause:3.1
registry.aliyuncs.com/google_containers/etcd:3.3.15-0
registry.aliyuncs.com/google_containers/coredns:1.6.2

# 提前下载镜像
[root@k8s-master01 k8s-install]# kubeadm config images pull --config kubeadm.yaml
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.16.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.16.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.16.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.16.2
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.1
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.3.15-0
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:1.6.2

```

#### 初始化k8s-master01节点

```shell
[root@k8s-master01 k8s-install]# kubeadm init --config kubeadm.yaml
[init] Using Kubernetes version: v1.16.2
......
```

初始化成功如下:

```shell
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:
  
  # 执行如下命令, 创建配置文件 
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 你需要创建pod网络, 你的pod才能正常工作
# k8s的集群网络, 需要借助额外的插件 
# 这个先不着急, 等下个步骤做
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

# 你需要用如下命令, 将node节点, 加入集群
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.29.58.226:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ebdc26b59529e03a62207699d60b8b64c2c19991db7a912f4aeae836615dd9ea 

```

> 注意看, 最后几行信息, 就是让k8s-node01/02 公认节点, 加入到k8s集群中的命令

> 按照上述提示, master01操作

```shell
[root@k8s-master01 k8s-install]# mkdir -p $HOME/.kube
[root@k8s-master01 k8s-install]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master01 k8s-install]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> 此时k8s集群状态, 应该是挂掉的, 这是正常的, 因为我们没有配置好k8s的网络插件

```shell
[root@k8s-master01 k8s-install]# kubectl get nodes
NAME           STATUS     ROLES    AGE    VERSION
k8s-master01   NotReady   master   7m6s   v1.16.2

```

接下来到k8s-node节点操作了。

#### 添加k8s-node到集群中

所有的node节点操作, 复制上述k8s-master01生成的信息

```shell
kubeadm join 172.29.58.226:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ebdc26b59529e03a62207699d60b8b64c2c19991db7a912f4aeae836615dd9ea 
    
    
# 成功
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

去k8s-master01检查nodes状态 (可以尝试 `-owide` 显示更详细)

```shell
[root@k8s-master01 k8s-install]# kubectl get nodes
NAME           STATUS     ROLES    AGE     VERSION
k8s-master01   NotReady   master   8m35s   v1.16.2
k8s-node01     NotReady   <none>   10s     v1.16.2
```

发现, 集群中, 已经存在我们加入的2个node节点了, 但是状态还是未就绪, 还是因为网络问题

#### 安装flannel网络插件 (k8s-master01执行)

这里可能有网络问题, 多尝试几次

```shell
wget https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```

修改配置文件, 指定机器的网卡名, 大约在190行

注意: ifconfig 查看自己的网卡名字, 不一定都是eth0; `[root@k8s-master01 k8s-install]# ifconfig`

```shell
[root@k8s-master01 k8s-install]# vim kube-flannel.yml 
```

```shell
189				args:
190				- --ip-masq
191				- --kube-subnet-mgr
192				- --iface=eth0  # 添加这个配置
```

下载flannel网络插件镜像

```shell
[root@k8s-master01 k8s-install]# docker pull quay.io/coreos/flannel:v0.11.0-amd64
v0.11.0-amd64: Pulling from coreos/flannel
cd784148e348: Pull complete 
04ac94e9255c: Pull complete 
e10b013543eb: Pull complete 
005e31e443b1: Pull complete 
74f794f05817: Pull complete 
Digest: sha256:7806805c93b20a168d0bbbd25c6a213f00ac58a511c47e8fa6409543528a204e
Status: Downloaded newer image for quay.io/coreos/flannel:v0.11.0-amd64
quay.io/coreos/flannel:v0.11.0-amd64


# 安装flannel网络插件
[root@k8s-master01 k8s-install]# kubectl create -f kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created

# 再次检查k8s集群, 确保所有节点是redy
[root@k8s-master01 k8s-install]# kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
k8s-master01   Ready    master   13m     v1.16.2
k8s-node01     Ready    <none>   4m47s   v1.16.2
k8s-node02     Ready    <none>   4m45s   v1.16.2

# 此时已经是全部正确状态
# 可以再看下集群中所有的pods状态, 确保都是正确的
[root@k8s-master01 k8s-install]# kubectl get pods -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-58cc8c89f4-fgjpn               1/1     Running   0          13m
kube-system   coredns-58cc8c89f4-zz245               1/1     Running   0          13m
kube-system   etcd-k8s-master01                      1/1     Running   0          12m
kube-system   kube-apiserver-k8s-master01            1/1     Running   0          12m
kube-system   kube-controller-manager-k8s-master01   1/1     Running   0          12m
kube-system   kube-flannel-ds-amd64-2sbrp            1/1     Running   0          80s
kube-system   kube-flannel-ds-amd64-lpplk            1/1     Running   0          80s
kube-system   kube-flannel-ds-amd64-zkn4k            1/1     Running   0          80s
kube-system   kube-proxy-dvnfk                       1/1     Running   0          5m5s
kube-system   kube-proxy-sbwkc                       1/1     Running   0          13m
kube-system   kube-proxy-sj8bz                       1/1     Running   0          5m3s
kube-system   kube-scheduler-k8s-master01            1/1     Running   0          12m

```

#### 到此为止, k8s集群已部署完毕~

### 首次使用k8s部署应用程序

#### K8s 部署 Nginx

##### 创建 deployment

> k8s deployment 用来部署应用。一个常见的 deployment 配置包括几个部分，详细可以参考官网有关介绍 Deployments。
>
> spec.selector：用于定义 deployment 如何查找要管理的 pod，例如，使用在 pod 模板中定义的标签，如 app:nginx
> spec.replicas：用于定义需要启动多少个副本
> spec.template：用于定义 pod 的属性，例如，容器名称，容器镜像，labels 字段，等
> 完整的 nginx-dep.yml 文件如下

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

为了创建 deployment，执行命令：

```shell
kubectl apply -f nginx-dep.yml
```

查看 deployment 状态：

```shell
kubectl get deploy -o wide
```

可以看到，刚创建的 nginx-deployment 的 3 个副本均处于 READY 状态：

```shell
[root@k8s-master01 k8s-install]# kubectl get deploy -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-deployment   3/3     3            3           21m   nginx        nginx:alpine   app=nginx
```

##### 创建 service

> k8s 使用 service 来实现服务发现，常见配置包括：
>
> spec.selector：用于定义如何选择 pod
> spec.ports：用于定义如何暴露端口，其中，nodePort 指定可以在外部访问的端口
> 完整的 nginx-svc.yml 文件如下：

```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

为创建 servcie，执行命令：

```shell
kubectl apply -f nginx-svc.yml
```

查看 service 的状态：

```shell
kubectl get svc nginx-service -o wide
```

输出

```shell
[root@k8s-master01 k8s-install]# kubectl get svc nginx-service -o wide
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
nginx-service   NodePort   10.107.57.238   <none>        80:30080/TCP   21m   app=nginx
```

为了验证在外部正常访问 nginx，打开浏览器，输入地址 http://localhost:30080/，可以看到 nginx 的欢迎页面，说明 nginx-service 创建成功：

![image-20220825105951622](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220825105951622.png)

### 参考网站

> Python3 - k8s架构的安装与使用(详细)
>
> https://blog.csdn.net/qq_31810357/article/details/123920868

> kubernetes后加入的工作节点处于“NotReady”状态
>
> https://blog.csdn.net/xiaohuixing16134/article/details/102784269

> K8s 部署 Nginx
>
> https://blog.csdn.net/Dyson_HQ/article/details/125522181