
#### （一）声明和使用命名模板

* ① 介绍



``` bash
{{ define "ChartName.TplName" }}
# 模板内容区域
{{ end }}
```

* ③ 嵌入yaml的命名模板

> 模板嵌入到 ConfigMap 中，使用template关键字在需要的地方

``` yml
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- $releaseName := .Release.Name }}
  {{- with .Values.address}}
  release: {{ $releaseName }}
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
```



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


> 运行查看效果 

``` bash
helm install --dry-run --debug  ./mychart
```



* ③ 独立tpl


> configmap.yaml 的子模板 移动到  templates/_helpers.tpl 


``` yml 
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
```


> 删除原来的模板看能否自动引入

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- $releaseName := .Release.Name }}
  {{- with .Values.address}}
  release: {{ $releaseName }}
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



> 模板名称是全局的：从模板文件 templates/configmap.yaml 中移除，当然还是需要保留 template 来嵌入命名模板内容，名称还是之前的 mychart.lables。
``` bash
helm install --dry-run --debug  ./mychart
```



 #### （二）模板的作用范围

* ① 修改上边的_helpers.tpl，增加内置对象

> 子模板头部加一个简单的文档块，用/**/包裹起来 ，用于描述注释。{{ /* 注释 */ }}

``` yml
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
```



> 运行查看结果

``` bash
helm install --dry-run --debug  ./mychart
```



* ② 修改template 末尾传递了"." 

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- $releaseName := .Release.Name }}
  {{- with .Values.address}}
  release: {{ $releaseName }}
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



#### （三）include 函数

*  ① 修改_helper.tpl 模板

``` yml
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
from: helm
date: {{ now | htmlDate }}
chart: {{ .Chart.Name }}
version: {{ .Chart.Version }}
{{- end }}
```

![](https://upload-images.jianshu.io/upload_images/11223715-fbe430efd08cfb13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* ② 修改configmap.yaml文件

> 在 metadata区域需要2个空格，所以在管道函数indent中，传入参数2就可以。还是完全按照yaml的格式 配置 indent 4 

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
{{- include "mychart.labels" . | indent 2 }}
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
  {{- $releaseName := .Release.Name }}
  {{- with .Values.address}}
  release: {{ $releaseName }}
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


* ③ 查看运行结果

``` bash
helm install --dry-run --debug  ./mychart
```

