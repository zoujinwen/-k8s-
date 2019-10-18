

#### （一）JOB 和 Cron Job 


#### （二）Job


* ① 代码演示

``` bash
---
apiVersion: batch/v1
kind: Job
metadata:
  name: demo-job
spec:
  template:
    metadata:
      name: demo-job
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command:
        - "bin/sh"
        - "-c"
        - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

> Job的RestartPolicy仅支持Never和OnFailure两种，执行完就结束，如果使用always任务执行完就重启，这样就不停的执行，不符合任务的特性。

``` bash
 cd ~
mkdir job
cd job/
vi job-demo.yaml
```

* ③ 代码运行，查看详情

``` bash
kubectl apply -f job-demo.yaml 

kubectl get jobs

kubectl describe jobs demo-job 
```


``` bash

kubectl get pods

kubectl logs demo-job-xs8z4

```



#### （三）CronJob
``` bash
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: demo-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: hello
            image: busybox
            args:
            - "bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

> 编写

``` bash
 vi demo-cronjob.yaml
```



* ⑤ 代码运行，查看详情

``` bash
kubectl apply -f demo-cronjob.yaml 

kubectl get cronjob

kubectl get job

kubectl get pods

kubectl get jobs
```
* ⑤ 删除定时job


``` bash
kubectl get cronjobs

kubectl delete cronjobs demo-cronjob

kubectl get cronjobs 

kubectl get jobs
```