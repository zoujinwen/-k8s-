
#### （一）定义 chart



#### （二）创建模板

* ① 删除原来的模板

``` bash
 rm -rf mychart/templates/*.*
```




``` bash
 rm -rf mychart/templates/tests/
```



* ② 自定义模板

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "idig8.com"
```



* ③ 安装模板

``` bash
cd ~/chart/
helm install ./mychart/
```


* ④ 看到渲染过后资源文件

``` bash
helm ls
helm get manifest historical-ragdoll 
```



``` bash
kubectl get configmap
```

#### （三）添加一个简单的模板

* ① 创建动态模板


``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
```


* ② 运行配置，查看渲染后的文件

``` bash
helm install ./mychart

helm ls

helm get manifest kissable-wasp
```



#### （四）调试



* ① 调试命令

``` bash
 helm install --dry-run --debug ./mychart
```



* ② 查看命令是否真实安装


``` bash
helm ls
```




#### （六）values 文件


* ① Values的来源


* ② 修改values.yaml文件

> 清空values.yaml文件内容，添加下面的内容

``` yaml
address: idig8.com
```


* ③ configmap.yaml  中添加
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ .Values.address }}
```


* ④ 用 debug 模式来查看模板渲染结果

``` bash
helm install --dry-run --debug ./mychart
```


* ⑤ 用 debug 模式来set渲染

``` bash
helm install --dry-run --debug --set address=k8s ./mychart     
```


* ⑥ 结果化的values内容

``` bash
address: 
  url: idig8.com
  name: hello-k8s
```

``` bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ .Values.address.url }}
  name: {{ .Values.address.name }}
```

``` bash
helm install --dry-run --debug  ./mychart  
```
