

#### （一）grafana-kubernetes-app插件



* ① grafana-kubernetes-app 获取方式


* ②  grafana-kubernetes-app 安装方式


> 找到对应的grafana的pod

``` bash
kubectl get pod -n kube-ops
```


> 进入对应grafana的pod

``` bash
kubectl exec -it grafana-69dc5bb7cc-65qp2 /bin/bash -n kube-ops
```

``` bash
grafana-cli plugins install grafana-kubernetes-app
```


* ③  grafana-kubernetes-app 重启grafana

> 退出pod，删除pod，因为之前是用过deployment自动新建

``` bash
kubectl get pod -n kube-ops
kubectl delete pod grafana-69dc5bb7cc-65qp2 -n kube-ops
kubectl get pod -n kube-ops
```




* ④ 查看grafana-kubernetes-app 


> 需要3个证书，证书查看

``` bash
cat ~/.kube/config
```


* ⑤ 点击下是否可用


#### （二）预警功能

* ① email报警功能

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-config
  namespace: kube-ops
data:
  grafana.ini: |
    [server]
    root_url = http://<你grafana的url地址>
    [smtp]
    enabled = true
    host = smtp.163.com:25
    user = idig8@163.com
    password = <邮箱密码>
    skip_verify = true
    from_address = idig8@163.com
    [alerting]
    enabled = true
    execute_alerts = true
```

> 修改grafana-deploy.yaml文件

``` yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: grafana
  namespace: kube-ops
  labels:
    app: grafana
spec:
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:6.3.5
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          name: grafana
        env:
        - name: GF_SECURITY_ADMIN_USER
          value: admin
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: admin123
        readinessProbe:
          failureThreshold: 10
          httpGet:
            path: /api/health
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /api/health
            port: 3000
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 100m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 256Mi
        volumeMounts:
        - mountPath: /var/lib/grafana
          subPath: grafana
          name: storage
        - mountPath: "/etc/grafana"
          name: config
      securityContext:
        fsGroup: 472
        runAsUser: 472
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: grafana
      - name: config
        configMap:
          name: grafana-config
```


> 创建configmap和更新deploy

``` bash
kubectl create -f grafana-cm.yaml

kubectl apply -f grafana-deploy.yaml
```

