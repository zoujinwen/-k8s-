
#### （一）apiserver的方式进行服务发现

#### （二）环境变量



* ① 创建test-nginx.yaml

``` yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    k8s-app: nginx-demo
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.8
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    name: nginx-service
spec:
  ports:
  - port: 5000
    targetPort: 80
  selector:
    app: nginx
```



* ② 创建服务

``` bash
kubectl apply -f test-nginx.yaml 

kubectl get deployment

kubectl get pods 

 kubectl get svc
```


> service 里面关联了3个pod


``` bash
 kubectl describe svc nginx-service 
```

* ③ 查看上边的服务信息，创建一个服务查看，所有的环境变量，顺便查看下nginx-server

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: test-api
spec:
  containers:
  - name: test-api
    image: busybox
    command: ["/bin/sh", "-c", "env"]
```


> 执行pod的yml，查看日志

``` bash
kubectl apply -f test-pod.yaml
```


``` yml
apiVersion: v1
kind: Pod
metadata:
  name: test-api
spec:
  initContainers:
    - name: init-nginx-server
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ['sh', '-c', 'until nslookup nginx-service;do echo waiting for nginx-service sleep 2; done;']
  containers:
  - name: test-api
    image: busybox
    command: ["/bin/sh", "-c", "env"]
```


``` bash
kubectl delete pod  test-api
kubectl apply -f test-pod.yaml
kubectl get pods

kubectl logs test-api 
```


#### （三）kubedns


``` bash
kubectl get pods -n kube-system
```

``` bash
 kubectl run -it --image=busybox:1.28.4 --rm --restart=Never sh

# cat /etc/resolv.conf 
```


``` bash
wget -q -O- nginx-service.default.svc.cluster.local:5000


wget -q -O- nginx-service.default:5000

wget -q -O- nginx-service.default.svc:5000
```