

#### （一）ConfigMap

> configMap.yaml

``` yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cm-demo
  namespace: default
data:
  data.1: hello
  data.2: world
  config: |
    property.1=value-1
    property.2=value-2
    property.3=value-3
```


``` yaml
kubectl create -f configMap.yaml

kubectl get configMap

kubectl describe configmap cm-demo
```

* ④ 配置ConfigMap的yaml文件，添加配置文件的方式


> 查看创建的命令实例

``` bash
kubectl create configmap -h
```



``` bash
mkdir testcm
cd testcm
vi mysql.conf
#ip=127.0.0.1
#port=3306

vi redis.conf
#ip=127.0.0.1
#port=6379
```


> 使用from-file关键字来创建包含这个目录下面所以配置文件的ConfigMap

``` bash
cd ~
kubectl create configmap cm-demo2 --from-file=testcm  
kubectl get configmap
kubectl describe configmap cm-demo2
```


* ⑤可以直接使用字符串进行创建，通过--from-literal参数传递配置信息，同样的，这个参数可以使用多次

``` bash
kubectl create configmap cm-demo3 --from-literal=ip=localhost --from-literal=port=33061 

kubectl get configmap

kubectl describe configmap cm-demo3
```

* ⑥ 使用configmap 查看所有的环境变量



``` yml
---
apiVersion: v1
kind: Pod
metadata:
  name: testcm1-pod
spec:
  containers:
    - name: testcm1
      image: busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: cm-demo3
              key: ip
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: cm-demo3
              key: ip
      envFrom:
        - configMapRef:
            name: cm-demo
```




> 运行上边的pod

``` bash
kubectl apply -f testcm1.yaml
```


> 执行完毕后，pod自动退出

``` bash
kubectl describe pod testcm1-pod

kubectl logs testcm1-pod  
```



* ⑦ 使用configmap 查看打印某个变量

``` yml
---
apiVersion: v1
kind: Pod
metadata:
  name: testcm2-pod
spec:
  containers:
    - name: testcm2
      image: busybox
      command: [ "/bin/sh", "-c", "echo $(DB_HOST)  $(DB_PORT) " ]
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: cm-demo3
              key: ip
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: cm-demo3
              key: port
      envFrom:
        - configMapRef:
            name: cm-demo
```



``` bash
kubectl apply -f testcm2.yaml

kubectl logs testcm2-pod  

kubectl describe pod testcm2-pod 
```



* ⑧ 使用ConfigMap的方式：通过数据卷使用，在数据卷里面使用ConfigMap

> 创建pod文件

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: testcm3-pod
spec:
  containers:
    - name: testcm3
      image: busybox
      command: [ "/bin/sh", "-c", "cat /etc/config/redis.conf" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: cm-demo2
```


>  运行pod，查看log

``` bash
kubectl apply -f testcm3-pod.yaml 

kubectl logs testcm3-pod  


```



* ⑨ 查看configmap的内部详情

``` bash
kubectl get configmap cm-demo2 -o  yaml 
```
