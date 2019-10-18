

#### （一）场景


* ① 使用场景



* ② 上边场景分析



#### （二）Replication Controller

* ①介绍


* ②能干啥


* ③看一手文档




#### （二）使用RC来管理我们前面使用的Nginx的Pod

> rc 是replicationController的缩写

* ①创建yaml

``` bash
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-replication-controller
spec:
  replicas: 3
  selector:
    app: hello-pod-v1
  template:
    metadata:
      labels:
        app: hello-pod-v1
    spec:
      containers:
      - name: my-pod
        image: nginx
        ports:
        - containerPort: 3000
```



* ② 创建rc
``` bash
kubectl apply -f rc.yaml 
```


* ③ 查看rc

``` bash
kubectl get rc
kubectl get pod
kubectl describe rc my-replication-controller
```

* ④ 删除rc中的pod

``` bash
kubectl delete pod my-replication-controller-dhll5
# 自动生成了新的pod
```

* ⑤修改rc

``` bash
 kubectl edit rc my-replication-controller
#replicas: 3 改成 replicas: 4
```


* ⑥ rolling-update 滚动升级



``` bash
kubectl rolling-update my-replication-controller --image=nginx:1.13.7
```

``` bash
kubectl rolling-update my-replication-controller -f rc.yaml
```

