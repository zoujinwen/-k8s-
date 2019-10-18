

#### （一）模板函数


* ② 调用quote模板函数

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ quote .Values.address.url }}
  name: {{ .Values.address.name }}
```

* ③ 通过debug的方式查看效果

> address 增加了双引号
``` bash
helm install --dry-run --debug  ./mychart 

```



#### （二）管道


* ① quote函数，而是调换了一个顺序，使用一个管道符|将前面的参数发送给后面的模板函数
``` yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ .Values.address.url | quote }}
  name: {{ .Values.address.name }}
```


> 效果跟直接在函数前面是一样的

``` bash
helm install --dry-run --debug  ./mychart 

```


* ② 管道中增加了一个upper函数，将字符串每一个字母都变成大写


``` yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ .Values.address.url | quote | upper }}
  name: {{ .Values.address.name }}
```


> 效果address里面的value值，变成了大写

``` bash
helm install --dry-run --debug  ./mychart 
```



* ③ 管道的顺序性，一定要注意

> repeat 3 放后面

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ .Values.address.url | quote | upper }}
  name: {{ .Values.address.name  | quote | repeat 3 }}
```



> 查看下效果 

``` bash
helm install --dry-run --debug  ./mychart 
```


> repeat 3 放前面

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "idig8.com"
  address: {{ .Values.address.url | quote | upper }}
  name: {{ .Values.address.name | repeat 3 | quote  }}
```



> 查看下效果 

``` bash
helm install --dry-run --debug  ./mychart 
```


#### （三）default 函数

* ① 修改 configmap.yaml

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  address: {{ .Values.address.url | quote | upper }}
  name: {{ .Values.address.name  | quote | repeat 3 }}
```



* ② 查看values.yaml的信息

``` bash
cat ./mychart/values.yaml
```


* ③ 测试运行查看效果

``` bash
helm install --dry-run --debug  ./mychart 
```

