
#### （一）健康检查

* ④ 测试 livenessProbe的exec的方式


``` bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: docker.io/nginx
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```


``` bash
vi liveness.yaml
kubectl apply -f liveness.yaml
kubectl describe pod liveness-exec
```

* ④ 测试 livenessProbe的http的方式


``` bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
    app: httpd
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: docker.io/httpd
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /index.html
        port: 80
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 5
      periodSeconds: 5
```


``` bash
vi http.yaml
kubectl apply -f http.yaml
kubectl describe pod liveness-http
```

> 删除index文件，报404http服务

``` bash
kubectl exec -it liveness-http -- /bin/bash
cd htdocs/
rm index.html 
exit
kubectl  get pod
```


* ⑤ TCP Socket

``` bash
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: docker.io/googlecontainer/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
