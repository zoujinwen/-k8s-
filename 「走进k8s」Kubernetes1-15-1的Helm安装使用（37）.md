


#### （一）Helm 介绍

* ① 官网


* ② 介绍


* ③ 组件




#### （二）Helm 安装
* ① 一键安装客户端是省时省力



``` bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh

chmod 700 get_helm.sh

 ./get_helm.sh
```


* ② 安装依赖的yum socat

``` bash
sudo yum install -y socat
```



* ③ 初始化helm


``` bash
 docker search tiller

# 一会要用这个jessestuart/tiller
```



* ④ 权限


``` yml

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```


> 运行权限赋予

``` bash
 kubectl create -f helm-rbac.yaml 
```





> 再次验证

``` bash
helm version
```



``` bash
kubectl get pod --all-namespaces

kubectl describe pod tiller-deploy-75f6c87b87-vdw2c -n kube-system 
``` 




``` bash
kubectl edit deploy tiller-deploy -n kube-system

# image: gcr.io/kubernetes-helm/tiller:v2.14.3   修改成图片上的jessestuart/tiller，上边docker search 的时候提到过。
```




* ⑤ kubernetes里面的pod信息

``` bash
kubectl get pod -n kube-system
```





``` bash
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

```


> 查看deploy的用户信息

``` bash
kubectl describe deploy --namespace kube-system tiller-deploy    
```



#### （三）Helm使用
* ① 创建

``` bash
mkdir helm

helm create hello-helm

tree
```



> 将默认的 stable 更改为 1.7.9， Service 的类型也改成 NodePort
``` bash
vi ./hello-helm/values.yaml 
```



* ④ 安装刚修改后的helm

``` bash
helm install ./hello-helm
```



> 查看helm的文件

``` bash
kubectl get pods -o wide

kubectl get svc -o wide
```



``` bash
helm package hello-helm
```



``` bash
helm delete intentional-ocelot

helm delete washed-panda 
```


``` bash
kubectl get pods 
```