
#### （一）Pod Hook
#### （二）代码演示

* ① 编写yaml


``` bash
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-post
spec:
  containers:
  - name: lifecycle-post-container-pod
    image: nginx

    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]
```

* ② 创建pod

``` bash
kubectl apply -f post.yaml
```


* ③ 查看pod

``` bash
kubectl get pods
```


* ④ 进入容器查看postStart的信息


``` bash
kubectl exec -it lifecycle-post -- /bin/bash
cat  /usr/share/message
```


* ⑤ 删除容器查看PreStop

> 新建立abc.yaml

``` bash
apiVersion: v1
kind: Pod
metadata:
  name: stop-demo2
  labels:
    app: hook
spec:
  containers:
  - name: stop-demo2
    image: nginx
    ports:
    - name: webport
      containerPort: 80
    volumeMounts:
    - name: message
      mountPath: /usr/share/
    lifecycle:
      preStop:
        exec:
          command: ['/bin/sh', '-c', 'echo Hello from the preStop Handler > /usr/share/message']
  volumes:
  - name: message
    hostPath:
      path: /tmp
```



``` bash
kubectl delete -f abc.yaml 

```


> 删除pod，然后在node1节点上查看是否输出

``` bash
#在node节点查看
 cat /tmp/message 
```