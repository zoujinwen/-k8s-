

#### （一）HPA


#### （二）Metrics-Server

* ① 官网


``` bash
mkdir metrics-server
cd metrics-server
vi aggregated-metrics-reader.yaml
vi auth-delegator.yaml
vi auth-reader.yaml
vi metrics-apiservice.yaml
vi metrics-server-deployment.yaml
vi metrics-server-service.yaml
vi resource-reader.yaml
```


> 修改metrics-server-deployment.yaml 文件

``` bash
vi metrics-server-deployment.yaml
```

``` yml
image: gcr.azk8s.cn/google_containers/metrics-server-amd64:v0.3.3
imagePullPolicy: IfNotPresent
command:
 - /metrics-server
 - --metric-resolution=30s
 - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
 - --kubelet-insecure-tls
```


``` bash
E0811 05:02:17.819517       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:k8s-master: unable to fetch metrics from Kubelet k8s-master (192.168.86.100): Get https://192.168.86.100:10250/stats/summary/: x509: cannot validate certificate for 192.168.86.100 because it doesn't contain any IP SANs, unable to fully scrape metrics from source kubelet_summary:k8s-node1: unable to fetch metrics from Kubelet k8s-node1 (192.168.86.101): Get https://192.168.86.101:10250/stats/summary/: x509: cannot validate certificate for 192.168.86.101 because it doesn't contain any IP SANs]
```

``` bash
 kubectl apply -f .
```


``` bash
kubectl -n kube-system get pods -l k8s-app=metrics-server

kubectl get svc -n kube-system  metrics-server
```


``` bash
kubectl top node
#出现error: metrics not available yet，等等
kubectl top nodes

kubectl top pods -n kube-system

kubectl cluster-info
```

> 一定要加  resources，cpu这部分，否则后面查看hpa的时候，初始化是unknow

``` bash
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hpa-nginx-deploy
  labels:
    app: nginx-demo
spec:
  revisionHistoryLimit: 15
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
         requests:
          cpu: 100m
        ports:
        - containerPort: 80
```

* ② 创建deployment

``` bash
kubectl apply -f auto-deployment.yaml
```




* ③ 创建自动扩所容的hpa


``` bash
kubectl autoscale deployment hpa-nginx-deploy --cpu-percent=5 --min=1 --max=5  
```

* ④ 查看上面命令行创建的HPA的YAML

``` bash
kubectl get hpa hpa-nginx-deploy -o yaml
```


* ⑤ 通过请求的方式增加cpu的使用，演示pod数量

> 目前的pod数量

``` bash
kubectl get pod
```

``` bash
kubectl run -i --tty load-generator --image=busybox /bin/sh


while true; do wget -q -O- http://10.244.1.27; done
```

``` bash
kubectl describe hpa hpa-nginx-deploy
```

``` bash
kubectl get pod
kubectl get hpa
```

``` bash
kubectl get pod

```
