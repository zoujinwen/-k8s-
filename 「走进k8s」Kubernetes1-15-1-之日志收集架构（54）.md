

#### （一）日志收集架构


* ② 创建容器打印日志

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  containers:
  - name: logger
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```


> 执行k8s-logger.yaml

``` bash
kubectl apply -f k8s-logger.yaml
kubectl get pod
```


> 查看日志信息

``` bash
kubectl logs logger
```


* ③ 信息收集的方式

#### （二）Node上部署一个日志收集程序

* ① 介绍





#### （三）Pod中附加专用日志收集的容器


* ② 操作

> 新建立yaml

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: logger-pod
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-1
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-2
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```



> 运行two-file-pod.yaml

``` bash
kubectl apply -f two-file-pod.yaml

kubectl get pod
```


> 查看日志

``` bash
kubectl logs logger-pod count-log-1

kubectl logs logger-pod count-log-2
```



#### （三）直接从应用程序收集日志

