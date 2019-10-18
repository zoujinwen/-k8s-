
#### （一）service



#### （三）service



> nginx-deploy.yaml

``` yml
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx-demo
spec:
  replicas: 3
  revisionHistoryLimit: 15
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: nginxweb
```


> demo-service.yaml 

``` yml
apiVersion: v1
kind: Service
metadata:
  name: demoservice
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: nginxweb

```
*  ② 执行yaml

``` bash
vi  nginx-deploy.yaml

vi demo-service.yaml
```




``` bash
kubectl apply -f nginx-deploy.yaml
kubectl apply -f demo-service.yaml
```

mageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 查看service情况。Endpoints就是service会持续的监听这个selector。

``` bash
kubectl get svc

kubectl describe svc demoservice
```


*  ③ 进入一个pod尝试访问service

>  上边service的ip ：10.107.50.167

``` bash
kubectl run -it testservice --image=busybox /bin/sh
wget -q -O- 10.107.50.167
```