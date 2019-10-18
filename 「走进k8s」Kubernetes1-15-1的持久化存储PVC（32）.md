

#### （一）新建PVC


* ① 新建pvc

``` yml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```




``` bash
kubectl get pv

kubectl create -f pvc-nfs.yaml

kubectl get pv
```




>  如果没有对应的pv的话，新建pvc的话会是怎么样的

``` yml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc2-nfs
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  selector:
    matchLabels:
      app: nfs
```

> 上面新建的 PVC 是没办法选择到合适的 PV 的

``` bash
kubectl apply -f pvc2-nfs.yaml 

kubectl get pvc
```




> 新建立一个PV，查看能否自动关联

``` bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2-nfs
  labels:
    app: nfs
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.86.100
    path: /data/k8s
```




> 运行查看pv和pvc的变化。
> 很快就发现该 PV 是 Bound 状态了，对应的 PVC 是 default/pvc2-nfs，证明上面的 pvc2-nfs 终于找到合适的 PV 进行绑定上了。

``` bash
kubectl apply -f pv2-nfs.yaml  

kubectl get pvc

kubectl get pv
```





>  分析一种情况，如果pv是2个g，pvc是4个g，他们会绑定吗？答案是他们不会被绑定的，因为pv满足不了pvc需求的4个g。如果pv是4个g，pvc是2个g，他们就会绑定，因为能满足他的大小。


#### （二）使用 PVC
>  nginx 的镜像来测试下

* ① 创建deployment

``` yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nfs-pvc
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nfs-pvc
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.8
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
      volumes:
      - name: www
        persistentVolumeClaim:
          claimName: pvc2-nfs

---

apiVersion: v1
kind: Service
metadata:
  name: nfs-pvc
  labels:
    app: nfs-pvc
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: web
  selector:
    app: nfs-pvc
```



* ② 运行deployment

``` bash

kubectl apply -f nfs-pvc-deploy.yaml
```





``` bash

kubectl get deployment

```


``` bash
kubectl get pods

kubectl get svc
```



* ③ 测试访问

``` bash
cd /data/k8s

echo "Hello.kubernetes~">>index.html

```



* ④ subPath共享分支目录


``` bash
vi nfs-pvc-deploy.yaml

# subPath: nginxpvc-test
```




``` bash
kubectl apply -f nfs-pvc-deploy.yaml
```

> 更新新的yaml

``` yml
kubectl apply -f ~/nfs-pvc-deploy.yaml
```



> 将index.html 迁移到分支目录下

``` bash
cd /data/k8s
mv index.html ./nginxpvcTest/
```

> 访问nginx 发现正常




* ⑤ 删除deployment ,看看共享目录下的文件是否存在

``` bash

kubectl get deployment

kubectl delete deployment nfs-pvc


kubectl get deployment

ll /data/k8s/nginxpvcTest/

# index.html还存在

```




> 重新载入 yaml 查看是否自动加载发现可以正常访问nginx

``` bash
kubectl apply -f ~/nfs-pvc-deploy.yaml
```

