
#### （一）DaemonSet 

* ⑤ DaemonSet创建yaml文件

``` bash
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: nginx-ds
  labels:
    k8s-app: nginx
spec:
  template:
    metadata:
      labels:
        k8s-app: nginx
    spec:
      containers:
      - image: nginx:1.7.8
        name: nginx
        ports:
        - name: http
          containerPort: 80
```

![](https://upload-images.jianshu.io/upload_images/11223715-8e09d527120336b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* ⑥ 运行yaml文件

``` bash
kubectl apply -f daemonset-demo.yaml 

kubectl get daemonset

kubectl get pods -l k8s-app=nginx 

kubectl get pods -l k8s-app=nginx -o wide
```


#### （二）DaemonSet 

* ③ 创建StatefulSet

``` bash

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    release: stable
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: /tmp/data
```
![](https://upload-images.jianshu.io/upload_images/11223715-edc60b15c98d10fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


``` bash

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv002
  labels:
    release: stable
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: /tmp/data
```


``` bash
kubectl create -f pv001.yaml && kubectl create -f pv002.yaml

kubectl get pv
```


> statefulset-demo.yaml

``` bash
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
    role: stateful

---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
        role: stateful
    spec:
      containers:
      - name: nginx
        image: cnych/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 2Gi
````



``` bash
kubectl create -f statefulset-demo.yaml

kubectl get pods -w -l role=stateful

```




* ④ 查看运行的状态


> 查看service的信息

``` bash
kubectl get svc

```

> pv 和 pvc 信息绑定

``` bash
kubectl get pvc

kubectl get pv
```