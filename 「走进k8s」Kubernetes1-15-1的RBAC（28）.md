

#### （一）RBAC




``` bash
- --authorization-mode=Node,RBAC
```

> 查看证书信息

``` bash
ls
# idig8,crt，idig8.csr ，idig8.key
``` 


> 证书文件和私钥文件在k8s集群中创建新的凭证和上下文(Context)

``` bash
 kubectl config set-credentials idig8 --client-certificate=idig8.crt  --client-key=idig8.key
```


> 用户idig8创建了，然后为这个用户设置新的 Context

``` bash
kubectl config set-context idig8-context --cluster=kubernetes --namespace=kube-system --user=idig8
```


* ② 创建角色



``` yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: idig8-role
  namespace: kube-system
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

> 执行创建

``` bash
kubectl apply -f idig8-role.yaml
```



> 查看信息

``` bash
kubectl get role -n kube-system
```




* ③ 角色权限绑定


``` yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: idig8-rolebinding
  namespace: kube-system
subjects:
- kind: User
  name: idig8
  apiGroup: ""
roleRef:
  kind: Role
  name: idig8-role
  apiGroup: ""
```



``` bash
kubectil apply -f idig8-rolebinding.yaml 
```



* ④ 测试下 idig8-context上下文来操作



``` bash
kubectl get pods --context=idig8-context

kubectl get pods -n kube-system 
```




> 如果查看-n=default 就会报错

``` bash
 kubectl get pods -n default
```




#### （三）创建访问某个 namespace 的用户



* ① 创建 ServiceAcount 对象

``` bash 
kubectl create sa idig8-sa -n kube-system
```


``` bash
kubectl create -f idig8-sa-role.yaml
```



* ③ 新建一个角色和用户的绑定



``` yml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: idig8-sa-rolebinding
  namespace: kube-system
subjects:
- kind: ServiceAccount
  name: idig8-sa
  namespace: kube-system
roleRef:
  kind: Role
  name: idig8-sa-role
  apiGroup: rbac.authorization.k8s.io
```



``` bash
kubectl create -f idig8-sa-rolebinding.yaml
````

``` bash
kubectl get secret -n kube-system |grep idig8-sa

# 查看上边对应的secret 打印出来
kubectl get secret idig8-sa-token-nxgqx -o jsonpath={.data.token} -n kube-system |base64 -d
```




* ⑤ 使用dashboard 查看



#### （四）创建访问所有 namespace 的用户



* ① 创建 ServiceAcount 对象

``` bash 
kubectl create sa idig8-sa2 -n kube-system
```


* ② 然后创建一个 ClusterRoleBinding 对象


``` yml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: idig8-sa2-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: idig8-sa2
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```


``` bash
kubectl create -f idig8-sa-rolebinding.yaml
````



* ③ 查看关联的token


``` bash
kubectl get secret -n kube-system |grep idig8-sa2

# 查看上边对应的secret 打印出来
kubectl get secret idig8-sa-token-nxgqx -o jsonpath={.data.token} -n kube-system |base64 -d
```



* ④ 使用dashboard 查看
