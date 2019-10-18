

#### （一）准备工作

``` bash
kubectl get all -n kube-ops
```


* ② 直接通过vagrant的方式 创建2台虚拟机，一共4台虚拟机。



* ③ 设计流程



#### （二）192.168.86.102 安装配置gitlab（一）

* ① 拉取镜像


``` bash
sudo curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://b81aace9.m.daocloud.io

sudo systemctl restart docker

docker pull gitlab/gitlab-ce:latest
```

![](https://upload-images.jianshu.io/upload_images/11223715-856a1170863ccb61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/11223715-d7d15c96a7178788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





* ② 编写脚本


``` bash
#!/bin/bash
HOST_NAME=gitlab.idig8.com
GITLAB_DIR=`pwd`
docker stop gitlab
docker rm gitlab
docker run -d \
    --hostname ${HOST_NAME} \
    -p 8443:443 -p 8080:80 -p 2222:22 \
    --name gitlab \
    -v ${GITLAB_DIR}/config:/etc/gitlab \
    -v ${GITLAB_DIR}/logs:/var/log/gitlab \
    -v ${GITLAB_DIR}/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```




* ③ 修改host文件


* ④ 运行脚本

``` bash
sh gitlab-start.sh
```



* ⑤ 查看gitlab信息

> http://gitlab.idig8.com:8080/




* ⑤ 修改ssh端口（如果主机端口使用的不是22端口）

``` bash
vi config/gitlab.rb 
```

``` bash
找到这一行：# gitlab_rails['gitlab_shell_ssh_port'] = 22
把22修改为你的宿主机端口（这里是2222）。然后将注释去掉。
```


> 重启服务

``` bash
docker restart gitlab
```


* ⑤ 设置root的密码



* ⑥ 登录



* ⑦ 新建立一个项目









* ⑧ 配置 ssh key



``` bash
#先看看是不是已经有啦，如果有内容就直接copy贴过去就行啦
cat ~/.ssh/id_rsa.pub

#如果上一步没有这个文件 我们就创建一个，运行下面命令（邮箱改成自己的哦），一路回车就好了
ssh-keygen -t rsa -C "394498036@qq.com"
cat ~/.ssh/id_rsa.pub
```


* ⑨ 测试功能

``` bash
 git clone http://gitlab.idig8.com:8080/liming/demo.git
```





#### （三）192.168.86.102 安装配置gitlab（第二种方式）

* ① 确认docker-compose安装是否成功

``` bash
docker-compose version
```


* ② docker-compose脚本
``` yml
version: '2'

services:
  redis:
    restart: always
    image: sameersbn/redis:latest
    # 如果拉取失败，呵呵，换用下面的阿里云源即可
    # image: registry.cn-hangzhou.aliyuncs.com/acs-sample/redis-sameersbn
    volumes:
    - /srv/docker/gitlab/redis:/var/lib/redis:Z

  postgresql:
    restart: always
    image: sameersbn/postgresql:latest
    # 如果拉取失败，呵呵，换用下面的阿里云源即可
    # image: registry.cn-hangzhou.aliyuncs.com/acs-sample/postgresql-sameersbn
    volumes:
    - /srv/docker/gitlab/postgresql:/var/lib/postgresql:Z
    environment:
    - DB_USER=gitlab
    - DB_PASS=password
    - DB_NAME=gitlabhq_production
    - DB_EXTENSION=pg_trgm

  gitlab:
    restart: always
    image: sameersbn/gitlab:latest
    # 如果拉取失败，呵呵，换用下面的阿里云源即可
    # image: registry.cn-hangzhou.aliyuncs.com/acs-sample/gitlab-sameersbn
    depends_on:
    - redis
    - postgresql
    ports:
    - "10080:80"
    - "10022:22"
    volumes:
    - /srv/docker/gitlab/git/data:/home/git/data:Z
    environment:
    # - DEBUG=false

    # postgresql 配置
    - DB_ADAPTER=postgresql
    - DB_HOST=postgresql
    - DB_PORT=5432
    - DB_USER=gitlab
    - DB_PASS=password
    - DB_NAME=gitlabhq_production

    # redis 配置
    - REDIS_HOST=redis
    - REDIS_PORT=6379

    # 端口配置
    - GITLAB_PORT=10080
    - GITLAB_SSH_PORT=10022
    - GITLAB_HOST=192.168.86.102

    # CI 所使用的加密密钥
    - GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alphanumeric-string
    # Session 加密密钥
    - GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alphanumeric-string
    # 数据库2FA密钥
    - GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alphanumeric-string
```



* ② 启动docker-compse

``` bash
docker-compose -f  docker-compose.yml up -d
```



* ③ 查看gitlab状态

``` bash
docker ps -a
```


* ④  设置语言




#### （四）192.168.86.104 安装配置Harbor


> 拷贝到自身目录

``` bash
cd /
ll
cp /vagrant/harbor-offline-installer-v1.9.1-rc1.tgz  ~
cd ~
ll
```


* ② 安装程序


``` bash
tar -xvf harbor-offline-installer-v1.9.1-rc1.tgz 
```



> 修改下主机信息

``` bash
cd harbor
vi harbor.yml
```



> 开始安装

``` bash
sh install.sh 
```


* ④  登录



*  ③ 配置http允许访问

``` bash
#新建docker目录
mkdir -p /etc/docker 
#新建daemon.json文件
vi /etc/docker/daemon.json 
```

> 编辑如下内容

``` bash
{ "insecure-registries":["192.168.86.104"] }
```



> 重启docker服务

``` bash
systemctl restart docker
docker login 192.168.86.104
# 用户名admin
# 密码 Harbor12345
```



> docker重启后，有容器启动失败，需要start对应的容器ID

``` bash
docker ps -a
docker start 容器ID
```

#### （五）192.168.86.103 安装配置Jenkins

* ① 一键安装jenkins脚本

``` bash
# @Author: liming
# @Date:   2019-10-10 00:16:59
# @Last Modified by:   liming
# @Last Modified time: 00:17:16
# @urlblog idig8.com
# 个人公众号  编程坑太多

#!/bin/bash
SOFT_PATH=/opt/soft

if [ ! -d $SOFT_PATH ];then
mkdir $SOFT_PATH
else
echo "文件夹已经存在"
fi

yum install -y wget 
#install jdk1.8
cd $SOFT_PATH
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
tar -zxvf jdk* -C $SOFT_PATH
cd jdk*
JAVA_HOME=`pwd` 

#install maven3.2.3
cd $SOFT_PATH
wget https://archive.apache.org/dist/maven/maven-3/3.2.3/binaries/apache-maven-3.2.3-bin.tar.gz
tar -zxvf apache-maven-3.2.3-bin.tar.gz -C $SOFT_PATH
mv apache-maven-3.2.3 maven-3.2.3
cd maven*
MAVEN_HOME=`pwd`

#install git 2.8.0
cd $SOFT_PATH
yum -y install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils- MakeMaker
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.8.0.tar.gz
tar -zxvf git-2.8.0.tar.gz -C $SOFT_PATH
cd git*
./configure
make install
ln -s /usr/local/bin/git /usr/bin/git

#追加环境变量
echo "export JAVA_HOME=${JAVA_HOME}" >> /etc/profile
echo "export PATH=$""JAVA_HOME/bin:$""PATH" >> /etc/profile
echo "export MAVEN_HOME=${MAVEN_HOME}" >> /etc/profile
echo "export PATH=$""MAVEN_HOME/bin:$""PATH" >> /etc/profile
source /etc/profile
#输出信息
echo "-----source update-----"
echo "java version"
java -version
echo "maven version"
mvn -v
echo "-----path-----"
echo "JAVA_HOME:"$JAVA_HOME
echo "MAVEN_HOME:"$MAVEN_HOME
source /etc/profile
source /etc/profile
source /etc/profile

#关闭iptables
systemctl disable firewalld

#关闭selinux
if [ `getenforce` == 'Enforcing' ];then
    setenforce 0
    sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
fi



cd /root
#下载jenkins
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.190.1/jenkins.war
#启动jenkins
nohup java -jar jenkins.war --ajp13Port=-1 --httpPort=8888 &
```
> 创建 jenkins.sh

``` bash
vi jenkins.sh
#填入上边的一键安装脚本
chmod 777 jenkins.sh
sh jenkins.sh
```

* ② 访问jenkins

* ③ 查看密码

``` bash
cat /root/.jenkins/secrets/initialAdminPassword

```