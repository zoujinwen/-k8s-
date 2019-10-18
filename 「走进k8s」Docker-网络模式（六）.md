
#### （一）网络模式介绍

``` bash
docker network ls
```


#### （二）bridge模式（docker默认的网络模式）

``` bash
sudo yum install net-tools
ifconfig -a
```


* ②数据流程


``` bash
docker run --name a1 -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
docker run --name a2 -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
```

> 容器互相通信

``` bash
docker exec -it a1 /bin/sh 
ifconfig
#查看到a1的ip是172.17.0.2
exit
docker exec -it a2 /bin/sh 
ifconfig
#查看到a2的ip是172.17.0.3
#在a2容器内可以ping通172.17.0.2
ping 172.17.0.2

#在a1容器内尝试ping下a2的ip 172.17.0.3
#在a1容器内可以ping通172.17.0.2
ping 172.17.0.3
```


``` bash
docker run --name a2 --link a1 -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
docker exec -it a2 /bin/sh
ping a1
```


* ④自定义网络


``` bash
docker network create -d bridge net-test
```


> 测试网络通信，创建容器，进行通信

``` bash
docker run --name test3 --network net-test -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
docker run --name test4 --network net-test -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
docker exec -it test3 /bin/sh
ping test4
exit
docker exec -it test4 /bin/sh
ping test3
exit
```




``` bash

docker network inspect net-test
```

#### （三）host模式（共享主机的网络模式）


``` bash
#network 更换成host
 docker run --name test5_host --network host -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"  
docker exec -it test5_host  /bin/sh
ifconfig
```



#### （四）none模式（空网络模式）


``` bash
 docker run --name test7_none --network none -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"  
docker exec -it test7_none /bin/sh
ifconfig
```

#### （四）container 模式（容器之前的共享模式，学习k8s这个很重要）


``` bash
# test7_container 依赖a1的网络模式
 docker run --name test7_container --network container:a1 -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done" 
# 分别进入test7_container 和a1查看ifconfig 发现两个是一样的
docker exec -it test7_container /bin/sh
ifconfig
exit
docker exec -it a1 /bin/sh
ifconfig
exit
```
