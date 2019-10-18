
#### （一）初始化容器



 > 创建一个pod，里面通过命令判断myservice服务 和 mydb 服务是否存在检测，检测没有问题，就main-container的pod就可以启动了。

``` bash
apiVersion: v1
kind: Pod
metadata:
  name: main-pod1
  labels:
    app: init
spec:
  containers:
  - name: main-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

``` bash
vi init-pod.yaml
#添加上边的脚本
kubectl apply -y init-pod.yaml
```

``` bash
kubectl describe pod main-pod1
```


> 通过yaml的方式启动myservice服务 和 mydb 服务，然后查看main-pod1的状态

``` bash
vi service-svc.yaml
```


``` bash
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 6376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 6377
```



>  查看带初始化容器的pod的详情

``` bash
kubectl get pod
kubectl describe pod main-pod1
```



> 跟上边的最初的init 0:2已经完全不一样了。


* ⑥ 参数配置初始化



``` bash
vi init-config.yaml
```

``` bash
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://www.idig8.com
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  volumes:
  - name: workdir
    emptyDir: {}
```



> 编译 yaml

``` bash
kubectl apply -f init-config.yaml
```




> 进入容器查看是否是init下载的html

``` bash
kubectl exec -it init-demo -- /bin/bash
# 进入容器
cd /usr/share/nginx/html
cat html
```

* ⑦ 查看某一个init-container的日志

> kubectl logs POD名称 -c 依赖的init-container

``` bash
 kubectl logs main-pod1 -c init-myservice
```