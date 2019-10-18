
#### （一）YAML 基础



* ① 官网


``` yml
---
friends:
  lastName: zhangsan
  age: 20
  
```

> 针对这个json格式或许你对上面的 JSON 文件更熟悉，但是不得不说json是不是更麻烦些

``` json
{
  "friends":: {
    "lastName": "zhangsan",
    "age":20
  }
}
```

* ②数组（List、Set）
> 说白了就是数组。
> 用- 值表示数组中的一个元素。
``` yml
pets:
 - pig
 - cat
 - dog
```
>对应的json格式

``` json
{
    "pets": ["pig","cat", "dog"]
}
```
* 数组List和Map的混合
> 来个混合的看看json和yml的对比
``` yml
person:
    lastName: liming
    age: 34
    boss: false
    birth: 2019/08/03
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 10
```
> 对应的json格式

``` json
{
    "person": {
        "lastName": "liming",
        "age": 34,
        "boss": false,
        "birth": "2019/08/03",
        "maps": {
            "k1": "v1",
            "k2": 12
        },
        "lists": [
            "lisi",
            "zhaoliu"
        ],
        "dog": {
            "name": "小狗",
            "age": 10
        }
    }
}
```


#### （三）Kubernetes 中 yaml的编写

* ② 尝试编写一个pod的yaml

``` yml
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



* ③ 执行yaml查看

``` bash
kubectl apply -f pod.yaml
kubectl describe first-pod

kubectl  get pods
```



* ④ 登录dashboard


``` bash
 kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/admin/{print $1}')   
```


* ④ 通过dashboard里面的yaml来生成pod
