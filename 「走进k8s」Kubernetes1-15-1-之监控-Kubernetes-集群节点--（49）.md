

#### （一）监控方案




#### （二）监控集群节点

>

* ② 部署 node-exporter

``` yml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-ops
  labels:
    name: node-exporter
spec:
  template:
    metadata:
      labels:
        name: node-exporter
    spec:
      hostPID: true
      hostIPC: true
      hostNetwork: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:v0.16.0
        ports:
        - containerPort: 9100
        resources:
          requests:
            cpu: 0.15
        securityContext:
          privileged: true
        args:
        - --path.procfs
        - /host/proc
        - --path.sysfs
        - /host/sys
        - --collector.filesystem.ignored-mount-points
        - '"^/(sys|proc|dev|host|etc)($|/)"'
        volumeMounts:
        - name: dev
          mountPath: /host/dev
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: rootfs
          mountPath: /rootfs
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Exists"
        effect: "NoSchedule"
      volumes:
        - name: proc
          hostPath:
            path: /proc
        - name: dev
          hostPath:
            path: /dev
        - name: sys
          hostPath:
            path: /sys
        - name: rootfs
          hostPath:
            path: /
```



* ③ 运行node-exporter

``` bash
kubectl create -f prometheus-node-exporter.yaml

kubectl get pods -n kube-ops

kubectl get svc -n kube-ops

kubectl get pods -n kube-ops -o wide
```


* ④ 获取metrics

``` bash
curl 127.0.0.1:9100/metrics
```

#### （三）prometheus监控集群节点

* ① 修改cm.yaml

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
    - job_name: 'mysql'
      static_configs:
        - targets: ['mysql.kube-ops.svc.cluster.local:9104']
    - job_name: 'node-export'
      static_configs:
        - targets: ['192.168.86.100:9100', '192.168.86.101:9100']
```

* ② 刷新configmap

``` bash
 kubectl delete -f cm.yaml 

kubectl create -f cm.yaml 
```


* ② 删除prometheus


``` bash
 kubectl delete -f svc.yaml 

kubectl create -f svc.yaml
 
```



* ③ 刷新prometheus

``` bash
kubectl get svc -n kube-ops

curl -X POST "http://10.96.245.208:9090/-/reload"
```


* ④ 请求url地址查看是否添加完毕
