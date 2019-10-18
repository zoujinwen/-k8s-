


#### （一）Dashboard


``` bash
vi /usr/lib/systemd/system/docker.service
#[Service] 这项下面添加

ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT
```

> 重启docker服务

``` bash
systemctl restart docker
```

* ① 官网

> [https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)



* ② 镜像 （集群中所有的节点需要进行这样的操作）

``` bash
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

sed -i '/targetPort:/a\ \ \ \ \ \ nodePort: 32500\n\ \ type: NodePort' kubernetes-dashboard.yaml
```



* ③ 安装（master节点）


``` bash
kubectl create -f kubernetes-dashboard.yaml
#删除kubectl delete -f kubernetes-dashboard.yaml
kubectl get pods -n kube-system
```



*  ④查看dashboard的详细信息

``` bash
kubectl get service --namespace=kube-system 
kubectl describe service kubernetes-dashboard --namespace=kube-system 
```


> 查看部署的Dashboard  在那个节点下
> 在node1的节点下，这样就知道要访问的地址了

``` bash
kubectl get pods --namespace=kube-system  -o wide
```

#### （二）token令牌认证登录

 > token认证的话，可以看到正常的信息，不会弹出上边的提示。

* ①创建serviceaccount

``` bash
kubectl create serviceaccount dashboard-admin -n kube-system
```


* ②把serviceaccount绑定在clusteradmin，授权serviceaccount用户具有整个集群的访问管理权限

``` bash
 kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
```


* ③获取serviceaccount的secret信息，可得到token（令牌）的信息

``` bash
kubectl get secret -n kube-system

#dashboard-admin-token-slfcr 通过上边命令获取到的
kubectl describe secret dashboard-admin-token-slfcr -n kube-system
```


``` bash
 kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/admin/{print $1}')   
```