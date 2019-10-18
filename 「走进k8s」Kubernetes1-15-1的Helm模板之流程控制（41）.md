
#### （一）if/else


* ① 介绍


* ② 实例

``` bash
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

* ③ 值为true 的情况，（其他为false）


* ④ 尝试修改，运行查看结果
``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  address: {{ .Values.address.url |quote | upper }}
  name: {{ .Values.address.name | repeat 3 | quote}}
  {{ if eq .Values.address.name "hello-k8s" }}web: "idig8.com" {{ end }}
```


> 运行查看结果

``` bash
helm install --dry-run --debug  ./mychart 
```



> 尝试set下变量，看能否打印web: "idig8.com"

``` bash
helm install --dry-run --debug  ./mychart  --set address.name="hello-docker"
```



``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  address: {{ .Values.address.url |quote | upper }}
  name: {{ .Values.address.name | repeat 3 | quote}}
  {{ if eq .Values.address.name "hello-k8s" }}
  web: "idig8.com"
  {{ end }}
```


> 运行下查看效果

``` bash
helm install --dry-run --debug  ./mychart
```




*  ⑥ else if 的使用

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  address: {{ .Values.address.url |quote | upper }}
  name: {{ .Values.address.name | repeat 3 | quote}}
  {{ if eq .Values.address.name "hello-k8s" }}
  web: "idig8.com"
  {{ else }}
  web: "toutiao.com"
  {{- end }}
```

``` bash
helm install --dry-run --debug  ./mychart --set address.name="baidu.com"
```




#### （二）空格控制


* ① 发现多了个空行，针对这个空行如何解决。

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  address: {{ .Values.address.url |quote | upper }}
  name: {{ .Values.address.name | repeat 3 | quote}}
  {{- if eq .Values.address.name "hello-k8s" }}
  web: "idig8.com"
  {{- else }}
  web: "toutiao.com"
  {{ end }}
```

``` bash
helm install --dry-run --debug  ./mychart
```



#### （三）使用 with 修改范围

* ① 介绍



* ② 语法

``` bash
{{ with PIPELINE }}
  #  restricted scope
{{ end }}

```



``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- with .Values.address}}
  address: {{ .url |quote | upper }}
  name: {{ .name | repeat 3 | quote}}
  {{- if eq .name "hello-k8s" }}
  web: "idig8.com"
  {{- else }}
  web: "toutiao.com"
  {{- end }}
  {{- end }}
```



``` bash
helm install --dry-run --debug  ./mychart
```



#### （四）range 循环



> 修改values.yaml文件

``` yaml
address:
  url: idig8.com
  name: hello-k8s
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```




``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- with .Values.address}}
  address: {{ .url |quote | upper }}
  name: {{ .name | repeat 3 | quote}}
  {{- if eq .name "hello-k8s" }}
  web: "idig8.com"
  {{- else }}
  web: "toutiao.com"
  {{- end }}
  {{- end }}
  pizzaToppings:
  {{- range .Values.pizzaToppings }}
  - {{ . | title | quote }}
  {{- end }}
```


* ④ 运行查看结果


``` bash
helm install --dry-run --debug  ./mychart
```


#### （五）变量


* ② 新增变量 

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- with .Values.address}}
  address: {{ .url |quote | upper }}
  name: {{ .name | repeat 3 | quote}}
  {{- if eq .name "hello-k8s" }}
  web: "idig8.com"
  {{- else }}
  web: "toutiao.com"
  {{- end }}
  {{- end }}
  pizzaToppings:
  {{- range .Values.pizzaToppings }}
  - {{ . | title | quote }}
  {{- end }}
  pizza:
  {{- range $index, $value := .Values.pizzaToppings }}
  {{ $index }}: {{ $value | title | quote }}
  {{- end }}
```



* ② 运行查看

``` bash
helm install --dry-run --debug  ./mychart
```



* ③ 上边说的作用域，如果有作用域的话，在使用with内部变量的话就会报错，为了解决这个问题，要引用变量

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- $releaseName := .Release.Name }}
  {{- with .Values.address}}
  address: {{ .url |quote | upper }}
  name: {{ .name | repeat 3 | quote}}
  release: {{ $releaseName }}
  {{- if eq .name "hello-k8s" }}
  web: "idig8.com"
  {{- else }}
  web: "toutiao.com"
  {{- end }}
  {{- end }}
  pizzaToppings:
  {{- range .Values.pizzaToppings }}
  - {{ . | title | quote }}
  {{- end }}
  pizza:
  {{- range $index, $value := .Values.pizzaToppings }}
  {{ $index }}: {{ $value | title | quote }}
  {{- end }}
```


> 运行下看看是否报错

``` bash
helm install --dry-run --debug  ./mychart
```
