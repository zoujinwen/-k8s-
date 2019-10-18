
#### （一）Harbor认识


* ① 介绍


* ② 含义


* ③ 认证的流程

* ④ Harbor和Registry的比较



#### （一）Harbor安装



``` bash
helm repo add harbor https://helm.goharbor.io

helm search harbor/harbor  
```


* ② 下载harbor的helm代码

``` bash
git clone https://github.com/goharbor/harbor-helm
```


``` bash
git branch -a
git checkout 1.2.0
git branch
git branch status
```

* ③ 新建values的覆盖值属性文件


``` yml
expose:
  type: ingress
  tls:
    enabled: true
  ingress:
    hosts:
      core: registry.idig8.com
      notary: notary.idig8.com
    annotations:
      kubernetes.io/ingress.class: "traefik"
      ingress.kubernetes.io/ssl-redirect: "true"
      ingress.kubernetes.io/proxy-body-size: "0"

externalURL: https://registry.idig8.com

persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      storageClass: "harbor-data"
    chartmuseum:
      storageClass: "harbor-data"
    jobservice:
      storageClass: "harbor-data"
    database:
      storageClass: "harbor-data"
    redis:
      storageClass: "harbor-data"
```



* ③ 创建 harbor-data 的 StorageClass 资源对象



``` bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: harbor-data
provisioner: fuseim.pri/ifs
```

``` bash
kubectl create -f harbor-data-sc.yaml
```



* ④ 在k8s里面通过helm安装harbor

``` bash
helm install --name harbor -f idig8-values.yaml . --namespace kube-ops
```


#### （二）登录

* ① 设置host文件



* ② 用户名和密码

> 用户名：admin
密码：Harbor12345


* ③ 不设置证书，防止登录docker login registry.idig8.com 提示没证书

 vi /usr/lib/systemd/system/docker.service

# 添加下面的内容
#ExecStart=/usr/bin/dockerd --insecure-registry registry.idig8.com
```
