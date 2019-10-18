

#### （一）一个Pod



* ②编写yaml单个文件
> wordpress-pod.yaml

>  mysql这个容器我们做了一个数据卷的挂载，这是为了能够将mysql的数据能够持久化到节点上，为了保证数据不丢失，容器挂了，重启数据还是通过挂载点

``` bash
apiVersion: v1
kind: Pod
metadata:
  name: wordpress
  namespace: wdblog
spec:
  containers:
  - name: wordpress
    image: wordpress
    ports:
    - containerPort: 80
      name: wdport
    env:
    - name: WORDPRESS_DB_HOST
      value: db:3306
    - name: WORDPRESS_DB_USER
      value: wordpress
    - name: WORDPRESS_DB_PASSWORD
      value: wordpress
  - name: mysql
    image: mysql:5.7
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 3306
      name: dbport
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: rootPass123456
    - name: MYSQL_DATABASE
      value: wordpress
    - name: MYSQL_USER
      value: wordpress
    - name: MYSQL_PASSWORD
      value: wordpress
    volumeMounts:
    - name: db
      mountPath: /var/lib/mysql
  volumes:
  - name: db
    hostPath:
      path: /var/lib/mysql
```

* ③ 运行yaml文件

``` bash
kubectl apply -f wordpress-pod.yaml
```

* ④ 查看描述详情

``` bash
kubectl describe pod wordpress -n wdblog

kubectl get pod -n wdblog 
```



#### （二）两个Pod


* ① 编写wordpress-all.yaml

```  yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mysql-deploy
  namespace: wdblog
  labels:
    app: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec: 
      containers:
      - name: mysql
        image: mysql:5.7
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
          name: dbport
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: rootPass123
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          value: wordpress
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        hostPath:
          path: /var/lib/mysql

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: wdblog
spec:
  selector:
    app: mysql
  ports:
  - name: mysqlport
    protocol: TCP
    port: 3306
    targetPort: dbport

---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: wordpress-deploy
  namespace: wdblog
  labels:
    app: wordpress
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      initContainers:
      - name: init-db
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ['sh', '-c', 'until nslookup mysql;do echo waiting for mysql service; sleep 2; done;']
      containers:
      - name: wordpress
        image: wordpress
        imagePullPolicy: IfNotPresent
        ports:
        - name: wdport
          containerPort: 80
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 3
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 10
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql:3306
        - name: WORDPRESS_DB_USER
          value: wordpress
        - name: WORDPRESS_DB_PASSWORD
          value: wordpress

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wdblog
spec:
  selector:
    app: wordpress
  type: NodePort
  ports:
  - name: wordpressport
    protocol: TCP
    port: 80
    nodePort: 31306
    targetPort: wdport
```




* ② 运行上边的yaml 尝试访问

``` bash

kubectl apply -f wordpress-all.yaml

kubectl get pod -n wdblog 
```



* ③ 自动应对流量高峰期

``` bash
kubectl autoscale deployment wordpress-deploy --cpu-percent=10 --min=1 --max=10 -n wdblog

kubectl get pods -n wdblog

kubectl get hpa -n wdblog 
```
