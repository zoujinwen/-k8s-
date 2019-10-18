

``` bash
 systemctl status kubelet
# 发现在这个文件夹下：/usr/lib/systemd/system/kubelet.service.d
cd /usr/lib/systemd/system/kubelet.service.d
ls
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 
cd /etc/kubernetes/manifests/ 
ls
```

``` bash
kubectl get pods -n kube-system
# 里面有几个都是在开机的时候自动跟这kubelet启动的。
```


> 因为直接放到manifests目录下，就不需要使用命令manifest-path=

``` bash
---
apiVersion: v1
kind: Pod
metadata:
  name: first-pod
  labels:
    app: bash
    tir: backend
spec:
  containers:
  - name: bash-container
    image: docker.io/busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```


* ② 查看静态pod


``` bash
kuectl get pods
# first-pod-k8s-master 就是静态pod
```




* ③ 删除静态pod

``` bash
kuectl delete pod first-pod-k8s-master
kuectl get pods
```


``` bash
 mv static-pod.yaml /tmp/
```