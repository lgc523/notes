---
title: "K8s 安装"
date: 2021-08-11T01:06:35+08:00
draft: true
author: "spider"
toc: false
tags:
  - cloud
  - k8s
---

```
1.systemctl stop firewalld && systemctl disable firewalld

2.sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config;cat /etc/selinux/config

3.sed -i.bak '/swap/s/^/#/' /etc/fstab

4.cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

5.sysctl -p /etc/sysctl.d/k8s.conf

6.cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

7.yum clean all

8.yum -y makecache

9.yum install -y yum-utils   device-mapper-persistent-data   lvm2

10.yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

11.yum install docker-ce-19.03.13 docker-ce-cli-19.03.13 containerd.io -y

12.systemctl enable --now docker

13.vi /etc/docker/daemon.json
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"],
  "insecure-registries":["guangchang.tech:5000"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

14.systemctl daemon-reload

15.systemctl restart docker

16.yum install -y kubelet-1.19.2 kubeadm-1.19.2 kubectl-1.19.2

17.kubeadm init phase preflight

18.echo "source <(kubectl completion bash)" >> ~/.bash_profile  source .bash_profile

19.vim image.sh
#!/bin/bash
url=registry.aliyuncs.com/google_containers
version=v1.19.2
images=(`kubeadm config images list --kubernetes-version=$version|awk -F '/' '{print $2}'`)
for imagename in ${images[@]} ; do
  docker pull $url/$imagename
  docker tag $url/$imagename k8s.gcr.io/$imagename
  docker rmi -f $url/$imagename
done

20.bash image.sh

21. # /var/lib/kubelet/kubeadm-flags.env cni

21.kubeadm init --apiserver-advertise-address=49.232.212.110 --kubernetes-version v1.19.2 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16

22.slave join
```
