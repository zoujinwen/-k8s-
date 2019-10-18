
#### （一）官方镜像仓库


``` bash
docker login
``` 


``` bash
docker search redis --filter=stars=50
```


``` bash
docker logout
```


``` bash
docker images
docker tag mysql:latest zhugeaming/mysql-test
docker login
docker push zhugeaming/mysql-test
```




#### （二）私有镜像仓库docker-registry



``` bash
docker run -d -p 5000:5000  --name registry registry:2.7.1
```


``` bash
/etc/docker/daemon.json
```

> 修改内容，192.168.66.100 是仓库的IP地址

``` bash
{"registry-mirrors": 
   ["http://b81aace9.m.daocloud.io"],
 "insecure-registries": [
    "192.168.66.100:5000"
  ]
}
```

> 修改完毕后

``` bash
service docker restart 
#重启容器registry 
docker container start registry 
```

* ③推送镜像

``` bash
docker pull hello-world
docker tag hello-world:latest 192.168.66.100:5000/hello-world:latest
docker push  192.168.66.100:5000/hello-world:latest
```


* ④查看仓库内的镜像



``` bash
docker images
docker rmi 192.168.66.100:5000/hello-world
docker rmi hello-world
docker pull 192.168.66.100:5000/hello-world
docker images
```
