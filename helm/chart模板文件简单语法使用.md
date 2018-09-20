参考网址: https://docs.helm.sh/chart_template_guide/#the-chart-template-developer-s-guide
###helm 模板

helm模板语法嵌套在```{{```和```}}```之间，有三个常见的
.Values.*
从value.yaml文件中读取
.Release.*
从运行Release的元数据读取
.Template.*
.Chart.*
从Chart.yaml文件中读取
.Files.*
.Capabilities.*

可以通过命令
```
helm install --debug --dry-run ./mychart
```
获取helm渲染后的k8s可执行的yaml文件（只渲染不运行）。

.Values.*的值可以来自以下
+ values.yaml文件
+ 如果是子chart，值来自父chart的values.yaml
+ 通过helm install -f标志的文件
+ 来自--set中的配置

顺序查找，下面找到的覆盖上面找到的值。
###模板函数和管道

quote是最常用的模板函数，它能把ABC转化为“ABC”。它带一个参数
```{{ quote .Values.favorite.drink }}```

| 管道，类似linux下的管道。
```{{ quote .Values.favorite.drink }}``` 与 ```{{ .Values.favorite.drink | quote }}``` 效果一样。

default 模板函数
制定默认值
drink: ```{{ .Values.favorite.drink | default "tea" | quote }}```
如果在values中无法找到favorite.drink，则配置为“tea”。

indent 模板函数
对左空出空格
例如
```
data:
  myvalue: "Hello World"
{{ include "mychart_app" . | indent 2 }}
```
会使渲染后的取值于左边空出两个空格，以符合yaml语法。
###模板流程控制
模板中控制流程语句可以控制值的生成
 常用的有
 + if/else 条件控制
 + with 范围控制
 + range 循环控制

此外，模板中还可以定义变量
values.yaml
```
favorite:
  drink: coffee
  food: pizza
```
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end}}
```
定义了$key和$val两个变量，并且赋值。渲染后
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: eager-rabbit-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
```
###命名模板使用
一般在template文件夹下的_helpers.tpl中定义命名模板。
通过define 函数定义命名模板
template使用命名模板
例如，在_helpers.tpl中定义命名模板
```
{{/* Generate basic labels */}}
{{- define "my_labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
```
第一行为描述命名模板```{{/* Generate basic labels */}}```。
使用命名模板
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "my_labels" . }}
```
渲染后为
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: plinking-anaco-configmap
  labels:
    generator: helm
    date: 2016-11-02
    chart: mychart
    version: 0.1.0
```
###在模板中使用文件
基本使用
例如，在chart的根目录下有三个文件

config1.toml:

message = Hello from config 1
config2.toml:

message = This is config 2
config3.toml:

message = Goodbye from config 3
在模板文件中使用
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $files := .Files }}
  {{- range tuple "config1.toml" "config2.toml" "config3.toml" }}
  {{ . }}: |-
    {{ $files.Get . }}
  {{- end }}
```
$files变量赋值为.Files对象，通过$files.Get 获取文件内容。
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: quieting-giraf-configmap
data:
  config1.toml: |-
    message = Hello from config 1

  config2.toml: |-
    message = This is config 2

  config3.toml: |-
    message = Goodbye from config 3
```
上面的用法对于单行文件在configmap中使用有效，但是对于文件内容较多，则使用另一种方法（不过文件内容不能超过1M，因为整个chart包传入tiller限制存储为1M）。
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: conf
data:
{{ (.Files.Glob "foo/*").AsConfig | indent 2 }}
---
apiVersion: v1
kind: Secret
metadata:
  name: very-secret
type: Opaque
data:
{{ (.Files.Glob "bar/*").AsSecrets | indent 2 }}
```
上面模板文件中查找chart根目录下foo目录的所有文件配置为configmap的内容。bar目录下所有文件配置为sectet的内容。
###子chart和全局values
子chart是父chart依赖的chart，父chart在根目录的requirements.yaml中声明依赖的子chart，并且在charts目录中存放子chart。
1. 子chart不能访问父chart的values
2. 父chart可以覆盖子chart的values
3. helm中可以定义global values被所有charts使用
4. 父chart和子chart可以共享模板函数，定义在任何地方的自定义模板函数可以被所有的chart使用。

global values声明与使用
mychart/values.yaml
```
favorite:
  drink: coffee
  food: pizza

mysubchart:
  dessert: ice cream

global:
  salad: caesar
```
在global下定义即可
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  salad: {{ .Values.global.salad }}
```