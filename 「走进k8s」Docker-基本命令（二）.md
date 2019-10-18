
#### （一）镜像

``` bash
docker pull redis:4.0 
```



* ②镜像列表

``` bash
docker image ls
```


* ③镜像层次

``` bash
#刚才那个镜像ID
docker image 67f7ad418fdf
```

* ④删除镜像



``` bash
docker image rm  镜像名:版本号
```

``` bash
docker rmi  镜像ID
```


* ⑤镜像迁移


``` bash
docker save 镜像名称 | gzip > alpine-latest.tar.gz
```

``` bash
docker load -i alpine-latest.tar.gz
```


#### （二）容器
* ①创建容器

``` bash
docker run -it centos:7 /bin/bash
exit
```


``` bash
docker run -it -d centos:7 /bin/bash 
```



* ②容器列表


``` bash
docker container ls -all
```




* ③进入容器


``` bash
docker exec -it 容器ID /bin/bash
```

* ④终止和启动容器


``` bash
docker container stop 容器名称/容器ID
docker container start 容器名称/容器ID
```

* ⑤容器日志


``` bash
docker log -f 容器名称/容器ID
```


* ⑥删除容器

``` bash
docker container rm  容器ID
```

>批量删除容器，慎用

``` bash
docker rm -f $(docker ps -qa)
```

* ⑦容器改变

``` bash
docker diff  容器ID
```


* ⑧容器变成镜像

``` bash
docker commit --author 'liming' --message '修改打包成镜像' 镜像ID 名称:版本号
```




#### （三）总体信息查看


``` bash
docker system df
```
