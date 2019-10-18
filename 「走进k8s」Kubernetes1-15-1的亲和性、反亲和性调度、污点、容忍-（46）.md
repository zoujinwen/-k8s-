
#### （一）概念亲和性和反亲和性


``` bash
kubectl get nodes --show-labels

kubectl label nodes k8s-master com=master

kubectl label nodes k8s-node1 com=node1
```



> 当 node 被打上了相关标签后，在调度的时候就可以使用这些标签了，只需要在 Pod 的spec字段中添加nodeSelector字段

``` bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: busybox-pod
  name: test-busybox
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: busybox
    imagePullPolicy: Always
    name: test-busybox
  nodeSelector:
    com: node1
```
> 执行查看分配节点

``` bash
kubectl apply -f node-selector-demo.yaml

kubectl describe pod test-busybox

kubectl get pods --label-columns=test-busybox
```

#### （二）亲和性和反亲和性



> pod 首先是要求不能运行在 master这个节点上，
有个节点满足com=node1的话就优先调度到这个节点上。
``` yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: affinity
  labels:
    app: affinity
spec:
  replicas: 1
  revisionHistoryLimit: 15
  template:
    metadata:
      labels:
        app: affinity
        role: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: nginxweb
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution: 
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: NotIn
                values:
                - master
          preferredDuringSchedulingIgnoredDuringExecution: 
          - weight: 1
            preference:
              matchExpressions:
              - key: com
                operator: In
                values:
                - node1
```


> 运行起来 

``` bash
kubectl apply -f node-affinity-demo.yaml 

kubectl get pods -l app=affinity -o wide
````

#### （三） podAffinity



> 使用硬策略，也就是说，如果不满足，app=busybox-pod 就不会生成deployment

``` yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: affinity
  labels:
    app: affinity
spec:
  replicas: 1
  revisionHistoryLimit: 15
  template:
    metadata:
      labels:
        app: affinity
        role: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: nginxweb
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - busybox-pod
            topologyKey: kubernetes.io/hostname
```


> 运行 

``` bash
kubectl delete -f node-affinity-demo.yaml
kubectl apply -f pod-affinity.yaml
kubectl get pod -l app=busybox-pod
kubectl get pod -l app=busybox-pod -o wide

kubectl get pods
````


#### （四）podAntiAffinity

> 设置污点：

``` bash
 kubectl taint node [node] key=value[effect]   
```     

> 去除污点：

``` bash
 kubectl taint nodes node_name key:[effect]-    #(这里的key不用指定value)
```                

>  去除指定key所有的effect: 
     kubectl taint nodes node_name key-
 示例：
     kubectl taint node node1 test:NoSchedule-
     kubectl taint node node1test:NoExecute-
     kubectl taint node node1test-










PS：目前k8s的安装是使用的kubeadmin，所以默认master节点是不安装pod的，这里污点和容忍都没有实际实例，大家只做了解。当实际使用的时候，在特殊说明，这里直说个概念。
