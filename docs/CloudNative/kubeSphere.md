---
sidebar_position: 3
---

# kubeSphere

- 文档

  [什么是 KubeSphere](https://www.kubesphere.io/zh/docs/v3.3/introduction/what-is-kubesphere/)

## 一、在 kubernetes 集群上安装 kubeSphere

### 1. 准备多台服务器（配置可以高点）

- 例：4核8g；centos 7.9；安全组打开 30000-32767 端口

### 2. 安装 docker

```bash
#移除旧的 docker
sudo yum remove docker*
# 安装yum
sudo yum install -y yum-utils

#配置docker的yum地址
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#安装指定版本 docker
sudo yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7 containerd.io-1.4.6
# 启动&开机启动docker
systemctl enable docker --now

# docker加速配置
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors":["https://82m9ar63.mirror.aliyuncs.com"],
	"exec-opts": ["native .cgroupdriver=systemd"],
	"log-driver":"json-file",
	"log-opts": {
		"max-size":"100m"
	},
	"storage-driver":"overlay2"
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker #重启 docker
```



### 3. 安装 kubernetes

1. 基本环境

   > 每个服务器使用内网互通
   >
   > 每个服务器配置独有的 hostname，不能使用 localhost

   ```bash
   #设置每个机器自己的 hostname
   hostnamectl set-hostname xxx
   
   #将 SELinux 设置为 permissive 模式(相当于将其禁用)
   sudo setenforce 0
   sudo sed -i's/^SELINUX=enforcing$/SELINUX=permissive/’ /etc/selinux/config
   
   #关闭swap
   swapoff -a
   sed -ri 's/.*swap.*/#&/' /etc/fstab
   
   #允许 iptables 检查桥接流量
   cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
   br netfilter
   EOF
   
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   sudo sysctl --system
   ```

2. 安装 kubelet、kubeadm、kubectl

   ```bash
   #配置 k8s 的 yum 源地址
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=0
   repo_gpgcheck=0
   gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   	http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   
   #安装 kubelet，kubeadm， kubectl
   sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9
   
   #启动kubelet
   sudo systemctl enable --now kubelet
   
   #给所有机器添加 master 域名
   echo "master服务器IP地址 k8s-master" >> /etc/hosts
   
   #确认是否 ping 通 master 节点
   ping k8s-master
   ```

3. 初始化 master 主节点

   > 记得在安全组暴露  k8s 集群对外访问的端口（30000 - 32767 之间）

   ```bash
   # 1. 初始化主节点
   kubeadm init \
   --apiserver-advertise-address=172.31.0.4 \
   --control-plane-endpoint=k8s-master \
   --cloud_native-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
   --kubernetes-version v1.20.9 \
   --service-cidr=10.96.0.0/16 \
   --pod-network-cidr=192.168.0./16
   
   #2. 记录关键信息（参考初始化后输出的日志信息）
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   
   #3. 安装网络组件（calico）
   curl https://docs.projectcalico.org/manifests/calico.yaml -0
   kubectl apply -f calico.yaml
   ```

4. 加入多个 worker 节点（参考初始化主节点后输出的日志信息，如下例子）

   ```bash
   # Then you can join any number of worker nodes by running the following on each as root:
   kubeadm join k8s-master:6443 --token 3vckmv.lvr105xpyftbs177
   --discovery-token-ca-cert-hash sha256:1dc274fed24778f5c284229d9fcba44a5df11efba018f9664cf5e8ff779
   ```

### 4. 安装 kubeSphere 前置环境

#### 1. nfs文件系统

> 完成目录挂载动态分配容量

1. 安装 nfs-server 文件系统

   ```bash
   #1. 在每个机器安装 nfs-utils
   yum install -y nfs-utils
   
   #2. 在master 执行以下命令
   echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)” > /etc/exports
   
   #3.  执行以下命令，启动 nfs 服务;创建共享目录
   mkdir -p /nfs/data
   # 在master执行
   systemctl enable rpcbind
   systemctl enable nfs-server
   systemctl start rpcbind
   systemctl start nfs-server
   #使配置生效
   exportfs -r
   #检查配置是否生效
   exportfs
   ```

2. 配置 nfs-client 

   > 用于 worker 节点同步 master 节点配置好的共享目录

   ```BASH
   #在 worker 节点执行
   showmount -e master节点IP
   mkdir -p /nfs/data
   mount -t nfs master节点IP:/nfs/data /nfs/data
   ```


#### 2. 安装 metrics-server

> metrics-server： 集群指标监控组件

- 安装步骤参考
  - [Kubernetes上安装Metrics-Server-CSDN博客](https://blog.csdn.net/cjj2006/article/details/135138843)

### 5. 安装 kubeSphere

- 参考文档
  - [在 Kubernetes 上最小化安装 KubeSphere](https://www.kubesphere.io/zh/docs/v3.4/quick-start/minimal-kubesphere-on-k8s/)
  - 一切步骤均以官方文档为主

```bash
yum install -y wget
yum install -y vim
# 此处可以先使用 wget 工具先把 yaml 文件线下载下来，再使用 vim 编辑器修改相关配置（启用可插拔组件）后再执行以下命令

kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.1/kubesphere-installer.yaml

kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.1/cluster-configuration.yaml

```

## 二、Linux 单节点部署 kubeSphere

- 参考官方文档
  - [在 Linux 上以 All-in-One 模式安装 KubeSphere](https://www.kubesphere.io/zh/docs/v3.4/quick-start/all-in-one-on-linux/)
  - 一切步骤均以官方文档为主

### 1. 准备一台 Linux 服务器

- 例：4核8g；centos 7.9；安全组打开 30000-32767 端口

- 指定 hostname

  ```bash
  hostnamectl set-hostname xxxx
  ```

### 2. 安装

#### 1. 准备 kubkey

#### 2. 使用 kubkey 引导安装集群

#### 3. 安装后开启功能（启用可插拔组件）

> 默认安装最小化的 kubeSphere

- 参考文档
  - [启用可插拔组件 (kubesphere.io)](https://www.kubesphere.io/zh/docs/v3.4/pluggable-components/)

## 三、Linux 多节点部署 kubeSphere

- 参考官方文档
  - [多节点安装 (kubesphere.io)](https://www.kubesphere.io/zh/docs/v3.4/installing-on-linux/introduction/multioverview/)
  - 一切步骤均以官方文档为主

### 1. 准备多台服务器（配置可以高点）

- 例：
  - 4核8g（master 节点）；
  - 8核16g（worker 节点）
  - centos 7.9；
  - 内网互通
  - 指定唯一 hostname
  - 安全组打开 30000-32767 端口

### 2. 准备 kubkey

### 3. 创建集群配置文件

> 创建好后可以对配置文件进行修改

### 4. 使用配置文件创建集群

> 创建过程中提示需要安装一些组件，安装上即可

## 四、kubeSphere 多租户

- 参考文档
  - [创建企业空间、项目、用户和平台角色 (kubesphere.io)](https://www.kubesphere.io/zh/docs/v3.4/quick-start/create-workspace-and-project/)

### 1. 创建多角色用户

- 多种内置角色类型，可以给用户分配相关权限
  - platform-self-provisioner
  - platform-regular
  - platform-admin

### 2. 企业空间

- 企业空间也内置多种角色
  - demo-workspace-admin
  - demo-workspace-self-provisioner
  - demo-workspace-viewer
- 可在企业空间邀请用户并指定用户企业空间角色
- 可在企业空间创建项目并邀请用户加入项目

### 3.项目

- 创建项目
- 邀请成员加入项目

## 五、使用 kubeSphere 在 kubernetes 集群上部署应用

- 参考文档
  - [创建并部署 WordPress (kubesphere.io)](https://www.kubesphere.io/zh/docs/v3.4/quick-start/wordpress-deployment/)

- 按需部署 MySQL，Redis，ES，Mq（有状态工作负载）
  - 创建工作负载（有状态副本集）
    - 创建配置集并挂载（configMap）
    - 创建存储卷并挂载（pvc）
  - 为指定负载指定服务
    - 内网访问（clusterIp）
    - 外网访问（nodePort）
- 应用商店部署（应用）
- 应用仓库部署
  - helm
    - 参考文档
      - [Helm | Chart](https://helm.sh/zh/docs/topics/charts/)
  - 管理员角色的成员可以在应用管理中添加 helm 仓库

## 六、kubeSphere 部署上云实战

- ruoyi-cloud 项目上云
  - [yangzongzhuan/RuoYi-Cloud: :tada: (RuoYi)官方仓库 基于Spring Boot、Spring Cloud & Alibaba的分布式微服务架构权限管理系统 (github.com)](https://github.com/yangzongzhuan/RuoYi-Cloud)