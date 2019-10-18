
#### （一）调度器介绍


* ① 介绍



* ② 为什么要了解调度器


* ③ 调度流程图

* ④ 调度过程中需要考虑的问题


* ⑤ 了解下关于scheduler的源码

* ⑥ 调度流程


#### （二）自定义调度

* ① 介绍


* ② 官方实例

``` json
{
  "kind" : "Policy",
  "apiVersion" : "v1",
  "predicates" : [
      {"name" : "PodFitsHostPorts"},
      {"name" : "PodFitsResources"},
      {"name" : "NoDiskConflict"},
      {"name" : "NoVolumeZoneConflict"},
      {"name" : "MatchNodeSelector"},
      {"name" : "HostName"}
  ],
  "priorities" : [
      {"name" : "LeastRequestedPriority", "weight" : 1},
      {"name" : "BalancedResourceAllocation", "weight" : 1},
      {"name" : "ServiceSpreadingPriority", "weight" : 1},
      {"name" : "EqualPriority", "weight" : 1}
  ]
}
```

#### （三）多调度器



>   选择使用自定义调度器 my-scheduler

``` yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  schedulerName: my-scheduler
  containers:
  - name: nginx
    image: nginx:1.15.1
```


#### （四）优先级调度


* ③ 实例

``` yml
apiVersion: v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."

```