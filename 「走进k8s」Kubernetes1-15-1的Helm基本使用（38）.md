

#### （一）仓库

* ① 查看仓库地址

``` bash
helm repo list
```

* ② 修改仓库地址

``` bash
helm repo remove stable
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo update
``` 


#### （二）查找 chart


* ① 查找命令
``` bash
helm search
```

* ② 缩小查询返回

``` bash
helm search mysql
```


* ③ 查看chart的描述



``` bash
helm inspect stable/mariadb
```


* ④ 安装 chart


``` bash
helm install stable/mariadb
```


``` bash
helm list
```

``` bash
helm status modest-chinchilla
```


* ⑤ 自定义 chart

>  查看 chart 上可配置的选项

``` bash
helm inspect values stable/mariadb
```


``` yml
mariadbUser: idig8
mariadbDatabase: idig8DB
service:
  type: NodePort
```



``` bash
 helm install -f helm-mariadb-config.yaml stable/mariadb --name mydb
```


> 查看安装结果

``` bash
kubectl get svc

kubectl get pods
```



``` bash
kubectl describe pods mydb-mariadb-7cc64cdccd-kgk7f 
```

``` bash
kubectl get pvc
```


#### （三）升级

* ① 数据持久化禁用掉

``` bash
mariadbUser: idig8
mariadbDatabase: idig8DB
service:
  type: NodePort
persistence:
  enabled: false
```


* ② 更新config文件

``` bash
helm upgrade -f config.yaml mydb stable/mysql
```


* ③ 查看pod信息

``` bash
kubectl get pods
```


* ④ 查看helm列表


``` bash
helm list
```


* ⑤ 查看历史版本

``` bash
 helm history mydb
```


* ⑥ 既然有历史版本了，肯定是可以回滚的


``` bash

helm rollback mydb 1

helm history mydb

helm rollback mydb 2
```


* ⑥ 每次配置里面都能生成对应configmap配置



``` bash
kubectl get configmap
```


#### （四）删除



``` bash
helm delete modest-chinchilla 
```



* ② 查看未彻底删除的

``` bash

helm list

helm delete modest-chinchilla 

helm list --deleted

helm list

```



* ③ 彻底删除，加入--purge

``` bash
helm delete modest-chinchilla --purge
```