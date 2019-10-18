

#### （一）Deployment

* ④ 演示yaml

``` bash
apiVersion: apps/v1beta2 # for kubectl versions >= 1.9.0 use apps/v1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: my-pod
        image: nginx
        ports:
        - containerPort: 3000
```

> 执行生成deploy.yaml

``` bash
kubectl apply -f deploy.yaml
kubectl get deployment
kubectl get rs
kubectl get pod
```


``` bash
kubectl get pods --show-labels
```


* ⑤ 增加滚动升级
``` bash
apiVersion: apps/v1beta2 # for kubectl versions >= 1.9.0 use apps/v1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: my-pod
        image: nginx
        ports:
        - containerPort: 3000
```
``` bash
kubectl apply -f deploy.yaml 
kubectl rollout status deployment hello-deployment
```

* ⑥ rollout命令

> 查看状态

``` bash
kubectl rollout status deployment <deployment>
```

> 暂停升级

``` bash
kubectl rollout pause deployment <deployment>
```

> 继续升级

``` bash
kubectl rollout resume deployment <deployment>
```

>查看rs的状态

``` bash
kubectl get rs
```




> 升级历史

``` bash
# kubectl rollout history deployment <deployment>

kubectl rollout history deployment hello-deployment
```


> 查看升级的全部信息

``` bash
kubectl describe deployment hello-deployment
```


> 回滚版本

``` bash
kubectl get rs
#  启动hello-deployment-5d5644bccf， 不启动hello-deployment-6678664459
kubectl rollout undo deployment hello-deployment
kubectl rollout status deployment  hello-deployment
kubectl get rs
#  启动hello-deployment-6678664459，不启动hello-deployment-5d5644bccf 
```





> 添加change-cause，命令行中添加 --record=true

``` bash
kubectl rollout history deployment  hello-deployment 
kubectl apply -f deloy.yaml --record=true
```



> rs 跟rollout 是对应的，如果rs删除了，rollout 也就看不到了

``` bash
kubectl get rs
kubectl delete rs hello-deployment-5d5644bccf 
kubectl rollout history deployment hello-deployment
kubectl get rs
```



> 指定版本，不在是回到上个版本 --to-revision

``` bash
kubectl rollout history deployment hello-deployment
kubectl rollout undo deployment hello-deployment --to-revision=4
kubectl rollout status deployment hello-deployment
kubectl get rs
kubectl rollout history deployment hello-deployment  
```
