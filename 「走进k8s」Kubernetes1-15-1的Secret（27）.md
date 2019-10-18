
#### （一） Secret 总体介绍



#### （二）Opaque 详解

* ② 演示
> 比如我们来创建一个用户名为 admin，密码为 admin123 的 Secret 对象，把这用户名和密码做 base64 编码。

``` bash
echo -n "admin" | base64
# YWRtaW4=

 echo -n "admin123" | base64   
# YWRtaW4xMjM=
```

> 需要记住上班的 2个对应的 base64 编码，下面要用。创建一个yaml的Secret。

``` yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: YWRtaW4xMjM=
```



``` yml
kubectl apply -f mysecret.yaml 

kubectl get secret

kubectl describe secret mysecret

 kubectl get secret mysecret -o yaml
```

* ③ secret使用方式


* ④ 环境变量

> 创建一个简单的buybox的pod，secretKeyRef关键字，上次的configMapKeyRef比较类似

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: secret1-pod
spec:
  containers:
  - name: secret1
    image: busybox
    command: [ "/bin/sh", "-c", "echo $(USERNAME) $(PASSWORD)" ]
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```




``` bash
kubectl apply -f my-pod-secret.yaml

kubectl get pod

kubectl logs secret1-pod 
```



> 可以看到有 USERNAME 和 PASSWORD 两个环境变量输出出来。admin admin123


* ⑤ Volume 挂载

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: secret2
    image: busybox
    command: ["/bin/sh", "-c", "ls /etc/secrets"]
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
  volumes:
  - name: secrets
    secret:
     secretName: mysecret
```



``` bash
kubectl apply -f my-volume-secret.yaml

kubectl get pod

kubectl logs secret1-pod 
```


#### （二）kubernetes.io/dockerconfigjson


* ② 通过命令的方式创建

``` java
kubectl create secret docker-registry myregistry --docker-server=DOCKER_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

* ③ 查看秘钥信息

``` bash
kubectl get secret
kubectl describe secret myregistry
kubectl get secret myregistry -o yaml

echo eyJhdXRocyI6eyJET0NLRVJfU0VSVkVSIjp7InVzZXJuYW1lIjoiRE9DS0VSX1VTRVIiLCJwYXNzd29yZCI6IkRPQ0tFUl9QQVNTV09SRCIsImVtYWlsIjoiRE9DS0VSX0VNQUlMIiwiYXV0aCI6IlJFOURTMFZTWDFWVFJWSTZSRTlEUzBWU1gxQkJVMU5YVDFKRSJ9fX0= | base64 -d
```



* ④ 从仓库中拉取，并使用仓库秘钥


``` yml
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
  - name: foo
    image: 192.168.1.200:5000/test:v1
  imagePullSecrets:
  - name: myregistrykey

```


#### （三）kubernetes.io/service-account-token


``` bash
kubectl create serviceaccount idig8
kubectl get secret
````


* ③ 在pod中使用service account

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: my-sa-demo
  namespace: default
  labels:
    name: myapp
    tier: appfront
spec:
  containers:
  - name: myapp
    image: nginx
    ports:
    - name: http
      containerPort: 80
  serviceAccountName: idig8
```


``` bash
kubectl apply -f pom-serviceaccount.yaml 

kubectl describe pod my-sa-demo
```
