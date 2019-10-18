

#### （一）普通应用监控



* ① 修改traefik.toml

> 添加metrics

``` bash
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
      entryPoint = "https"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      CertFile = "/ssl/tls.crt"
      KeyFile = "/ssl/tls.key"

[metrics]
  [metrics.prometheus]
    entryPoint = "traefik"
    buckets = [0.1, 0.3, 1.2, 5.0]
```


* ② traefik configmap及traefik pod



``` bash
kubectl get configmap -n kube-system

kubectl delete configmap traefik-conf -n kube-system

kubectl create configmap traefik-conf --from-file=traefik.toml -n kube-system

kubectl delete -f traefik.yaml 

kubectl apply -f traefik.yaml
```



* ③ 查看 svc的traefik信息，看看metrics信息

``` bash
kubectl get svc -n kube-system | grep traefik

curl 10.98.51.124:8080/metrics
```

* ④ 刚才是在traefik中加入了metrics，需要修改下prometheus

``` bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-ops
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      scrape_timeout: 15s
    scrape_configs:
    - job_name: 'prometheus'
      static_configs:
      - targets: ['localhost:9090']
    - job_name: 'traefik'
      static_configs:
        - targets: ['traefik-ingress-service.kube-system.svc.cluster.local:8080']
```



* ⑤ 重新加载服务，自动生效


``` bash
kubectl delete -f cm.yaml 

kubectl create -f prome-cm.yaml
```

``` bash
kubectl get svc -n kube-ops

#只为热更新
curl -X POST "http://10.100.181.180:9090/-/reload"

```


#### （二）使用 exporter 监控应用redis

>
* ① 官网

> [https://prometheus.io/docs/instrumenting/exporters/](https://prometheus.io/docs/instrumenting/exporters/)


* ② redis的exporter



* ③ redis的yaml


``` bash
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis
  namespace: kube-ops
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9121"
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:4
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
      - name: redis-exporter
        image: oliver006/redis_exporter:v1.1.1-alpine
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 9121
---
kind: Service
apiVersion: v1
metadata:
  name: redis
  namespace: kube-ops
spec:
  selector:
    app: redis
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
  - name: prom
    port: 9121
    targetPort: 9121
```


> redis-exporters 的镜像


* ④ redis的yaml运行查看信息

``` bash
kubectl apply -f prometheus-redis-exporter.yaml 

kubectl get pods -n kube-ops
```

> 查看pod的详情，里面两个pod

``` bash
kubectl describe pod redis-6ff7fdd999-r62l6 -n kube-ops  
```


* ⑤ 尝试访问下9121端口的metrics

``` bash
kubectl get svc -n kube-ops
#发现里面有数据，说明监控成功了。
curl 10.101.68.4:9121/metrics
```


* ⑥  redis-exporter 添加到加入了metrics，需要修改下prometheus

``` bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-ops
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      scrape_timeout: 15s
    scrape_configs:
    - job_name: 'prometheus'
      static_configs:
      - targets: ['localhost:9090']
    - job_name: 'traefik'
      static_configs:
        - targets: ['traefik-ingress-service.kube-system.svc.cluster.local:8080']
    - job_name: 'redis'
      static_configs:
        - targets: ['redis.kube-ops.svc.cluster.local:9121']
```



* ⑤ 重新加载服务，自动生效


``` bash
kubectl delete -f cm.yaml 

kubectl create -f cm.yaml
```


``` bash
kubectl get svc -n kube-ops

#只为热更新
curl -X POST "http://10.100.181.180:9090/-/reload"

```


* ② mysql-exporter 的yaml编写

``` yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: mysql
  namespace: kube-ops
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9104"
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.6
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
      - name: mysqld-exporter
        image: prom/mysqld-exporter:v0.12.1
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 9104
        env:
        - name: DATA_SOURCE_NAME
          value: root:root@(mysql:3306)
---
kind: Service
apiVersion: v1
metadata:
  name: mysql
  namespace: kube-ops
spec:
  selector:
    app: mysql
  ports:
  - name: mysql
    port: 3306
    targetPort: 3306
  - name: mysqld-exporter
    port: 9104
    targetPort: 9104
```


* ③  mysql的yaml运行查看信息

``` bash
kubectl apply -f prometheus-mysql-exporter.yaml 
kubectl get pods -n kube-ops
```


> 查看pod的详情，里面两个pod

``` bash
kubectl describe pod mysql-664458b75d-nhg68 -n kube-ops 
```



* ④  尝试访问下9104端口的metrics

``` bash
kubectl get svc -n kube-ops
#发现里面有数据，说明监控成功了。
curl 10.108.205.217:9104/metrics
```



* ⑤ mysql-exporter 添加到加入了metrics，需要修改下prometheus

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-ops
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      scrape_timeout: 15s
    scrape_configs:
    - job_name: 'prometheus'
      static_configs:
      - targets: ['localhost:9090']
    - job_name: 'traefik'
      static_configs:
        - targets: ['traefik-ingress-service.kube-system.svc.cluster.local:8080']
    - job_name: 'redis'
      static_configs:
        - targets: ['redis.kube-ops.svc.cluster.local:9121']
    - job_name: 'mysql'
      static_configs:
        - targets: ['mysql.kube-ops.svc.cluster.local:9104']
```


* ⑥ 重新加载服务，自动生效

``` bash
kubectl delete -f cm.yaml 

kubectl create -f cm.yaml
```



``` bash
kubectl get svc -n kube-ops

#只为热更新
curl -X POST "http://10.100.181.180:9090/-/reload"
```

