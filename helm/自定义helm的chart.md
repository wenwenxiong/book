###识别chart的其他容器名
wordpress中通过```{{ template "mariadb.fullname" . }}```识别子chart mariadb的容器全名，mariadb.fullname是通过在wordpress的chart目录下template目录下的_helpers.tpl文件中定义的
```
...
{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
*/}}
{{- define "mariadb.fullname" -}}
{{- printf "%s-%s" .Release.Name "mariadb" | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```
在定义自己的chart时可参考此做法来识别chart的其他容器名。
###自定义chart在monocular上的图标和描述、版本
每个chart包中包含一个Chart.yml文件
里面可以定义以下字段，包含图标，描述，版本内容。
```
name: The name of the chart (required)
version: A SemVer 2 version (required)
description: A single-sentence description of this project (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this project's home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
maintainers: # (optional)
  - name: The maintainer's name (required for each maintainer)
    email: The maintainer's email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
engine: gotpl # The name of the template engine (optional, defaults to gotpl)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). This needn't be SemVer.
deprecated: Whether or not this chart is deprecated (optional, boolean)
tillerVersion: The version of Tiller that this chart requires. This should be expressed as a SemVer range: ">2.0.0" (optional)
```
问题：描述，版本等文本类的修改后生效，但是图片修改后没有效果，可能是对图片的大小有所限制。
###定义chart的依赖集
在chart包目录下定义一个requirements.yaml文件，里面编写该chart的依赖，例如
```
dependencies:
  - name: apache
    version: 1.2.3
    repository: http://example.com/charts
  - name: mysql
    version: 3.2.1
    repository: http://another.example.com/charts
```
需要填写依赖chart的name，version，repository。
使用命令
```
helm dependency update parentchart
```
会下载parentchart所需的依赖chart包到charts文件夹下。

