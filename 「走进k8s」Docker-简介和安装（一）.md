

#### （七）Docker 安装

``` bash
vi /etc/resolv.conf
#nameserver 8.8.8.8 

# 或者直接通过这个命令 echo nameserver 8.8.8.8 >> /etc/resolv.conf 
systemctl restart network
```

* ② docker在线安装

``` bash
sudo curl -sSL https://get.docker.com/ | sh
```


* ③ docker 加速器

``` bash
sudo curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://b81aace9.m.daocloud.io
sudo systemctl restart docker
```

* ⑤ 安装配合工具

``` bash
sudo yum -y install epel-release
sudo yum -y install python-pip
sudo yum clean all
```


* ⑤ 安装docker-compose

``` bash
sudo pip install docker-compose
```


* ⑥ 安装完毕

``` bash
docker-compose version
docker version
```

