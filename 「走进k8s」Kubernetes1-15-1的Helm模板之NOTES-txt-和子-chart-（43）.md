

#### （一）NOTES.txt 文件

* ② 编写NOTES.txt

``` bash
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get {{ .Release.Name }}
```


* ③ 编写configmap.yaml

``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "idig8.com" | quote }}
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


* ④ 运行查看NOTES.txt打印

``` bash
helm install  ./mychart
```


#### （二）子 chart 包


``` bash
cd charts
helm create mysubchart
rm -rf mysubchart/templates/*.*
rm -rf mysubchart/templates/tests/
```



``` bash
cd ..
cd ..

tree
```



* ④ 创建configmap.yaml

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap2
data:
  in: {{ .Values.in }}
```




*  ⑤ 子chart创建values.yaml

``` bash
in: mysubchart
```




* ⑥ 测试运行

``` bash
helm install --dry-run --debug .
```




#### （三）值覆盖

* ① 修改父的values.yaml

``` yml
address:
  url: idig8.com
  name: hello-k8s
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
mysubchart:
  in: "1245"
```



* ② 运行父chart


``` bash
helm install --dry-run --debug .
```


#### （四）全局值


``` bash
address:
  url: idig8.com
  name: hello-k8s
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
mysubchart:
  in: "1245"
global:
  idig8: helm-global
```




``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap2
data:
  in: {{ .Values.in }}
hello:
  name:{{ .Values.global.idig8 }}
```



* ③ 运行结果


``` bash
helm install --dry-run --debug .
```



