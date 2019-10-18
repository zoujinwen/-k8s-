

#### （一）prometheus 介绍



#### （二）prometheus 准备安装



* ① prometheus 的镜像

> [https://hub.docker.com/r/prom/prometheus](https://hub.docker.com/r/prom/prometheus)



``` bash
timedatectl set-timezone Asia/Shanghai 
```



* ② 创建命名空间
``` bash
 kubectl create namespace kube-ops
```


* ③ 准备好nfs服务


* ③① 【Master节点配置】数据目录：/data/k8s/

``` bash
 mkdir -p /data/k8s/
```



* ③② 【Master节点配置】关闭防火墙

``` bash
systemctl stop firewalld.service

systemctl disable firewalld.service
```


* ③③ 【Master节点配置】安装配置 nfs

``` bash
 yum -y install nfs-utils rpcbind
```


* ③④【Master节点配置】 设置权限共享目录

``` bash
chmod 777 /data/k8s/
```




* ③⑤ 【Master节点配置】配置 nfs，nfs 的默认配置


``` bash
 vi /etc/exports
# /data/k8s  *(rw,sync,no_root_squash)
```



* ③⑥ 【Master节点配置】启用 rpc 注册


``` bash
systemctl start rpcbind.service

systemctl enable rpcbind

systemctl status rpcbind

```



* ③⑦ 【Master节点配置】启动 nfs 服务

> 看到 Started 证明启动成功了

``` bash
systemctl start nfs.service

systemctl enable nfs

systemctl status nfs

```


* ③⑧ 【Master节点配置】确定rpc 和 nfs 已经关联

``` bash
rpcinfo -p|grep nfs

```



* ③⑨ 【Master节点配置】具体文件挂载权限 

``` bash
cat /var/lib/nfs/etab
```


* ③⑩ 【node1节点配置】关闭防火墙

``` bash
systemctl stop firewalld.service

systemctl disable firewalld.service
```




* ③⑪【node1节点配置】安装nfs

``` bash
yum -y install nfs-utils rpcbind
```


* ③⑫【node1节点配置】先启动 rpc、然后启动 nfs

``` bash
systemctl start rpcbind.service 

systemctl enable rpcbind.service 

systemctl start nfs.service   

systemctl enable nfs.service
```


* ③⑬【node1节点配置】客户端来挂载下 nfs 测试下

``` bash
 showmount -e 192.168.86.100

```



* ③⑭【node1节点配置】新建目录，将 nfs 共享目录挂载到上面的目录

``` bash
mkdir -p /root/course/kubeadm/data

mount -t nfs 192.168.86.100:/data/k8s /root/course/kubeadm/data
```



* ③⑮【node1节点配置】输入文件查看

``` bash
echo "idig8.com">>/root/course/kubeadm/data/a.txt

cat /root/course/kubeadm/data/a.txt
```


* ③⑯【master节点配置】 nfs 服务端查看
``` bash
cat /data/k8s/a.txt 
```


> 上边说明NFS服务搭建完毕。


* ③⑰【master节点配置】 nfs 服务端查看
```
 vi /etc/exports 
rpcinfo -p localhost
```




> 之前都介绍过helm了，这里都不在介绍了直接开搞！

* ④ 准备存储卷

> 创建用下面的yaml

``` yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```




> 之前已经创建了
``` bash
kubectl apply -f scs.yaml 
kubectl  get storageclasses
```





#### （三）helm的方式安装 prometheus


* ① 通过helm 查看prometheus 

``` bash
helm search prometheus
```



* ② 安装命令


``` bash
helm install --namespace kube-ops --name prometheus stable/prometheus \
  --set alertmanager.persistentVolume.storageClass="managed-nfs-storage" \
  --set server.persistentVolume.storageClass="managed-nfs-storage"
```


* ③ 查看状态

``` bash
helm list

kubectl get all -n  kube-ops 
```


* ④ 暴露nodeport 访问UI

``` bash
 kubectl edit svc prometheus-prometheus-server -n  kube-ops 
```


``` bash
 kubectl get all -n  kube-ops 
```


* ④ helm删除prometheus也是很简单的

``` bash
helm list
helm delete prometheus  --purge
kubectl get all -n  kube-ops
```



（三）yaml方式安装 prometheus

* ① 创建目录 prometheus

``` bash
mkdir prometheus

cd prometheus/
```



* ② 用 ConfigMap 的形式进行管理

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
```


> 运行cm.yml文件

``` bash
kubectl create -f cm.yaml
```


* ③ 创建 prometheus.yml

``` bash
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```


> --config.file=/etc/prometheus/prometheus.yml  这个固定写法，这个不要修改

``` yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: prometheus
  namespace: kube-ops
  labels:
    app: prometheus
spec:
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: prometheus
      containers:
      - image: prom/prometheus:v2.12.0
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--storage.tsdb.retention=24h"
        - "--web.enable-admin-api" 
        - "--web.enable-lifecycle"
        ports:
        - containerPort: 9090
          protocol: TCP
          name: http
        volumeMounts:
        - mountPath: "/prometheus"
          subPath: prometheus
          name: data
        - mountPath: "/etc/prometheus"
          name: config-volume
        resources:
          requests:
            cpu: 100m
            memory: 512Mi
          limits:
            cpu: 100m
            memory: 512Mi
      securityContext:
        runAsUser: 0
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: prometheus
      - configMap:
          name: prometheus-config
        name: config-volume
```

* ⑤ 创建好这个 pvc 对象

``` yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prometheus
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.86.100
    path: /data/k8s

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus
  namespace: kube-ops
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```


> 创建

``` bash
kubectl create -f volume.yaml 
```


* ⑥ 需要配置 rbac 认证，需要在 prometheus 中去访问 Kubernetes 的相关信息

``` yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-ops
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-ops
```


> 获取权限

``` bash
kubectl create -f rbac.yaml
```



*  ⑦ 添加 promethues 的资源对象

``` bash
kubectl apply -f deploy.yaml

kubectl get pods -n kube-ops

kubectl logs -f prometheus-65d7dff4c7-774pn -n kube-ops
```


*  ⑧  Pod 创建成功后，为了能够在外部访问到 prometheus 的 webui 服务，还需要创建一个 Service 对象

``` yml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: kube-ops
  labels:
    app: prometheus
spec:
  selector:
    app: prometheus
  type: NodePort
  ports:
    - name: web
      port: 9090
      targetPort: http
```

> 创建，查看service的暴露端口

``` bash
kubectl create -f svc.yaml   

kubectl get svc -n kube-ops
```

