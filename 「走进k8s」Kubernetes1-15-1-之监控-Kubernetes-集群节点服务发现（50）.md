

#### （一）服务发现配置


* ① 修改configmap

``` bash
 cd prometheus/
vi cm.yaml
```

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
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:9100'
        target_label: __address__
        action: replace
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:10255'
        target_label: __address__
        action: replace
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
```


* ② 增加内容

``` yml
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:9100'
        target_label: __address__
        action: replace
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:10255'
        target_label: __address__
        action: replace
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
```







#### （二）运行configmap，执行查看自动发现


* ① 刷新configmap

``` bash
kubectl delete -f cm.yaml 

kubectl create -f cm.yaml
```


* ② 刷新地址

``` bash
kubectl get svc -n kube-ops

 curl -X POST "http://10.96.245.208:9090/-/reload"
```



* ③ 查看效果

