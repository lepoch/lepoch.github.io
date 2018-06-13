---
title: Go小记-初识
date: 2018-03-08 17:00:00
tags:
 - kubernetes
 - k8s
 - kubeadmin

categories:
 - 运维

---

跟着官方文档，使用kubeadmin搭建kubernetes。

<!-- MORE -->
### 环境
6台阿里云ecs，CPU： 2核    内存：4 GB  ， centOS 7.4 64位， root用户
- k8s1    192.168.0.248
- k8s2    192.168.0.249
- k8s3    192.168.0.250
- k8s4    192.168.0.251
- k8s5    192.168.0.252
- k8s6    192.168.0.247



```
yum install -y docker
systemctl enable docker && systemctl start docker

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

sed -i "s/cgroup-driver=systemd/cgroup-driver=systemd/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload
systemctl restart kubelet


echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bash_profile
source ~/.bash_profile


# kubeadm join --token 0c13a5.bf9bda1f47c070d3 192.168.0.131:6443 --discovery-token-ca-cert-hash sha256:4576241c6828d3b635bf3cab14cfce7eee7e5016502fd1cb69fbea1a7ea87ea2
```



```
yum install -y docker
systemctl enable docker && systemctl start docker

cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF

setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

sed -i "s/cgroup-driver=systemd/cgroup-driver=systemd/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload
systemctl restart kubelet

# kubeadm join --token 6b71c7.2816073a294e1f3d 192.168.0.131:6443 --discovery-token-ca-cert-hash sha256:f8db9e3bd9aa5e0c9b5954a54e0dbbec3d70e1133423c257ff474f479a87333a

```

### 安装
#### 安装docker
```
yum install -y docker
systemctl enable docker && systemctl start docker
```

#### 安装kubeadm，kubelet和kubectl
- kubeadm：引导群集的命令。
- kubelet：在群集中的所有机器上运行的组件，并执行诸如启动荚和容器之类的组件。
- kubectl：命令行util与您的群集通话。

天朝网络不可达，得另寻出路
```
cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF

setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

```

#### 在主节点上配置kubelet使用的cgroup驱动程序  
查看驱动：
```
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```


修改cgroup-driver为docker使用的驱动
```
sed -i "s/cgroup-driver=systemd/cgroup-driver=systemd/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
重启kubelet
```
systemctl daemon-reload
systemctl restart kubelet
```

### 创建集群

#### 初始化master  
指定版本，不然就得翻墙。。。
```
kubeadm init --kubernetes-version=1.9.0 --token-ttl 0
```
- root用户：`export KUBECONFIG=/etc/kubernetes/admin.conf`
