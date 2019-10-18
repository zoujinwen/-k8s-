
#### （一）Docker Machine


``` bash
sudo curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine
sudo chmod +x /usr/local/bin/docker-machine
docker-machine -v
```


#### （二）docker-machine来创建virtualbox虚拟机



``` bash
yum -y install kernel-devel

yum update kernel*

yum -y install wget

wget http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc

rpm --import oracle_vbox.asc

wget http://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo -O /etc/yum.repos.d/virtualbox.repo

yum install VirtualBox-6.0.x86_64 

sudo /sbin/vboxconfig
#需要重启下kernel的需要
reboot
yum install kernel-devel 
yum install kernel
```



* ②创建一台 Docker 主机

``` bash
 docker-machine create -d virtualbox default
```

* ③进入Docker主机

``` bash
docker-machine ssh default
```


* ④docker常用命令解释

``` bash
docker-machine 命令 主机
```



#### （二）为什么Docker Machine
