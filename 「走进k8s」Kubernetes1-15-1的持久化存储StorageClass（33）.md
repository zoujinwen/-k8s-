

#### （一）介绍

> 修改nfs-clinet-deployment.yaml，里面的对应的参数替换成 nfs 配置，NFS_SERVER更换成PV那次的 192.168.86.100，NFS_PATH更换成/data/k8s

``` yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.168.86.100
            - name: NFS_PATH
              value: /data/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.86.100
            path: /data/k8s
```


>  修改nfs-clinet-rbac.yaml，创建serviceAccount绑定上对应的权限

``` yml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

> 创建 nfs-clinet-class.yaml，nfs-client 的 Deployment 声明完成后，创建一个StorageClass对象，provisioner对应的值一定要和上面的Deployment下面的 PROVISIONER_NAME 这个环境变量的值一样。

``` yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"

```

> 这步很重要，因为国内无法获取镜像quay.io/external_storage/nfs-client-provisioner:latest，这里通过阿里的获取，获取后更改image的名称的方式

``` bash
docker pull registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner

docker tag registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner:latest quay.io/external_storage/nfs-client-provisioner:latest
```



> 执行这3个yaml文件

``` yaml

#因为已经把这三个文件放入指定的目录下了，直接通过下面这个命令运行目录下的所有yaml

kubectl apply -f .
```



> 查看创建的结果

``` bash

 kubectl get pods
kubectl get deployment
kubectl get sc
# 上边的命令等于 kubectl get storageclass
```


#### （三）新建



* ① 新建立test-sc-pvc.yaml，这里的managed-nfs-storage 就是上边创建StorageClass名称。


``` bash

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```




* ② 运行查看效果

> 只创建了一个pvc，自动生成一个对应的pv，


``` bash
kubectl apply -f test-sc-pvc.yaml

kubectl get sc

kubectl get pvc

kubectl get pv
```




#### （四）测试
* ① 用 StorageClass 方式声明的 PVC 对象


``` yml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
    - "/bin/sh"
    args:
    - "-c"
    - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
    - name: nfs-pvc
      mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
  - name: nfs-pvc
    persistentVolumeClaim:
      claimName: test-pvc
```




> 运行yml，用一个 busybox 容器，在 /mnt 目录下面新建一个 SUCCESS 的文件，然后把 /mnt 目录挂载到上面我们新建的 test-pvc 这个资源对象上面了，查看 nfs 服务器的共享数据目录下面查看下数据

``` bash
 kubectl apply -f test-sc-pod.yaml

ls /data/k8s/

cat /data/k8s/default-test-pvc-pvc-0ef7327d-3be8-4f15-8229-0a3d85b6069d/SUCCESS
```

> 生成的规则：${namespace}-${pvcName}-${pvName}




*  ② StatefulSet  不在手动创建pvc的方式

``` yml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: nfs-web
spec:
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nfs-web
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.beta.kubernetes.io/storage-class: managed-nfs-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```




>  volumeClaimTemplates 下面就是一个 PVC 对象的模板，就类似于这里 StatefulSet 下面的 template，实际上就是一个 Pod 的模板，我们不单独创建成 PVC 对象，而用这种模板就可以动态的去创建了对象了

``` bash
 kubectl apply -f test-statefulset-sc.yaml 

kubectl get statefulset

 kubectl get pods 

```




> 查看3个对应的pvc

``` bash
kubectl get pvc
```



> 查看3个对应的pv

``` bash
kubectl get pv
```



> 查看对应的nfs

``` bash
ll /data/k8s/
```

