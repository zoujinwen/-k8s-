

#### （一）ingress的介绍


#### （二）traefik的介绍

* ① 官网

> [https://traefik.io/](https://traefik.io/)



* ② 介绍



* ③ RBAC 安全认证方式(traefik-rbac.yaml)

``` yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```


> 运行traefik-rbac.yaml

``` bash
kubectl apply -f traefik-rbac.yaml 
```





* ③ 创建 Deployment 来管理 Pod，直接使用官方的 traefik 镜像部署，nodePort的方式访问

``` yaml
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      tolerations:
      - operator: "Exists"
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
        - name: admin
          containerPort: 8080
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
  type: NodePort
```

> traefik 还提供了一个 web ui 工具，就是上面的 8080 端口对应的服务，为了能够访问到该服务，将服务设置成的 NodePort。



> 创建traefik 

``` bash
kubectl apply -f traefik.yaml
```


> 查看创建信息

``` bash
kubectl get pods -n kube-system -o wide

kubectl get svc -n kube-system

```



* ④ 创建ingress ，ingress的方式访问traefik



``` yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik.idig8.com
    http:
      paths:
      - backend:
          serviceName: traefik-ingress-service
          servicePort: 8080
```






> 运行ingress.yaml

``` bash
kubectl apply -f ingress.yaml

kubectl get ingress -n kube-system 


kubectl get svc -n kube-syste

```


> 上面的访问 还是比较麻烦还需要记住端口，如果不想记住端口直接通过域名来访问的话需要修改 traefik.yaml

> 加入 hostPort: 80
``` yaml
ports:
- name: http
  containerPort: 80
  hostPort: 80

```

> 直接通过域名的方式访问：traefik.idig8.com

``` bash

kubectl apply -f traefik.yaml   
```
