

#### （一）Dockerfile


* ①编写Dockerfile

``` bash
mkdir mynginx
cd mynginx
vi Dockerfile
```

> 编辑内容

``` bash
FROM nginx
RUN echo '<h1>Hello,World,Dockerfile</h1>' > /usr/share/nginx/html/index.html
```

* ②构建镜像


``` bash
docker build -t nginx:v0 .
```


* ③该镜像历史



``` bash
docker history nginx:v0
```

#### （二）Dockerfile命令合集

``` bash
FROM <image>
FROM <image>:<tag>
FROM <image>@<digest>
```

``` bash
FROM scratch #制作base Image
FROM centos #使用base Image
FROM centos:7.9
FROM mysql:5.6
```

* ②LABEL


``` bash
LABEL maintainer="394498036@qq.com"
LABEL version="1.0"
LABEL description="This is description \
欢迎关注：编程坑太多"
```

* ③RUN

``` bash
#不建议使用
RUN yum update
RUN yum install -y vim
RUN python-dev

#建议使用
RUN yum update && yum install -y vim \
          python-dev #反斜线换行
```

``` bash
RUN  apt-get update && apt-get install -y perl \
          pwgen --no-install-recommends && rm -rf \
          /var/lib/apt/lists/*   #注意清理cache
```

* ④ENV


``` bash
ENV MYSQL_VERSION 5.6
E-NV apt-get install -y mysql-server = "${MYSQL_VERSION}" \
&& rm -rf /var/lib/apt/lists/* #引用常亮
```

* ⑤COPY

``` bash
COPY ["", ""]
COPY nginx.conf /etc/nginx/nginx.conf
```


* ⑥WORKDIR



``` bash
WORKDIR /test #如果没有会自动创建test目录
WORKDIR idig8
RUN pwd          #输出结果应该是/test/idig8
```


