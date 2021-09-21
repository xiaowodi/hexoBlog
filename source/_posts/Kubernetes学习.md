





---
title: Kubernetes入门学习
---











# Kubernetes学习笔记

## 第一部分：K8s概念和架构

### 1. K8S概述和特性

####  Kubernetes 基本介绍

kubernetes，简称   开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes 的应用简单并且高（powerful）,Kubernetes ，更新，维护的一种机制。

传统的应用部署方式是通过插件或脚本来安装应用。这样做的缺点是应用的运行、配置、管理、所有生存周期将与当前操作系统绑定，这样做并不利于应用的升级更新/回滚等操作，当然也可以通过创建虚拟机的方式来实现某些功能，但是虚拟机非常重，并不利于可移性。

新的方式是通过部署容器方式实现，每个容器之间互相隔离，每个容器有自己的文件系统 资源。相对于虚拟机，容器能快速部署，由于容器与底层设施、机器文件系统解耦的，所以它能在不同云、不同版本操作系统间进行迁移。

容器占用资源少、部署快，每个应用可以被打包成一个容器镜像，每个应用与容器间成一对一关系也使容器有更大优势，使用容器可以在  建容器镜像，因为每个应用不需要与其余的应用堆栈组合，也不依赖于生产环境基础结构，这使得从研发到测试、生产能提供一致环境。类似地，容器比虚拟机轻量、更“透明”，这更便于监控和管理。

Kubernetes  动化部署、大规模可伸缩、应用容器化管理。在生产环境中部署一个应用程序时，通常要部署该应用的多个实例以便对应用请求进行负载均衡。

在 个容器里面运行一个应用实例，然后通过内置的负载均衡策略，实现对这一组应用实例的管理、发现、访问，而这些细节都不需要运维人员去进行复杂的手工配置和处理。

##### 

Kubernetes是一个轻便的和可扩展的开源平台，用于管理容器化应用和服务。通过Kubernetes能够进行应用的自动化部署和扩缩容。在Kubernetes中，会将组成应用的容器组合在一个逻辑单元以更易管理和发现。Kubernetes累积了Google生产环境运行工作负载15年的经验，并吸收了来组社区的最佳想法和实践。



### 2.K8S架构组件

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.mamicode.com%2Finfo%2F201811%2F20181121230758563675.png&refer=http%3A%2F%2Fimage.mamicode.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627273055&t=80481fe36bc98fabdad850e85819c4b8" alt="img" style="zoom:80%;" />

1. **主控节点Master节点中的组件**
   - **API Server**
     - 集群统一入口，以restful方式，交给etcd存储
   - **scheduler**
     - 节点调度，选择node节点应用部署
   - **controller-manager**
     - 处理集群中常规后台任务，一个资源对应一个控制器
       - 举个例子：部署一个订单的应用，那么订单这个应用就会对应一个controller；如果还有一个购物车的应用，那么购物车也会对应一个controller。
   - **etcd**
     - 存储系统，用于保存集群相关的数据。
2. **工作节点node节点中的组件**
   - **kubelet**
     - master派到node节点中的代表，管理本机的容器
   - **kube-proxy**
     - 提供网络代理，可实现负载均衡等操作



### 3.K8S核心概念初识

1. **Pod**
   - k8s中最小的部署单元（一个pod中可以包含多个容器）
   - 一组容器的集合
   - 一个Pod中的容器是共享网络的
   - Pod的生命周期是短暂的
2. **controller**（通过controller创建pod，部署pod）
   - 确保预期的pod副本数量
   - 无状态的应用部署
   - 有状态的应用部署
   - 确保所有的node运行同一个pod
   - 一次性任务和定时任务
3. **Service**（统一部署）
   - 定义一组pod的访问规则











## 第二部分：从零搭建K8s集群、

### 前置准备工作

#### 1. 搭建k8s环境平台规划

> 平台规划

1. 单master集群

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fassets.sumologic.com%2Fapplication-content%2Fkubernetes_cluster.png%3Fmtime%3D20191022144431&refer=http%3A%2F%2Fassets.sumologic.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627274979&t=28ae11eca3968dbc42570f385274a6b6" alt="img" style="zoom:60%;" />



2. 多master集群（高可用 ）

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F9620980-be963c83b5952fca.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627274636&t=99675bcc0d1a3562958f3b6ab768b79d" alt="img" style="zoom:60%;" />



#### 2. 服务器硬件配置要求

#### 硬件要求

测试环境下：

| 节点类型 | CPU  | 内存 | 硬盘 |
| -------- | ---- | ---- | ---- |
| Master   | 2核  | 4G   | 20G  |
| node     | 4核  | 8G   | 40G  |



#### 3. 搭建k8s集群部署方式

搭建k8s集群有多种部署方式，常用的有以下两种方式：

1. **基于客户端工具（kubeadm）**

>Kubeadm是一个k8s部署工具，提供kubeadm init 和 kubeadm join， 用于快速部署Kubernetes集群
>
>官方地址：https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/

2. **基于二进制包方式**

> 从github下载发行版的二进制包，手动部署每个组件，组成 Kubernetes集群。
> Kubeadm 蔽了很多细节，遇到问题很难排查。如果想更容易可控，推荐使用二进制包部署 期间可以学习很多工作原理，也利于后期维护。

### 搭建集群

虚拟机硬件配置：一台master，两台node

- 硬件配置
  - 2GB或更多RAM，2个CPU或更多CPU，硬盘30G或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF



cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64



wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  /etc/yum.repos.d/docker-ce.repo





#### **基于二进制包方式**

##### 搭建etcd集群

Etcd是一个分布式键值存储系统，Kubernetes使用Etcd进行数据存储，所以先准备一个Etcd数据库，为解决Etcd单点故障，应采用集群方式部署，这里使用3台组建集群，可容忍1台机器故障。

| 节点名称 | IP            | 备注 |
| -------- | ------------- | ---- |
| etcd-1   | 192.168.0.110 |      |
| etcd-2   | 192.168.0.111 |      |
| etcd-3   | 192.168.0.112 |      |

###### 1.准备cfssl证书生成工具

cfssl是一个开源的证书管理工具，使用json文件生成证书，相比openssl更方便使用。

在Master节点上，执行下列命令：

```shell
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```

###### 2. 生成Etcd证书

（1）自签证书颁发机构（CA）

```shell
# 创建工作目录
$ mkdir -p ~/TLS/{etcd,k8s} 
$ cd TLS/etcd
# 自签CA
$ cat > ca-config.json<< EOF
{
	"signing":{
		"default":{
			"expiry":"87600h"
		},
		"profiles":{
			"www":[
				"signing",
				"key encipherment",
				"server auth",
				"client auth"
			]
		}
	}
}
EOF

$ cat  > ca-csr.json<< EOF
{
	"CN":"etcd CA",
	"key":{
		"algo":"rsa",
		"size":"2048"
	},
	"names":[
		{
			"C":"CN",
			"L":"Beijing",
			"ST":"Beijing"
		}
	]
}
EOF

# 生成证书：
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
$ ls *pem
ca-key.pem	ca.pem
```

（2）使用自签CA签发Etcd Https证书

```shell
# 创建证书申请文件
$ cat > server-csr.json<< EOF
{
	"CN":"etcd",
	"hosts": [
		"192.168.0.110",
		"192.168.0.111",
		"192.168.0.112"
	],
	"key":{
		"algo":"rsa",
		"size":2048
	},
	"names":[
		{
			"C":"CN",
			"L":"Beijing",
			"ST":"Beijing"
		}
	]
}
EOF
# 注： 上述文件hosts字段中ip为所有etcd节点的集群内部通信IP，一个都不能少！为了方便后期扩容可以多写几个预留的IP

# 生成证书
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server
$ ls server*pem
server-key.pem	server.pem
```

###### 3.从github下载二进制文件

下载地址：https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz

```shell
#1. 创建工作目录并解压etcd二进制包
mkdir /opt/etcd/{bin,cfg,ssl} -p
tar zxvf etcd-v3.4.14-linux-amd64.tar.gz
mv etcd-v3.4.14-linux-amd64/{etcd,etcdctl} /opt/etcd/bin/
#2.建etcd.conf（修改对应的master的ip地址）
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.0.110:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.0.110:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.0.110:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.0.110:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.0.110:2380,etcd-2=https://192.168.0.111:2380,etcd-3=https://192.168.0.112:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
# ETCD_NAME：节点名称，集群中唯一
# ETCD_DATA_DIR：数据目录
# ETCD_LISTEN_PEER_URLS：集群通信监听地址
# ETCD_LISTEN_CLIENT_URLS：客户端访问监听地址
# ETCD_INITIAL_ADVERTISE_PEER_URLS：集群通告地址
# ETCD_ADVERTISE_CLIENT_URLS：客户端通告地址
# ETCD_INITIAL_CLUSTER：集群节点地址
# ETCD_INITIAL_CLUSTER_TOKEN：集群 Token
# ETCD_INITIAL_CLUSTER_STATE：加入集群的当前状态，new 是新集群，existing 表示加入 已有集群

#3. systemd 管理 etcd  创建etcd.service
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
EnvironmentFile=/opt/etcd/cfg/etcd.conf
ExecStart=/opt/etcd/bin/etcd \
--cert-file=/opt/etcd/ssl/server.pem \
--key-file=/opt/etcd/ssl/server-key.pem \
--peer-cert-file=/opt/etcd/ssl/server.pem \
--peer-key-file=/opt/etcd/ssl/server-key.pem \
--trusted-ca-file=/opt/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/opt/etcd/ssl/ca.pem \
--logger=zap
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

# 4. 拷贝刚才生成的证书 到 配置文件的路径
cp ~/TLS/etcd/ca*pem ~/TLS/etcd/server*pem /opt/etcd/ssl/
# 5. 启动并设置开机启动(稍后执行，待从节点文件同步后再一起执行)
systemctl daemon-reload
systemctl start etcd  # 启动后会卡住因为需要和node节点一起执行改命令
systemctl enable etcd


# 6. 将上面节点 1 所有生成的文件拷贝到节点 2 和节点 3
scp -r /opt/etcd/ root@192.168.0.111:/opt/
scp /usr/lib/systemd/system/etcd.service root@192.168.0.111:/usr/lib/systemd/system/
scp -r /opt/etcd/ root@192.168.0.112:/opt/
scp /usr/lib/systemd/system/etcd.service root@192.168.0.112:/usr/lib/systemd/system/

# 7. 然后在节点 2 和节点 3 分别修改 etcd.conf 配置文件中的节点名称和当前服务器 IP
# 节点 2 改为 etcd-2，节点 3 改为 etcd-3
vim /opt/etcd/cfg/etcd.conf
# 同时需要更改etcd.conf 中的当前所在机器对应的ip地址

# 8.然后三台节点同时执行 步骤5的指令

# 9. 查看集群状态
ETCDCTL_API=3 /opt/etcd/bin/etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.0.110:2379,https://192.168.0.111:2379,https://192.168.0.112:2379" endpoint health
# 输出：
https://192.168.0.112:2379 is healthy: successfully committed proposal: took = 57.453222ms
https://192.168.0.110:2379 is healthy: successfully committed proposal: took = 55.09686ms
https://192.168.0.111:2379 is healthy: successfully committed proposal: took = 61.813391ms
# etcd 集群搭建成功！
```

如果输出上面信息，就说明集群部署成功。如果有问题第一步先看日志：
/var/log/message  或 journalctl  -u etcd

<font color=red>如果服务器死机，宕机，故障，导致etcd集群中个节点的数据不同步，进而导致k8s集群异常：可参考以下这2个帖子</font>

https://www.cnblogs.com/dudu/p/12161010.html

https://blog.csdn.net/qq_42883074/article/details/112789206

这里列出我遇到的问题：

```shell
# 突遇电脑死机，整个用虚拟机搭建的集群突然宕掉
# 重启后，运行kubectl get nodes 卡住，最后报错：Unable to connect to the server
# 查看3台节点上的etcd服务， 有两台服务状态为faild
journalctl -f -u kubelet.service # 报错，找不到节点
```

https://blog.csdn.net/weiyongle1996/article/details/75128239?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.base&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.base

enss3无法使用ip地址





##### 安装Docker（三台节点都需要安装）

下载地址：https://download.docker.com/linux/static/stable/x86_64/docker-19.03.9.tgz

```shell
# 1. 先卸载旧版本的docker
yum list installed | grep docker # 查找之前安装过的软件
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 2. 解压并移动
tar zxvf docker-19.03.9.tgz 
mv docker/* /usr/bin

# 3. systemd管理docker服务
cat > /usr/lib/systemd/system/docker.service << EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF

# 启动服务
[root@K8s-Node2 software]# systemctl daemon-reload
[root@K8s-Node2 software]# systemctl start docker
[root@K8s-Node2 software]# systemctl enable docker
```

##### 部署Master Node

在Master节点上操作

###### 1. 生成kube-apiserver证书

```shell
# 1. 自签证书颁发机构（CA）
cd TLS/k8s
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
         "expiry": "87600h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF

cat > ca-csr.json << EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

# 2. 生成证书
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
ls *pem

# 3. 使用自签CA签发kube-apiserver HTTPS证书
# 创建证书申请文件：
cat > server-csr.json << EOF
{
    "CN": "kubernetes",
    "hosts": [
      "10.0.0.1",
      "127.0.0.1",
      "192.168.0.110",
      "192.168.0.111",
      "192.168.0.112",
      "192.168.0.113",
      "192.168.0.114",
      "192.168.0.115",
      "192.168.0.116",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server

ls server*pem
```

###### 2. 从github下载二进制文件

下载地址：
https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1183

下载kubernetes-server-linux-amd64.tar.gz即可

###### 3.解压kubernetes-server-linux-amd64.tar.gz文件并移动到相应目录下

```shell
mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}
tar zxvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
cp kube-apiserver kube-scheduler kube-controller-manager /opt/kubernetes/bin
cp kubectl /usr/bin/
```

###### 4. 部署kube-apiserver

```shell
# 1. 创建配置文件
cat > /opt/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--etcd-servers=https://192.168.0.110:2379,https://192.168.0.111:2379,https://192.168.0.112:2379 \\
--bind-address=192.168.0.110 \\
--secure-port=6443 \\
--advertise-address=192.168.0.110 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.0.0.0/24 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/opt/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-32767 \\
--kubelet-client-certificate=/opt/kubernetes/ssl/server.pem \\
--kubelet-client-key=/opt/kubernetes/ssl/server-key.pem \\
--tls-cert-file=/opt/kubernetes/ssl/server.pem  \\
--tls-private-key-file=/opt/kubernetes/ssl/server-key.pem \\
--client-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--etcd-cafile=/opt/etcd/ssl/ca.pem \\
--etcd-certfile=/opt/etcd/ssl/server.pem \\
--etcd-keyfile=/opt/etcd/ssl/server-key.pem \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log"
EOF

#######################
# 参数说明
# 注：上面两个\ \ 第一个是转义符，第二个是换行符，使用转义符是为了使用 EOF 保留换 行符。
# –logtostderr：启用日志
# —v：日志等级
# –log-dir：日志目录
# –etcd-servers：etcd 集群地址
# –bind-address：监听地址
# –secure-port：https 安全端口
# –advertise-address：集群通告地址
# –allow-privileged：启用授权
# –service-cluster-ip-range：Service 虚拟 IP 地址段
# –enable-admission-plugins：准入控制模块
# –authorization-mode：认证授权，启用 RBAC 授权和节点自管理
# –enable-bootstrap-token-auth：启用 TLS bootstrap 机制
# –token-auth-file：bootstrap token 文件
# –service-node-port-range：Service nodeport 类型默认分配端口范围
# –kubelet-client-xxx：apiserver 访问 kubelet 客户端证书
# –tls-xxx-file：apiserver https 证书
# –etcd-xxxfile：连接 Etcd 集群证书
# –audit-log-xxx：审计日志
#######################

# 2. 拷贝刚才生成的证书 到 kubernetes配置文件中的路径
cp ~/TLS/k8s/ca*pem ~/TLS/k8s/server*pem /opt/kubernetes/ssl/
```

> 3. **启用 TSL Bootstrapping机制**
>
> TSL Bootstrapping：Master apiserver启动TLS认证后，Node节点kubelet和kube-proxy要与kube-apiserver进行通信，必须使用CA签发的有效证书才可以，当Node节点很多时，这种客户端证书颁发需要大量工作，同样也会增加集群扩展复杂度。为了简化流程，Kubernetes引入了TLS bootstrapping机制来自动颁发客户端证书，kubelet会以一个低权限用户自动向apiserver申请证书，kubelet的证书有apiserver动态签署
>
> 所以强烈建议在Node上使用这种方式，目前主要用于kubelet；kube-proxy还是由我们统一颁发一个证书。
>
> TLS bootstraping工作流程：
>
> <img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.52wzj.com%2Fwp-content%2Fuploads%2F2020%2F6%2FjQjeAr.png&refer=http%3A%2F%2Fwww.52wzj.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627316875&t=0fd6871c7e84397a9e2e51cd1e6daacb" alt="img" style="zoom:50%;" />
>
> ```shell
> # 1. 创建上述配置文件中的token文件
> # 生成 一串 随机的 32位串，作为用户组token
> $ head -c 16 /dev/urandom | od -An -t x | tr -d ' '
> 220ee81becf461cfc2d2b0e983d4347c
> $ cat > /opt/kubernetes/cfg/token.csv << EOF
> 220ee81becf461cfc2d2b0e983d4347c,kubelet-bootstrap,10001,"system:node-bootstrapper"
> EOF
> #说明： 格式：token，用户名，UID，用户组 token 也可自行生成替换：
> ```
>
> 

> 4. **systemd管理apiserver**
>
> ```shell
> cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
> [Unit]
> Description=Kubernetes API Server
> Documentation=https://github.com/kubernetes/kubernetes
> [Service]
> EnvironmentFile=/opt/kubernetes/cfg/kube-apiserver.conf
> ExecStart=/opt/kubernetes/bin/kube-apiserver \$KUBE_APISERVER_OPTS
> Restart=on-failure
> [Install]
> WantedBy=multi-user.target
> EOF
> ```
>
> 5. **启动并设置开机自启**
>
> ```shell
> systemctl daemon-reload
> systemctl start kube-apiserver
> systemctl enable kube-apiserver
> systemctl status kube-apiserver 
> ```
>
> 6. **授权kubelet-bootstrap用户允许请求证书**
>
> ```shell
> kubectl create clusterrolebinding kubelet-bootstrap \
> --clusterrole=system:node-bootstrapper \
> --user=kubelet-bootstrap
> ```

###### 5.部署kube-controller-manager

> ```shell
> cat > /opt/kubernetes/cfg/kube-controller-manager.conf << EOF
> KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=false \\
> --v=2 \\
> --log-dir=/opt/kubernetes/logs \\
> --leader-elect=true \\
> --master=127.0.0.1:8080 \\
> --bind-address=127.0.0.1 \\
> --allocate-node-cidrs=true \\
> --cluster-cidr=10.244.0.0/16 \\
> --service-cluster-ip-range=10.0.0.0/24 \\
> --cluster-signing-cert-file=/opt/kubernetes/ssl/ca.pem \\
> --cluster-signing-key-file=/opt/kubernetes/ssl/ca-key.pem  \\
> --root-ca-file=/opt/kubernetes/ssl/ca.pem \\
> --service-account-private-key-file=/opt/kubernetes/ssl/ca-key.pem \\
> --experimental-cluster-signing-duration=87600h0m0s"
> EOF
> # 说明
> –master：通过本地非安全本地端口 8080 连接 apiserver。
> 
> –leader-elect：当该组件启动多个时，自动选举（HA）
> 
> –cluster-signing-cert-file/–cluster-signing-key-file：自动为 kubelet 颁发证书的 CA，与 apiserver 保持一致
> 
> # systemd管理controller-manager
> cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
> [Unit]
> Description=Kubernetes Controller Manager
> Documentation=https://github.com/kubernetes/kubernetes
> [Service]
> EnvironmentFile=/opt/kubernetes/cfg/kube-controller-manager.conf
> ExecStart=/opt/kubernetes/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
> Restart=on-failure
> [Install]
> WantedBy=multi-user.target
> EOF
> 
> # 启动并开机自启
> systemctl daemon-reload
> systemctl start kube-controller-manager
> systemctl enable kube-controller-manager
> systemctl status kube-controller-manager
> 
> ```

###### 6.部署kuber-scheduler

> ```shell
> # 1. 创建kuber-scheduler配置文件
> cat > /opt/kubernetes/cfg/kube-scheduler.conf << EOF
> KUBE_SCHEDULER_OPTS="--logtostderr=false \
> --v=2 \
> --log-dir=/opt/kubernetes/logs \
> --leader-elect \
> --master=127.0.0.1:8080 \
> --bind-address=127.0.0.1"
> EOF
> # #
> # –master：通过本地非安全本地端口 8080 连接 apiserver。
> # –leader-elect：当该组件启动多个时，自动选举（HA）
> # #
> # 2. systemd管理kuber-scheduler
> cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
> [Unit]
> Description=Kubernetes Scheduler
> Documentation=https://github.com/kubernetes/kubernetes
> [Service]
> EnvironmentFile=/opt/kubernetes/cfg/kube-scheduler.conf
> ExecStart=/opt/kubernetes/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
> Restart=on-failure
> [Install]
> WantedBy=multi-user.target
> EOF
> # 启动并设置开机自启
> systemctl daemon-reload
> systemctl start kube-scheduler
> systemctl enable kube-scheduler
> systemctl status kube-scheduler
> ```
>
> 



###### 7. **查看集群状态**

```shell
$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
```

##### 部署Worker Node

下面还是在Master Node上操作，即同时作为Worker Node

```shell
#1. 所有的worker node创建工作目录
$ mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}
#2.从master节点拷贝
$ cd kubernetes/server/bin
$ cp kubelet kube-proxy /opt/kubernetes/bin
```

###### 1.部署kubelet

1. 创建配置文件

```shell
cat > /opt/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--hostname-override=K8s-Master \\
--network-plugin=cni \\
--kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/opt/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/opt/kubernetes/ssl \\
--pod-infra-container-image=lizhenliang/pause-amd64:3.0"
EOF
```

–hostname-override：显示名称，集群中唯一
–network-plugin：启用CNI
–kubeconfig：空路径，会自动生成，后面用于连接apiserver
–bootstrap-kubeconfig：首次启动向apiserver申请证书
–config：配置参数文件
–cert-dir：kubelet证书生成目录
–pod-infra-container-image：管理Pod网络容器的镜像



2. 配置参数文件

```shell
cat > /opt/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS:
- 10.0.0.2
clusterDomain: cluster.local 
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /opt/kubernetes/ssl/ca.pem 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF
```

3. 生成bootstrap.kubeconfig文件

```shell
KUBE_APISERVER="https://192.168.0.110:6443" # apiserver IP:PORT
TOKEN="220ee81becf461cfc2d2b0e983d4347c" # 与token.csv里保持一致
```

```shell
# 生成kubelet bootstrap kubeconfig配置文件
$ kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig
$ kubectl config set-credentials "kubelet-bootstrap" \
  --token=${TOKEN} \
  --kubeconfig=bootstrap.kubeconfig
$ kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=bootstrap.kubeconfig
$ kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

mv bootstrap.kubeconfig /opt/kubernetes/cfg
```

4. systemd管理kubelet

```shell
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kubelet.conf
ExecStart=/opt/kubernetes/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet
```

<font color=red>kubelet 服务启动不起来</font>

```shell
journalctl -xefu kubelet  # 查看日志
```

通常情况，再使用二进制搭建集群的时候，我们需要卸载之前我们安装的历史版本

```shell
yum list installed | grep kube
# 把历史版本卸载干净之后再进行配置
```

5. 批准kubelet证书请求

```shell
kubectl get csr  # 查看还未认证的节点，如果节点启动失败，则不会显示任何信息
NAME                                                   AGE     SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-m-RuMCihhrALjCJWwNQBKIP2JprawKzyZ6VWQPrOzCQ   6m37s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending
# 批准申请
kubectl certificate approve node-csr-m-RuMCihhrALjCJWwNQBKIP2JprawKzyZ6VWQPrOzCQ

# 查看节点
[root@K8s-Master bin]# kubectl get nodes
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   NotReady   <none>   3s    v1.18.3
# 由于网络插件还没有部署，节点会没有准备就绪 NotReady
```

###### 2. 部署kube-proxy

1. 创建配置文件

```shell
cat > /opt/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--config=/opt/kubernetes/cfg/kube-proxy-config.yml"
EOF
```

2. 创建参数文件、

```shell
cat > /opt/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /opt/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: K8s-Master
clusterCIDR: 10.0.0.0/24
EOF
```

3. 生成kube-proxy.kubeconfig

```shell
# Master生成，然后再上传到Node节点上
# 切换工作目录
cd TLS/k8s

# 创建证书请求文件
cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```

4. 生成kube-proxy.kubeconfig文件

```shell
$ KUBE_APISERVER="https://192.168.0.110:6443"
$ kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig
$ kubectl config set-credentials kube-proxy \
  --client-certificate=./kube-proxy.pem \
  --client-key=./kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
$ kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
$ kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig

$ cp kube-proxy.kubeconfig /opt/kubernetes/cfg/

```

5. systemd管理kube-proxy

```shell
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-proxy.conf
ExecStart=/opt/kubernetes/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl start kube-proxy
systemctl enable kube-proxy
systemctl status kube-proxy
```

###### 3. 部署CNI网络

下载地址：
https://github.com/containernetworking/plugins/releases/download/v0.8.6/cni-plugins-linux-amd64-v0.8.6.tgz

1. 解压二进制文件并移动到默认工作目录

```shell
$ mkdir -p /opt/cni/bin
$ tar -zxvf cni-plugins-linux-amd64-v0.8.6.tgz -C /opt/cni/bin/
```

2. 部署CNI网络

```shell
cd /opt/cni
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 更改kube-flannel.yml文件的镜像源为 lizhenliang/flannel:v0.12.0-amd64
- name: install-cni
        # image: quay.io/coreos/flannel:v0.14.0
        image: lizhenliang/flannel:v0.12.0-amd64
containers:
      - name: kube-flannel
        # image: quay.io/coreos/flannel:v0.14.0
        image: lizhenliang/flannel:v0.12.0-amd64

```

3. 应用CNI网络

```shell
kubectl apply -f kube-flannel.yml

[root@K8s-Master cni]# kubectl get pods -n kube-system
NAME                    READY   STATUS     RESTARTS   AGE
kube-flannel-ds-pctmp   0/1     Init:0/1   0          27s
# 稍稍等那么一小会，待网络启动后，node的状态会变为ready
[root@K8s-Master cni]# kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    <none>   45m   v1.18.3

```

4. 授权apiserver访问kubelet

```shell
cat > apiserver-to-kubelet-rbac.yaml<< EOF 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
      - pods/log
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

kubectl apply -f apiserver-to-kubelet-rbac.yaml
```

<font color=green>以上操作都是在Master节点操作的，即Master机器即使Master有时Worker Node</font>

接下来介绍如何新增Worker Node

###### 4. 新增加Worker Node

1. 拷贝已部署好的Node相关文件到新节点

```shell
scp -r /opt/kubernetes root@192.168.0.111:/opt
scp -r /opt/kubernetes root@192.168.0.112:/opt

scp -r /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@192.168.0.111:/usr/lib/systemd/system
scp -r /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@192.168.0.112:/usr/lib/systemd/system

 scp -r /opt/cni/ root@192.168.0.111:/opt/
 scp -r /opt/cni/ root@192.168.0.112:/opt/

scp /opt/kubernetes/ssl/ca.pem root@192.168.0.111:/opt/kubernetes/ssl/

scp /opt/kubernetes/ssl/ca.pem root@192.168.0.112:/opt/kubernetes/ssl/

```

2. 删除kubelet证书和kubeconfig文件

这几个文件是证书申请审批后自动生成的，每个Node不同，必须删除重新生成

```shell
#Node1 和 Node2 都需要删除
rm /opt/kubernetes/cfg/kubelet.kubeconfig
rm -rf /opt/kubernetes/ssl/kubelet*
```

3. 修改配置文件中的主机名

```shell
vim /opt/kubernetes/cfg/kubelet.conf 
--hostname-override=K8s-Node1

vim /opt/kubernetes/cfg/kube-proxy-config.yml 
hostnameOverride: K8s-Node1

vim /opt/kubernetes/cfg/kubelet.conf 
--hostname-override=K8s-Node2

vim /opt/kubernetes/cfg/kube-proxy-config.yml 
hostnameOverride: K8s-Node2

# 重启服务，并设置开机启动
systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet
systemctl start kube-proxy
systemctl enable kube-proxy

```

4. 在Master上批准新Node kubelet证书申请

```shell
ca.pem                                                                                                      100% 1359   241.8KB/s   00:00    
[root@K8s-Master cni]# kubectl get csr
NAME                                                   AGE    SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-0WbEBs72A03Y-WhgrPvbd91b9T8vw8hwmaQ_t4FiIJ4   96s    kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending
node-csr-Fo25Zn3aZshIrmT2WKAtfHeELEAcdq18HR_QpNlexyY   2m6s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending
node-csr-m-RuMCihhrALjCJWwNQBKIP2JprawKzyZ6VWQPrOzCQ   80m    kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Approved,Issued


$ kubectl certificate approve node-csr-0WbEBs72A03Y-WhgrPvbd91b9T8vw8hwmaQ_t4FiIJ4

$ kubectl certificate approve node-csr-Fo25Zn3aZshIrmT2WKAtfHeELEAcdq18HR_QpNlexyY

$ kubectl get node 
[root@K8s-Master cni]# kubectl get node
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   Ready      <none>   75m   v1.18.3
k8s-node1    NotReady   <none>   40s   v1.18.3
k8s-node2    NotReady   <none>   46s   v1.18.3
[root@K8s-Master cni]# kubectl get node
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    <none>   76m    v1.18.3
k8s-node1    Ready    <none>   111s   v1.18.3
k8s-node2    Ready    <none>   117s   v1.18.3
# 稍稍等一会，节点的状态就会从NotReady变为Ready
```



##### 测试Kubernetes集群

在kubernetes集群中创建一个pod，验证是否正常运行：

```shell
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
[root@K8s-Master cni]# kubectl get pod,svc
NAME                        READY   STATUS              RESTARTS   AGE
pod/nginx-f89759699-rwjd9   0/1     ContainerCreating   0          18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP        15h
service/nginx        NodePort    10.0.0.182   <none>        80:30077/TCP   9s

```

测试：http://192.168.0.110:30077









## 第三部分：K8s核心概念



1. Pod
2. Controller
3. Service
4. Ingress
5. RABC
6. Helm（类似于yun）
7. 持久化存储

### kubernetes集群命令行工具kubectl

#### 1. kubectl概述

kubectl是Kubernetes集群的命令行工具，通过kubectl能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。

#### 2. kubectl命令的语法

```shell
$ kubectl [command] [TYPE] [NAME] [flags]
```

1. command:指定要对资源执行的操作，例如：create、get、describe和delete

2. TYPE：指定资源类型，资源类型是大小写敏感的，开发者能够以单数、复数和缩略的形式。例如：

   ```shell
   $ kubectl get pod pod1
   $ kubectl get pods pod1
   $ kubectl get po pod1
   ```

3. NAME:指定资源的名称，名称也大小写敏感。如果省略名称，则会显示所有的资源，例如：

   ```shell
   $ kubectl get pods
   ```

4. flags:指定可选的参数。例如：可用-s或者-server参数指定Kubernetes API server 的地址和端口。

#### 3. kubectl help获取更多信息

```shell
$ kubectl --help
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create        Create a resource from a file or from stdin.
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run           Run a particular image on the cluster
  set           Set specific features on objects

Basic Commands (Intermediate):
  explain       Documentation of resources
  get           Display one or many resources
  edit          Edit a resource on the server
  delete        Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout       Manage the rollout of a resource
  scale         Set a new size for a Deployment, ReplicaSet or Replication Controller
  autoscale     Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate   Modify certificate resources.
  cluster-info  Display cluster info
  top           Display Resource (CPU/Memory/Storage) usage.
  cordon        Mark node as unschedulable
  uncordon      Mark node as schedulable
  drain         Drain node in preparation for maintenance
  taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe      Show details of a specific resource or group of resources
  logs          Print the logs for a container in a pod
  attach        Attach to a running container
  exec          Execute a command in a container
  port-forward  Forward one or more local ports to a pod
  proxy         Run a proxy to the Kubernetes API server
  cp            Copy files and directories to and from containers.
  auth          Inspect authorization

Advanced Commands:
  diff          Diff live version against would-be applied version
  apply         Apply a configuration to a resource by filename or stdin
  patch         Update field(s) of a resource using strategic merge patch
  replace       Replace a resource by filename or stdin
  wait          Experimental: Wait for a specific condition on one or many resources.
  convert       Convert config files between different API versions
  kustomize     Build a kustomization target from a directory or a remote url.

Settings Commands:
  label         Update the labels on a resource
  annotate      Update the annotations on a resource
  completion    Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  alpha         Commands for features in alpha
  api-resources Print the supported API resources on the server
  api-versions  Print the supported API versions on the server, in the form of "group/version"
  config        Modify kubeconfig files
  plugin        Provides utilities for interacting with plugins.
  version       Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

```



#### 4. kubectl 子命令使用分类

| 分类                   | 命令                            | 说明                                                         |
| ---------------------- | ------------------------------- | ------------------------------------------------------------ |
| **基础命令**           | create                          | 通过文件名或标准输入创建资源                                 |
|                        | expose                          | 将一个资源公开为一个新的Service                              |
|                        | run                             | 在集群中运行一个特定的功能                                   |
|                        | set                             | 在对象上设置特定的功能                                       |
|                        | get                             | 显示一个或多个资源                                           |
|                        | explain                         | 文档参考资料                                                 |
|                        | edit                            | 使用默认的编辑器编辑一个资源                                 |
|                        | delete                          | 通过文件名、标准输入、资源名称或标签选择器来删除资源         |
|                        |                                 |                                                              |
| **部署命令**           | rollout                         | 管理资源的发布                                               |
|                        | rolling-update                  | 对给定的复制控制器滚动更新                                   |
|                        | scale                           | 扩容或缩容Pod数量，Deployment、ReplicaSet、RC或Job           |
|                        | autoscale                       | 创建一个自动选择扩容或缩容并设置Pod数量                      |
|                        |                                 |                                                              |
| **集群管理命令**       | certificate                     | 修改证书资源                                                 |
|                        | cluster-info                    | 显示集群信息                                                 |
|                        | top                             | 显示资源（CPU/Memory/Storage）使用。需要Heapster运行         |
|                        | cordon                          | 标记节点不可调度                                             |
|                        | uncordon                        | 标记节点可调度                                               |
|                        | drain                           | 驱逐节点上的应用，准备下线维护                               |
|                        | taint                           | 修改节点taint标记                                            |
|                        |                                 |                                                              |
| **故障诊断和调试命令** | <font color=red>describe</font> | 显示特定资源或资源组的详细信息                               |
|                        | <font color=red>logs</font>     | 在一个Pod中打印一个容器日志。如果Pod只有一个容器，容器名称是可选的 |
|                        | attach                          | 附加到一个运行的容器                                         |
|                        | <font color=red>exec</font>     | 执行命令到容器                                               |
|                        | port-forward                    | 转发一个或多个本地端口到一个pod                              |
|                        | proxy                           | 运行一个proxy到Kubernetes API Server                         |
|                        | cp                              | 拷贝文件或目录到容器中                                       |
|                        | auth                            | 检查授权                                                     |
|                        |                                 |                                                              |
| **其他命令**           |                                 |                                                              |
| 高级命令               | apply                           | 通过文件名或标准输入对资源应用配置                           |
|                        | patch                           | 使用补丁修改、更新资源的字段                                 |
|                        | replace                         | 通过文件名或标准输入替换一个资源                             |
|                        | convert                         | 不同的API版本之间转换配置文件                                |
| 设置命令               | label                           | 更新资源上的标签                                             |
|                        | annotate                        | 更新资源上的注释                                             |
|                        | completion                      | 用于实现kubectl工具自动补全                                  |
| 其他命令               | api-versions                    | 打印受支持的API版本                                          |
|                        | config                          | 修改kubeconfig文件（用于访问API， 比如配置认证信息）         |
|                        | help                            | 所有命令帮助                                                 |
|                        | plugin                          | 运行一个命令行插件                                           |
|                        | version                         | 打印客户端和服务版本信息                                     |



### YAML文件说明

#### 1. YAML文件概述

k8s集群中对资源管理和资源对象编排部署都可以通过声明样式（YAML）文件来解决，也就是可以把需要对资源对象操作编辑到YAML格式文件中，我们把这种文件叫做资源清单文件，通过kubectl命令直接使用资源清单文件就可以实现对大量的资源对象进行编排部署了。

#### 2. YAML语法格式

- 使用空格作为缩进（不使用tap）
- 缩进的空格数目不重要，只要相同层级的元素左侧对其即可
- 低版本缩进时不允许使用tap键，只允许使用空格
- 使用#表示注释，从这个字符一直到行尾，都会被解释器忽略

#### 3. yaml文件组成部分

1. 控制器定义
2. 被控制对象

#### 4. 资源清单描述方法

1. 在k8s中，一般使用YAML格式的文件来创建符合我们预期期望的pod，这样的YAML文件成为资源清单
2. 常用字段

| 字段名     | 解释       |
| ---------- | ---------- |
| apiVersion | API版本    |
| kind       | 资源类型   |
| metadata   | 资源元数据 |
| spec       | 资源规格   |
| replicas   | 副本数量   |
| selector·  | 标签选择器 |
| template   | Pod模板    |
| metadata   | Pod元数据  |
| spec       | Pod规格    |
| containers | 容器配置   |

<font color=red>必须存在的属性</font>

| 参数名                  | 字段类型 | 说明                                                         | 备注 |
| ----------------------- | -------- | ------------------------------------------------------------ | ---- |
| version                 | String   | k8s API的版本，目前基本是v1，可以用kubectl api-version命令查询 |      |
| kind                    | String   | 这里指的是yaml文件定义的资源类型和角色，比如：Pod            |      |
| metadata                | Object   | 元数据对象，固定值写 metadata                                |      |
| metadata.name           | String   | 元数据对象的名字，这里由我们编写，比如命令Pod的名字          |      |
| metadata.namespace      | String   | 元数据对象的命名空间，由我们自身定义                         |      |
| Spec                    | Object   | 资源规格；详细定义对象，固定值写Spec                         |      |
| spec.containers[]       | list     | 这里是Spec对象的容器列表定义，是个列表                       |      |
| spec.containers[].name  | String   | 这里定义容器的名字                                           |      |
| spec.containers[].image | String   | 这里定义要用到的镜像名称                                     |      |

<font color=red>spec主要对象</font>

| 参数名                                     | 字段类型 | 说明                                                         | 备注 |
| ------------------------------------------ | -------- | ------------------------------------------------------------ | ---- |
| spec.containers[].name                     | String   | 定义容器的名字                                               |      |
| spec.containers[].image                    | String   | 定义要用到的镜像的名称                                       |      |
| **spec.containers[].imagePullPolicy**      | String   | **定义镜像拉取策略**，有Always，Never，IfNotPresent三个值可选<br />（1）Always：意思是每次尝试重新拉取镜像；（2）Never：表示仅使用本地镜像；（3）IfNotPresent：如果本地有镜像就使用本地镜像，没有就拉取在线镜像。上面三个值都没有设置的话，默认为：Always |      |
| spec.containers[].command[]                | list     | 指定容器启动命令，因为是数组可以指定多个，不指定则使用镜像打包时使用的启动命令 |      |
| spec.containers[].args[]                   | list     | 指定容器启动命令参数，因为是数组可以指定多个                 |      |
| spec.containers[].workingDir               | String   | 指定容器的工作目录                                           |      |
| spec.containers[].volumeMounts[]           | List     | 指定容器内部的存储卷设置                                     |      |
| spec.containers[].volumeMounts[].name      | String   | 指定可以被容器挂载的存储卷的名称                             |      |
| spec.containers[].volumeMounts[].mountPath | String   | 指定可以被容器挂载的容器卷的路径                             |      |
| spec.containers[].volumeMounts[].readOnly  | String   | 设置指定存储卷路径的读写模式：true或者false；默认为读写模式  |      |
| spec.containers[].ports[]                  | list     | 指定容器需要用到的端口列表                                   |      |
| spec.containers[].ports[].name             | String   | 指定端口名称                                                 |      |
| spec.containers[].ports[].containerPort    | String   | 指定容器需要监听的端口号                                     |      |
| spec.containers[].ports[].hostPort         | String   | 指定容器所在主机需要监听的端口号，默认跟上面containerPort相同，注意设置了hostPort同一台主机无法启动该容器的相同副本（因为主机的端口号不能相同，这样会冲突） |      |
| spec.containers[].ports[].protocol         | String   | 指定端口协议，支持TCP和UDP，默认值为TCP                      |      |
| spec.containers[].env[]                    | List     | 指定容器运行时所需设置的环境变量列表                         |      |
| spec.containers[].env[].name               | string   | 指定环境变量名称                                             |      |
| spec.containers[].env[].value              | string   | 指定环境变量值                                               |      |
| **Pod的资源限制**                          |          |                                                              |      |
| spec.containers[].resources                | Object   | 指定资源限制和资源强求的值（这里开始就是设置容器的资源上限） |      |
| spec.containers[].resources.limits         | Object   | 指定设置容器运行时资源的运行上限                             |      |
| spec.containers[].resources.limits.cpu     | String   | 指定cpu的限制，单位为core数，将用于docker run --cpu-shares 参数 |      |
| spec.containers[].resources.limits.memory  | String   | 指定MEM内存的限制，单位为：MIB，GIB                          |      |
| spec.containers[].resources.requests       | Object   | 指定容器启动和调度时的限制设置                               |      |
| spec.containers[].resources.cpu            | String   | cpu请求，单位为core数，容器启动时初始化可用数量              |      |
| spec.containers[].resources.memory         | String   | 内存请求，单位为MIB，GIB；容器启动的初始化可用数量           |      |

<font color=red>额外的参数</font>

| 参数名                    | 字段类型 | 索命                                                         | 备注 |
| ------------------------- | -------- | ------------------------------------------------------------ | ---- |
| spec.restartPolicy        | String   | **定义Pod重启策略**，可以选择为：Always，OnFailure，默认值为Always。（1）Always：Pod一旦终止运行，则无论容器是如何终止的，kubelet服务都将重启它；（2）OnFailure：只有Pod以非零退出码终止时，kubelet才会重启该容器。如果容器正常结束（退出码为0），则kubelet将不会重启它。（3）Never：Pod终止后，kubelet将退出码报告给Master，不会重启该Pod |      |
| spec.nodeSelector         | Object   | 定义Node的Label过滤标签，以科研：value格式指定               |      |
| spec.imagePullSecrets     | Object   | 定义pull镜像是使用secret名称，以name：secretkey格式指定      |      |
| spec.hostNetwork          | Boolean  | 定义是否使用主机网络模式，默认值为false。设置true表示使用宿主机网络，不使用docker网桥，同时设置了true将无法在同一台宿主机上启动第二个副本 |      |
| **Pod健康检查**           |          |                                                              |      |
| livenessProbe             | Object   | 存活检查；如果检查失败，将杀死容器，根据Pod的restartPolicy来操作 |      |
| readinessProbe            | Object   | 就绪检查：如果检查失败，k8s会把pod从service endpoint中剔除   |      |
| Probe支持以下三种检查方法 |          | 1. httpGet：发送HTTP请求，返回200-400范围状态码为成功<br />2. exec：执行Shell命令返回状态码是0为成功<br />3. tcpSocket:发起TCP Socket 建立成功。 |      |

####  5. 实际环境下yaml文件的快速生成方式

##### 第一种方式： 使用kubectl create，命令生成yaml文件

```shell
[root@K8s-Master ~]# kubectl create deployment web --image=nginx -o yaml --dry-run
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: "2021-07-03T08:18:42Z"
  generation: 1
  labels:
    app: web
  managedFields:
  - apiVersion: apps/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:app: {}
      f:spec:
        f:progressDeadlineSeconds: {}
        f:replicas: {}
        f:revisionHistoryLimit: {}
        f:selector:
          f:matchLabels:
            .: {}
            f:app: {}
        f:strategy:
          f:rollingUpdate:
            .: {}
            f:maxSurge: {}
            f:maxUnavailable: {}
          f:type: {}
        f:template:
          f:metadata:
            f:labels:
              .: {}
              f:app: {}
          f:spec:
            f:containers:
              k:{"name":"nginx"}:
                .: {}
                f:image: {}
                f:imagePullPolicy: {}
                f:name: {}
                f:resources: {}
                f:terminationMessagePath: {}
                f:terminationMessagePolicy: {}
            f:dnsPolicy: {}
            f:restartPolicy: {}
            f:schedulerName: {}
            f:securityContext: {}
            f:terminationGracePeriodSeconds: {}
    manager: kubectl
    operation: Update
    time: "2021-07-03T08:18:42Z"
  name: web
  namespace: default
  resourceVersion: "145011"
  selfLink: /apis/apps/v1/namespaces/default/deployments/web
  uid: 5f8fa363-1640-4900-be13-e5ee32ad2aad
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
# 可以写入到指定的文件中
[root@K8s-Master ~]# kubectl create deployment web --image=nginx -o yaml --dry-run > myyaml.yaml
W0703 18:04:22.589508   43652 helpers.go:535] --dry-run is deprecated and can be replaced with --dry-run=client.
$ cat myyaml.yaml
```

##### 第二种方式： 使用kubectl get 命令导出yaml文件

这种方式适用于已经部署好的文件

```shell
[root@K8s-Master ~]# kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    0/1     1            0           8s
# 通过get命令导出已经部署好的yaml代码
[root@K8s-Master ~]# kubectl get deploy web -o=yaml --export > myyaml.yaml
Flag --export has been deprecated, This flag is deprecated and will be removed in future.
[root@K8s-Master ~]# cat myyaml.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: null
  generation: 1
  labels:
    app: web
  managedFields:
  - apiVersion: apps/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:deployment.kubernetes.io/revision: {}
      f:status:
        f:conditions:
          .: {}
          k:{"type":"Available"}:
            .: {}
            f:lastTransitionTime: {}
            f:lastUpdateTime: {}
            f:message: {}
            f:reason: {}
            f:status: {}
            f:type: {}
          k:{"type":"Progressing"}:
            .: {}
            f:lastTransitionTime: {}
            f:lastUpdateTime: {}
            f:message: {}
            f:reason: {}
            f:status: {}
            f:type: {}
        f:observedGeneration: {}
        f:replicas: {}
        f:unavailableReplicas: {}
        f:updatedReplicas: {}
    manager: kube-controller-manager
    operation: Update
    time: "2021-07-03T10:20:44Z"
  - apiVersion: apps/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:app: {}
      f:spec:
        f:progressDeadlineSeconds: {}
        f:replicas: {}
        f:revisionHistoryLimit: {}
        f:selector:
          f:matchLabels:
            .: {}
            f:app: {}
        f:strategy:
          f:rollingUpdate:
            .: {}
            f:maxSurge: {}
            f:maxUnavailable: {}
          f:type: {}
        f:template:
          f:metadata:
            f:labels:
              .: {}
              f:app: {}
          f:spec:
            f:containers:
              k:{"name":"nginx"}:
                .: {}
                f:image: {}
                f:imagePullPolicy: {}
                f:name: {}
                f:resources: {}
                f:terminationMessagePath: {}
                f:terminationMessagePolicy: {}
            f:dnsPolicy: {}
            f:restartPolicy: {}
            f:schedulerName: {}
            f:securityContext: {}
            f:terminationGracePeriodSeconds: {}
    manager: kubectl
    operation: Update
    time: "2021-07-03T10:20:44Z"
  name: web
  selfLink: /apis/apps/v1/namespaces/default/deployments/web
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
```

### Kubernetes核心技术-Pod

#### 1. Pod概述

Pod是k8s系统中可以创建和管理的最小单元，是资源对象模型中由用户创建或部署的最小资源对象模型，也是在k8s上运行容器化应用的资源对象，其他的资源对象都是用来支撑或者扩展Pod对象功能的，比如**控制器**对象是用来管控Pod对象的，**Service**或者**Ingress**资源对象是用来暴露Pod引用对象的，PersistentVolume资源对象是用来为Pod提供存储等等，k8s不会直接处理容器，而是Pod，Pod是有一个或多个container组成。

Pod是Kubernetes的最重要概念，每一个Pod都有一个特殊的被称为“**根容器**”的**Pause容器**。**Pause容器**对应的镜像属于Kubernetes平台的一部分，除了Pause容器，每个Pod还包含一个或多个紧密相关的**用户业务容器**。

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F5333071-57db1cb170adec26.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627905475&t=88c8502c297f8f6d1aca68ed85eae33b)

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.morecoder.com%2Farticle%2F201812%2F20181230121245190929.png&refer=http%3A%2F%2Fimage.morecoder.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627905524&t=df98d88c967aa9d462ca707b5784b687)

小总结：

1. pod是k8s的最小部署单元
2. 一个pod中包含多个容器（一组容器的集合）
3. 一个pod中容器共享网络命名空间
4. pod是短暂的



**Pod存在的意义**

1. 创建容器使用docker，一个docker对应一个容器，一个容器对应一个进程，一个容器一般运行一个应用程序。（Docker中的容器是单进程的）
2. Pod是多进程设计，运行多个应用程序。
   - 一个Pod有多个容器，一个容器里面运行一个应用程序
3. Pod的存在为了亲密性应用
   - 两个应用之间进行交互
   - 网络之间调用
   - 两个应用需要频繁调用

> Pod VS 应用：
>
> ​	每个pod都是应用的一个实例，有专用的ip
>
> Pod VS 容器：
>
> ​	一个Pod可以有多个容器，彼此间共享网络和存储资源，每个Pod中有一个（根容器）Pause容器保存所有的容器状态，通过管理Pause容器，达到管理Pod中所有容器的效果
>
> Pod VS 节点：
>
> ​	同一个Pod中的容器总会被调度到相同Node节点，不同节点间Pod的通信基于虚拟二层网络技术实现
>
> Pod VS Pod：
>
> ​	普通的Pod和静态Pod

#### 2.Pod特性

##### 1. 资源共享

一个Pod里的多个容器可以共享存储和网络，可以看作一个逻辑的主机。共享的如namespace，cgroup或者其他的隔离资源。

<font color=green>多个容器共享同一network namespace，由此在一个Pod里的多个容器共享Pod的IP和端口namespace，所以一个Pod内的多个容器之间可以通过localhost来进行通信，所需要注意的是不同容器要注意不要有端口冲突即可。</font>不同的Pod有不同的IP，不同Pod内的多个容器之间通信，不可以使用IPC（如果没有特殊指定的话）通信，通常情况下使用Pod的IP进行通信。

一个Pod里的多个容器可以共享存储卷，这个存储卷会被定义为Pod的一部分，并且可以挂载到该Pod里的所有容器的文件系统上。



##### 2. 声明周期短暂

Pod属于生命周期比较短暂的组件,比如,当Pod所在节点发生故障,那么该节点上的Pod会被调度到其他节点,但需要注意的是,被重新调度的Pod是一个全新的Pod,跟之前的Pod没有半毛钱关系.

##### 3. 平坦的网络

k8s集群中的所有Pod都在同一个共享网络地址空间中,也就是说每个Pod都可以通过其他Pod的IP地址来实现访问.

#### 3. Pod的调度机制

```shell
$ kubectl get pods -o wide
```

**Pod创建流程：**

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Foscimg.oschina.net%2Foscnet%2F04062caf14fd406c4cbdc5fbe5229116b61.png&refer=http%3A%2F%2Foscimg.oschina.net&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1628003614&t=36360dd0a7039cb66c3ab4363f2b3b0d)



##### 1. 影响调度的属性

1. **Pod资源限制对Pod调度产生影响**

   - 根据request找到足够node节点进行调度

2. **节点选择器标签影响Pod调度**

   - ```yaml
     nodeSelector:
      env_role: dev  # 具体节点的标签名（可以是一组）
     ```

   - ```shell
     $ kubectl label node node1 env_role=dev  # 为节点1打一个标签env_role=dev
     # 可以通过如下命名查看节点的标签明细
     $ kubectl get nodes k8snode1 --show-lablels
     
     [root@K8s-Master ~]# kubectl label node k8s-node1 env_role=dev 
     node/k8s-node1 labeled
     [root@K8s-Master ~]# kubectl get nodes k8s-node1 --show-labels
     NAME        STATUS   ROLES    AGE   VERSION   LABELS
     k8s-node1   Ready    <none>   24h   v1.18.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,env_role=dev,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
     
     [root@K8s-Master ~]# kubectl label node k8s-node1 env-role2=dev2
     node/k8s-node1 labeled
     [root@K8s-Master ~]# kubectl get nodes k8s-node1 --show-labels
     NAME        STATUS   ROLES    AGE   VERSION   LABELS
     k8s-node1   Ready    <none>   24h   v1.18.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,env_role=dev,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
     ```

3. **节点亲和性影响Pod调度**

   - ```yaml
     nodeAffinity:
      # nodeAffinity和之前的nodeSelector基本一样的，根据节点上标签约束来决定Pod调度到哪些节点上
     # 功能比nodeSelector要强大
     ```

   - 硬亲和性

     - ```yaml
       # 硬亲和性，约束条件必须满足
       spec:
        affinity:
         nodeAffinity:
          requiredDuringSchedulingIgnoreDuringExecution:
           nodeSelectorTerms:
           - matchExpressions:
            - key: env_role
              operator: In  # 操作符， in 代表值在下面两个里面
              values:
              - dev
              - test
       ```

     - 软亲和性

     ```yaml
     # 尝试满足，不保证
     spec:
      affinity:
       nodeAffinity:
        preferredDuringSchedulingIgnoreDuringExecution:
        - weight: 1 #权重
          preference:
           matchExpressions:
           - key: group
             operator: In
             values:
             - otherprod
     ```
     - ```yaml
       # 常用的操作符
       In：在里面
       NotIn：不在里面
       Exists：存在
       Gt：大于
       Lt：小于
       DoesNotExists：不存在
       # 使用Not可实现反亲和性
       ```

4. **Taint** **污点和污点容忍**

   > nodeSelector和nodeAffinity将Pod调度到某些节点上，是Pod中的属性，调度的时候实现
   >
   > Taint污点：节点不做普通分配调度，它是节点的属性
   >
   > - <font color=green>污点值有三个:</font>
   >   - <font color=green>NoSchedule：一定不被调度</font>
   >   - <font color=green>PreferNoSchdule： 尽量不被调用</font>
   >   - <font color=green>NoExecute：不会调度，并且还会驱逐Node已有的Pod</font>

   - 应用场景

     - 专用节点
     - 配置特点硬件节点
     - 基于Taint驱逐

   - 具体演示

     - ```shell
       # 1. 查看节点污点情况
       [root@K8s-Master ~]# kubectl describe nodes k8s-master | grep Taint
       Taints:             <none>
       # 2. 为节点添加污点
       $ kubectl taint node [节点主机名：k8s-master] key=value:污点的三个值
       
       # 测试
       # 创建一个pod，然后设置污点，然后增加pod的副本
       $ kubectl create deployment web --images=nginx
       $ kubectl get pods -o wide
       NAME                   READY   STATUS              RESTARTS   AGE   IP       NODE        NOMINATED NODE   READINESS GATES
       web-5dcb957ccc-zmj4x   0/1     ContainerCreating   0          75s   <none>   k8s-node1   <none>           <none>
       [root@K8s-Master ~]# kubectl scale deployment web --replicas=5
       deployment.apps/web scaled
       [root@K8s-Master ~]# kubectl get pods -o wide
       NAME                   READY   STATUS              RESTARTS   AGE     IP       NODE         NOMINATED NODE   READINESS GATES
       web-5dcb957ccc-7mw6s   0/1     ContainerCreating   0          4s      <none>   k8s-master   <none>           <none>
       web-5dcb957ccc-dl84z   0/1     ContainerCreating   0          4s      <none>   k8s-node2    <none>           <none>
       web-5dcb957ccc-tqf5k   0/1     ContainerCreating   0          4s      <none>   k8s-node1    <none>           <none>
       web-5dcb957ccc-z747m   0/1     ContainerCreating   0          4s      <none>   k8s-node2    <none>           <none>
       web-5dcb957ccc-zmj4x   0/1     ContainerCreating   0          4m49s   <none>   k8s-node1    <none>           <none>
       # 删除pod 重新建立
       $ kubectl delete deployment web
       $ kubectl taint node k8s-node2 key=value:NoSchedule
       node/k8s-node2 tainted
       $ kubectl scale deployment web --replicas=5
       $ kubectl get pods -o wide
       [root@K8s-Master ~]# kubectl scale deployment web --replicas=5
       deployment.apps/web scaled
       [root@K8s-Master ~]# kubectl get pods -o wide
       NAME                   READY   STATUS              RESTARTS   AGE    IP       NODE         NOMINATED NODE   READINESS GATES
       web-5dcb957ccc-55k2m   0/1     ContainerCreating   0          16s    <none>   k8s-master   <none>           <none>
       web-5dcb957ccc-9jhqm   0/1     ContainerCreating   0          16s    <none>   k8s-node1    <none>           <none>
       web-5dcb957ccc-hmsrj   0/1     ContainerCreating   0          16s    <none>   k8s-master   <none>           <none>
       web-5dcb957ccc-mc2qz   0/1     ContainerCreating   0          2m8s   <none>   k8s-node1    <none>           <none>
       web-5dcb957ccc-nt956   0/1     ContainerCreating   0          16s    <none>   k8s-node1    <none>           <none>
       
       # 查看node2上的污点的情况
       $ kubectl describe node k8s-node2 | grep Taint
       [root@K8s-Master ~]# kubectl describe node k8s-node2 | grep Taint
       Taints:             env_role=dev:NoSchedule
       # 删除污点
       $ kubectl taint node k8s-node2 env_role:NoSchedule-
       [root@K8s-Master ~]# kubectl taint node k8s-node2 env_role:NoSchedule-
       node/k8s-node2 untainted
       [root@K8s-Master ~]# kubectl describe node k8s-node2 | grep Taint
       Taints:             <none>
       ```
       
     - 污点容忍
     
       - ```yaml
         spec:
          tolerations:   # 虽然设置了污点，但是也可能被调度到
          - key: "key"
            operator: "Equal"
            value: "value"
            effect: "NoSchedule"
          containers:
          - name: webdemo
            image:nginx
         ```



### Kubernetes核心技术-controller



#### 1.  什么是controller？











#### 2. Pod是Controller关系











#### 3. Deployment控制器应用场景









#### 4. yaml文件字段说明







#### 5.Deployment控制器部署应用







#### 6. 升级回滚









#### 7. 弹性伸缩





























































## 第四部分： 搭建集群监控平台系统





## 第五部分：从零搭建高可用K8s集群





## 第六部分：在集群环境部署项目







