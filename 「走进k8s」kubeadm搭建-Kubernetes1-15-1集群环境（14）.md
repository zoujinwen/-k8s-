

#### （一）kubeadm
* ①官网


* ②介绍


* ③环境准备


> docker的安装，这个是前提

``` bash
sudo curl -sSL https://get.docker.com/ | sh
sudo curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://b81aace9.m.daocloud.io
sudo systemctl restart docker
sudo yum -y install epel-release
sudo yum -y install python-pip
sudo yum clean all
sudo pip install docker-compose
```

``` bash
docker info
# Cgroup Driver: cgroupfs

```
>修改docker的Cgroup Driver
>修改/etc/docker/daemon.json文件

``` bash
sudo curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://b81aace9.m.daocloud.io

vi /etc/docker/daemon.json
```

``` bash
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```


> 重启docker

``` bash
systemctl daemon-reload
systemctl restart docker
docker info 
```


> 增加dns解析，所有机器

``` bash
echo nameserver 8.8.8.8 >> /etc/resolv.conf
systemctl restart network
```

> 增加host文件，所有机器

``` bash
cat /etc/hosts
192.168.86.100  master
192.168.86.101  node1
192.168.86.102  node2
192.168.86.103  node3
```



``` bash
systemctl stop firewalld
systemctl disable firewalld
```

> 禁用SELINUX，所有机器

``` bash
setenforce 0
cat /etc/selinux/config
#SELINUX=disabled

```

> 创建/etc/sysctl.d/k8s.conf文件

``` bash
vi /etc/sysctl.d/k8s.conf

#增加下面内容
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```


> 执行如下命令

``` bash
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```




> 公司内服务器域名解析总有问题，时好时不好，很烦，这里直接用hosts做解析

``` bash
vi /etc/hosts
#添加ping下看看这个ip可用不，找个可用的ip
61.168.101.240  mirrors.aliyun.com

```


* ④配置国内数据源

``` bash

yum install -y wget

mkdir /etc/yum.repos.d/bak && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak


wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo

wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo

yum clean all && yum makecache
```

* ⑤配置国内Kubernetes源

``` bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

EOF
```


* ⑥安装kubeadm、kubelet、kubectl


``` bash
yum install -y kubelet kubeadm kubectl
#或者跟下面阿里的镜像保持一致
#yum install -y kubelet-1.15.1-0 kubeadmt-1.15.1-0 kubectlt-1.15.1-0
systemctl enable kubelet
```

* ⑦关闭Swap

``` bash
swapoff -a
systemctl enable docker.service
systemctl enable kubelet.service
```

* ⑧初始化 master节点

``` bash
kubeadm init --kubernetes-version=v1.15.1 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.86.100 --ignore-preflight-errors=Swap
```





``` bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1


docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.1 k8s.gcr.io/kube-apiserver:v1.15.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.1 k8s.gcr.io/kube-controller-manager:v1.15.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.1 k8s.gcr.io/kube-scheduler:v1.15.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.1 k8s.gcr.io/kube-proxy:v1.15.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
```


``` bash
vi master.sh
#把上面的代码复制进来

chmod 777 master.sh
sh ./master.sh
```


> 查看镜像

``` bash
docker images
```





``` bash
kubeadm init --kubernetes-version=v1.15.1 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.86.100 --ignore-preflight-errors=Swap
```



> 在执行节点上执行如下操作，初始化一下k8s环境。

``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



> kubectl 默认使用的配置文件

``` bash
cat ~/.kube/config
```


* ⑨ flannel网络设置 master节点

``` bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f  kube-flannel.yml
```

> 所有下面的pods

``` bash
kubectl get pods --all-namespaces
```



``` bash
kubectl get nodes
```




* ⑩ token 和 sha256 值 master节点


``` bash
kubeadm token list

 openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```



* ⑩k8s node节点添加到master节点上


``` bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.1


docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.1 k8s.gcr.io/kube-proxy:v1.15.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1

```

``` bash
vi node.sh
# 复制上边的镜像拉取和tag
chmod 777 node.sh
sh ./node.sh
```


``` bash
kubeadm join 192.168.86.100:6443 --token ukoxff.qwxf33of4dievupw --discovery-token-ca-cert-hash sha256:0e11e4f1e7ad3906551599ef052ee9eaa86a3cef7309339997ea1ecc2f44af23
```


* ⑪加入node节点后，进入master节点查看

``` bash
kubectl get nodes
```


* ⑫node节点也想查看


* ⑬查看master状态

``` bash
kubectl get cs
```



* ⑭ 重启k8s的服务


``` bash
systemctl restart kubelet 
```


#### （二）kubeadm 清除


``` bash
systemctl stop kubelet
docker rm -f -v $(docker ps  -a -q)

rm -rf /etc/kubernetes
rm -rf  /var/lib/etcd
rm -rf   /var/lib/kubelet
rm -rf  $HOME/.kube/config
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

yum reinstall -y kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet
```


#### （三）重启电脑导致虚拟机关机如何重启k8s


* ① 需要关闭交换内存

``` bash
swapoff -a
systemctl status kubelet 
systemctl restart kubelet 
```


* ②设置swap开机不启动



``` bash
vi /etc/fstab
```


* ③ k8s命令补全功能


``` bash
yum install -y bash-completion*
# 添加环境变量

source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

```

* ④kubectl logs、exec、port-forward 执行失败问题解决

``` bash
kubectl get nodes

```



``` bash
vi /etc/sysconfig/kubelet
#master节点的配置
#--node-ip=192.168.86.100

#node节点的配置
#--node-ip=192.168.86.101
```


> 重启服务，所有节点master和node

``` bash
service kubelet restart
kubectl get nodes
```




 > 注意：因为我是用vagrant创建的虚拟机，需要修改/etc/sysconfig/kubelet，如果是正常安装的虚拟机，可能需要修改

``` bash
# 所有节点 master和node
vi /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# master节点的配置
# Environment="KUBELET_EXTRA_ARGS=--node-ip=192.168.86.100"

# node节点的配置
# Environment="KUBELET_EXTRA_ARGS=--node-ip=192.168.86.101"
```






``` bash
ifconfig
#enp0s8 有个唯一的ip
```



``` bash
sudo kubectl edit daemonset kube-flannel-ds-amd64 -n kube-system
```

``` bash
- args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=enp0s8
```



``` bash
kubectl get pod -n kube-system

#找到上边所有带kube-flannel-ds-amd64-*
kubectl delete pod -n kube-system \
kube-flannel-ds-amd64-mgvp9\
kube-flannel-ds-amd64-xpjqx
```


``` bash
service kubelet restart 
```





