

####  （一）认识PV/PVC/StorageClass


####  （二）NFS安装

>
* ① 【Master节点配置】数据目录：/data/k8s/

``` bash
 mkdir -p /data/k8s/
```


* ② 【Master节点配置】关闭防火墙

``` bash
systemctl stop firewalld.service

systemctl disable firewalld.service
```


* ③ 【Master节点配置】安装配置 nfs

``` bash
 yum -y install nfs-utils rpcbind
```


* ④【Master节点配置】 设置权限共享目录

``` bash
chmod 777 /data/k8s/
```



* ⑤ 【Master节点配置】配置 nfs，nfs 的默认配置

> 文件在 /etc/exports 文件下，在该文件中添加下面的配置信息

``` bash
 vi /etc/exports
# /data/k8s  *(rw,sync,no_root_squash)
```


*⑥ 【Master节点配置】启用 rpc 注册

> 看到 Started 证明启动成功了

``` bash
systemctl start rpcbind.service

systemctl enable rpcbind

systemctl status rpcbind

```


* ⑦ 【Master节点配置】启动 nfs 服务

> 看到 Started 证明启动成功了

``` bash
systemctl start nfs.service

systemctl enable nfs

systemctl status nfs

```


* ⑧ 【Master节点配置】确定rpc 和 nfs 已经关联

``` bash
rpcinfo -p|grep nfs

```



* ⑨ 【Master节点配置】具体文件挂载权限 

``` bash
cat /var/lib/nfs/etab
```




* ⑩ 【node1节点配置】关闭防火墙

``` bash
systemctl stop firewalld.service

systemctl disable firewalld.service
```


* ⑪【node1节点配置】安装nfs

``` bash
yum -y install nfs-utils rpcbind
```



* ⑫【node1节点配置】先启动 rpc、然后启动 nfs

``` bash
systemctl start rpcbind.service 

systemctl enable rpcbind.service 

systemctl start nfs.service   

systemctl enable nfs.service
```



* ⑬【node1节点配置】客户端来挂载下 nfs 测试下

``` bash
 showmount -e 192.168.86.100

```


* ⑭【node1节点配置】新建目录，将 nfs 共享目录挂载到上面的目录

``` bash
mkdir -p /root/course/kubeadm/data

mount -t nfs 192.168.86.100:/data/k8s /root/course/kubeadm/data
```



* ⑮【node1节点配置】输入文件查看

``` bash
echo "idig8.com">>/root/course/kubeadm/data/a.txt

cat /root/course/kubeadm/data/a.txt
```

* ⑯【master节点配置】 nfs 服务端查看
``` bash
cat /data/k8s/a.txt 
```



> 上边说明NFS服务搭建完毕。



#### （二）PV



* ③创建yaml文件

``` yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv1
spec:
  capacity: 
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /data/k8s
    server: 192.168.86.100
```


* ④ 运行yaml文件

``` bash
kubectl apply -f pv-demo.yaml
```
