

# helm介绍、组件、安装和目录结构

## 传统服务部署到k8s集群的流程

拉取代码-->打包编译-->构建镜像-->准备一堆相关部署的yaml文件(如:deployment、senvice、ingres等)-->kubectl apply 部署到k8s集群

## 传统方式部署引发的问题

- 随着引用的增多，需要维护大量的yaml文件
- 不能根据一套yaml文件来创建多个环境，需要手动进行修改。

例如：一般环境都分为dev、预生产、生产环境，部署完了dev这套环境，后面再部署预生产和生产环境，还需要复制出两套，并手动修改才行。

## 什么是helm

Helm是Kubernetes的包管理工具，可以方便地发现、共享和构建Kubernetes应用。helm是k8s的包管理器，相当于centos系统中的yum工具，可以将一个服务相关的所有资源信息整合到一个chart包中，并且可以使用一套资源发布到多个环境中， 可以将应用程序的所有资源和部署信息组合到单个部署包中。就像Linux下的rpm包管理器，如yum/apt等，可以很方便的将之前打包好的yaml文件部署kubemetes上。

## helm的组件

1. Chart:

helm的一个整合后的chart包，包含一个应用所有的kubemmetes声明模版，类似于yum的rpm包或者apt的dpkg文件。helm将打包的应用程序部暑到k8s,并将它们构建成Chart。这些Chant将所有预配置的应用程序资源以及所有版本都包含在一个易于管理的包中。Helm把kuberhetes资源(如deployments、senvices或ingress等)打包到一个chant中，chart被保存到chant仓库。通过chant仓库可用来存储和分享chart

2. Helm客户端

helm的客户端组件，负责和k8s apiserver通信

3. Repository

用于发布和存储chart包的仓库，类似yum仓库或docker仓库

4. Release

用chart包部署的一个实例。通过chart在k8s中部署的应用都会产生一个唯一的Release.同-chart部署多次就会产生多个Release理解:将这些yaml部署完成后，他也会记录部署时候的一个版本，维护了一个release版本状态，通过Release这个实例，他会具体帮我们创建pod,

 ## heml3和helm2的区别

1. helm3移除了Tiller组件

helm2中helm客户端和k8s通信是通过Tiller组件和k8s通信，helm3移除了Tiler组件，直接使用kubeconfg文件和k8s的apiserver通信

2. 删除 release 命令变更

```shell
helm delete release-name -purge --> helm uninstall release-name
```

3. 查看 charts 信息命令变更

```shell
helm inspect release-name --> helm show release-name
```

4. 拉取 charts 包命令变更

```shell
helm fetch chart-name --> helm pull chart-name
```

5. helm3中必须指定release名称，生成一个随机名称需要加选项--generate-name，helm2中不指定release名称，会自动生成

```shell
helm install ./mychart --generate-name
```

## 安装

```shell
https://helm.sh/zh/docs/intro/install/
```

```shell
$ wget https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz
$ tar -zxvf helm-v3.7.1-linux-amd64.tar.gz
$ cd linux-amd64/
$ ls
helm  LICENSE  README.md
$ cp helm /usr/bin/
```

## 目录

```shell
# 创建
helm helm create mychart
# 目录
➜  mychart tree
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

| 字段            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| Chart.yaml      | 保存chart的基本信息，包括名字、描述信息及版本等，这个变量文件都可以被templates目录下文件所引用 |
| charts          | 存放子chart的目录，目录力存放着这个chart依赖的所有子chart    |
| templates       | 模板文件目录，存放着所有的yaml模板文件，包含所有部署应用的yaml文件 |
| _helpers.tpl    | 放置模板助手，可以在整个chart中重复使用，是放一些templates目录下这些yaml都有可能会用的一些模板 |
| NOTES.txt       | 部署chart后给用户的提示信息，介绍chart帮助信息，helm instali部署后展示给用户，如何使用chart等 |
| deployment.yaml | 创建deployment对象的模板文件                                 |
| tests           | 用于测试的文件，测试完部署完chart后，如web，做一个链接，看看你是否部署正常 |
| values.yaml     | 用于渲染模板的文件(变量文件，定义变量的值) 定义templates目录下的yaml文件可能引用到的变量#values.yaml用于存储 templates 目录中模板文件中用到变量的值，这些变量定义都是为了让templates目录下yaml引用 |

# 编写一个chart和helm内置对象详解

## 初识

### 创建chart包

```shell
helm create mychart
```

### 创建chart实例

```shell
helm install myconfigmap ./mychart/
```

### 查看创建后的相关信息和验证是否已经在k8s中创建成功

```
helm get manifest myconfigmap
```

### 删除实例

```shell
helm uninstall myconfigmap
```

### 预运行命令，只渲染，测试使用

```shell
helm install myconfigmap2 ./mychart/ --debug --dry-run
```

## 内置对象

### 常用的内置对象

- Release
- Values
- Chart
- Capabilities
- Template

### Release

| 名称              | 含义                                                        |
| ----------------- | ----------------------------------------------------------- |
| Release.Name      | release的名称                                               |
| Release.Namespace | release的命名空间                                           |
| Release.IsUpgrade | 如果当前操作是升级或回滚的话，是true                        |
| Release.IsInstall | 如果当前操作是安装的话，是true                              |
| Release.Revision  | 获取此次修订的版本号，初次安装时是1，每次升级和回滚都会递增 |
| Release.Service   | 获取渲染当前模板的服务名称，一般都是helm                    |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  value1: "{{ .Release.IsUpgrade }}"
  value2: "{{ .Release.IsInstall }}"
  value3: "{{ .Release.Revision }}"
  value4: "{{ .Release.Service }}"
```

### Value

| Value键值对               | 获取方式           |
| ------------------------- | ------------------ |
| name1:test1               | .Values.name1      |
| info:<br/>	name2:test2 | .Values.info.name2 |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  value1: "{{ .Values.name1 }}"
  value2: "{{ .Values.info.name2 }}"
```

### Chart

| 名称           | 含义      |
| -------------- | --------- |
| .Chart.Name    | Chart名称 |
| .Chart.Version | Chart版本 |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  value1: "{{ .Chart.Name }}"
  value2: "{{ .Chart.Version }}"
```

### Capabilities

| 名称                                                         | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| .Capabilities.APIVersion                                     | Kubernetes集群API版本信息集合                                |
| .Capabilities.APIVersion.Has $Version                        | 检测指定版本或资源在k8s集群中是否可用，例如：apps/v1/Deployment |
| .Capabilities.KubeVersion和.Capabilities.KubeVersion.Version | 获取Kubernetes版本号                                         |
| .Capabilities.KubeVersion.Major                              | 获取Kubernetes主版本号                                       |
| .Capabilities.KubeVersion.Minor                              | 获取Kubernetes小版本号                                       |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  value1: "{{ .Capabilities.APIVersions }}"
  value2: '{{ .Capabilities.APIVersions.Has "apps/v1/Deployment" }}'
  value3: "{{ .Capabilities.KubeVersion.Version }}"
  value4: "{{ .Capabilities.KubeVersion.Major }}"
  value5: "{{ .Capabilities.KubeVersion.Minor }}"
```

### Template

| 名称               | 含义                     |
| ------------------ | ------------------------ |
| .Template.Name     | 描述当前模板的名称和路径 |
| .Template.BasePath | 获取当前模板的路径       |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  value1: "{{ .Template.Name }}"
  value2: "{{ .Template.BasePath }}"
```

# helm3常用命令和部署应用实战案例

## 常用命令

```shell
Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  push        push a chart to remote
  registry    login to or logout from a registry
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information

Flags:
      --burst-limit int                 client-side default throttling limit (default 100)
      --debug                           enable verbose output
  -h, --help                            help for helm
      --kube-apiserver string           the address and the port for the Kubernetes API server
      --kube-as-group stringArray       group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string             username to impersonate for the operation
      --kube-ca-file string             the certificate authority file for the Kubernetes API server connection
      --kube-context string             name of the kubeconfig context to use
      --kube-insecure-skip-tls-verify   if true, the Kubernetes API server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kube-tls-server-name string     server name to use for Kubernetes API server certificate validation. If it is not provided, the hostname used to contact the server is used
      --kube-token string               bearer token used for authentication
      --kubeconfig string               path to the kubeconfig file
  -n, --namespace string                namespace scope for this request
      --qps float32                     queries per second used when communicating with the Kubernetes API, not including bursting
      --registry-config string          path to the registry config file (default "/Users/tu/Library/Preferences/helm/registry/config.json")
      --repository-cache string         path to the directory containing cached repository indexes (default "/Users/tu/Library/Caches/helm/repository")
      --repository-config string        path to the file containing repository names and URLs (default "/Users/tu/Library/Preferences/helm/repositories.yaml")
```

### 添加仓库

```shell
➜  helm helm repo add stable http://mirror.azure.cn/kubernetes/charts
➜  helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
➜  helm repo list
NAME  	URL
stable	http://mirror.azure.cn/kubernetes/charts
aliyun	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
➜  helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

### 搜索

```
➜  helm helm search repo tomcat
NAME         	CHART VERSION	APP VERSION	DESCRIPTION
stable/tomcat	0.4.3        	7.0        	DEPRECATED - Deploy a basic tomcat application ...
➜  helm helm show chart stable/tomcat
...
image:
  webarchive:
    repository: ananwaresystems/webarchive
    tag: "1.0"
  tomcat:
    repository: tomcat
    tag: "7.0"
  pullPolicy: IfNotPresent
  pullSecrets: []
...
```

### 拉取

```shell
➜  helm pull aliyun/redis --version 1.1.15 --untar
➜  ls
mychart redis
➜  helm pull aliyun/redis --version 1.1.15
➜  ls
mychart          redis            redis-1.1.15.tgz
```

### 安装

```shell
helm install redis stable/redis
```

## 案例

nginx-deploy-service.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Values.deployment_name }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.pod_label }}
  template:
    metadata:
      labels:
        app: {{ .Values.pod_label }}
    spec:
      containers:
        - image: {{ .Values.image }}:{{ .Values.imageTag }}
          name: {{ .Values.container_name }}
          ports:
          - containerPort: {{ .Values.containerport }}

---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service_name }}
  namespace: {{ .Values.namespace }}
spec:
  type: NodePort
  ports:
    - port: {{ .Values.port }}
      targetPort: {{ .Values.targetport }}
      nodePort: {{ .Values.nodeport }}
      protocol: TCP
  selector:
    app: {{ .Values.pod_label }}
```

values.yaml

```yaml
deployment_name: nginx-deployment
replicas: 2
pod_label: nginx-pod-label
image: nginx
imageTag: 1.20.0
container_name: nginx-container
service_name: nginx-service
namespace: default
port: 80
targetport: 80
containerport: 80
nodeport: 30001
```

命令

```shell
helm install nginx-release ./mychart/
helm get manifest nginx-release
helm list
helm upgrade nginx-release ./mychart/ -f ./mychart/values.yaml
helm rollback nginx-release
helm list
helm history nginx-release
helm rollback nginx-release 2
helm list
helm upgrade nginx-release ./mychart/ --set imageTag=1.19
helm list
helm history nginx-release
```

# helm内置函数详解

## 内置函数

###  常用函数

- quote和sqlute
- upper和lower 
- repeat 
- default
- lookup

### 函数使用

```shell
格式1: 函数名 arg1 arg2 ...
格式2: arg1 | 函数名
```

 ### 案例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  values1: {{ quote .Values.name }}               # 加双引号
  values2: {{ squote .Values.name }}              # 加单引号
  values3: {{ .Values.name | quote }}             # 加双引号
  values4: {{ .Values.name | squote }}            # 加单引号
  values5: {{ .Values.name | upper | quote }}     # 将字母大写
  values6: {{ .Values.name | lower | squote }}    # 将字母小写
  values7: {{ .Values.name | repeat 2 }}          # 重复输出两次
  values8: {{ .Values.test | default "haha" }}    # 默认值

```

lookup 

使用lookup函数用于在当前的k8s集群中获取一些资源的信息，功能有些类似于kubectl get，函数的格式如下:

```shell
lookup "apiversion" "kind" "namespace""name"
```

其中namespace和name都是可选的，或可以指定为空字符串， 函数执行完成后会返回特定的资源常用格式和kubect命令相互对应关系:

| kubectl命令                          | lookup 函数                              |
| ------------------------------------ | ---------------------------------------- |
| kubectl get pod mypod -n mynamespace | lookup "v1" "Pod" "mynamespace" "mypod"  |
| kubectl get pods -n mynamespace      | lookup "v1" "Pod" "'mynamespace" ""      |
| kubectl get pods --all-namespaces    | lookup "v1" "Pod" "" ""                  |
| Kubectl get namespace mynamespace    | lookup "v1" "Namespace" "" "mynamespace" |
| kubectl get namespaces               | lookup "v1" "Namespace" "" ""            |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  values1: {{ lookup "v1" "Namespace" "" "" | quote }}
```

```shell
helm get manifest myconfigmap
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  values1: "map[apiVersion:v1 items:[map[apiVersion:v1 kind:Namespace metadata:map[creationTimestamp:2025-04-15T03:12:48Z labels:map[kubernetes.io/metadata.name:default] managedFields:[map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:metadata:map[f:labels:map[.:map[] f:kubernetes.io/metadata.name:map[]]]] manager:kube-apiserver operation:Update time:2025-04-15T03:12:48Z]] name:default resourceVersion:193 uid:a7fca939-a5ca-4575-ba78-6cb687cb7bb2] spec:map[finalizers:[kubernetes]] status:map[phase:Active]] map[apiVersion:v1 kind:Namespace metadata:map[creationTimestamp:2025-04-15T03:12:47Z labels:map[kubernetes.io/metadata.name:kube-node-lease] managedFields:[map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:metadata:map[f:labels:map[.:map[] f:kubernetes.io/metadata.name:map[]]]] manager:kube-apiserver operation:Update time:2025-04-15T03:12:47Z]] name:kube-node-lease resourceVersion:6 uid:1bf92c1a-0a4d-449e-8865-33f9254673d2] spec:map[finalizers:[kubernetes]] status:map[phase:Active]] map[apiVersion:v1 kind:Namespace metadata:map[creationTimestamp:2025-04-15T03:12:47Z labels:map[kubernetes.io/metadata.name:kube-public] managedFields:[map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:metadata:map[f:labels:map[.:map[] f:kubernetes.io/metadata.name:map[]]]] manager:kube-apiserver operation:Update time:2025-04-15T03:12:47Z]] name:kube-public resourceVersion:5 uid:6c02e29c-ef13-43bb-a271-0a9797bc85a3] spec:map[finalizers:[kubernetes]] status:map[phase:Active]] map[apiVersion:v1 kind:Namespace metadata:map[creationTimestamp:2025-04-15T03:12:47Z labels:map[kubernetes.io/metadata.name:kube-system] managedFields:[map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:metadata:map[f:labels:map[.:map[] f:kubernetes.io/metadata.name:map[]]]] manager:kube-apiserver operation:Update time:2025-04-15T03:12:47Z]] name:kube-system resourceVersion:3 uid:88c285fc-de38-4533-8f4b-0dba029b03ce] spec:map[finalizers:[kubernetes]] status:map[phase:Active]]] kind:NamespaceList metadata:map[resourceVersion:43280]]"
```

## 逻辑和流控制函数

| 函数名  | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| eq      | 用于判断两个参数是否相等，如果等于则为true，不等于则为false  |
| ne      | 用于判断两个参数是否不相等，如果不等于则为true，等于则为false |
| lt      | 判断第一个参数是否小于第二个参数，如果小于则为true，如果大于则为false |
| le      | 判断第一个参数是否小于等于第二个参数，如果成立则为true，如果不成立则为false |
| gt      | 判断第一个参数是否大于第二个参数，如果大于则为tue，如果小于则为false |
| ge      | 判断第一个参数是否大于等于第二个参数，如果成立则为true，如果不成立则为 false |
| and     | 返回两个参数的逻辑与结果(布尔值)，也就是说如果两个参数为真，则结果为tue，否认哪怕一个为假，则返回false |
| or      | 判断两个参数的逻辑或的关系，两个参数中有一个为真，则为真     |
| not     | 用于对参数的布尔值取反                                       |
| default | 用来设置一个默认值，在参数的值为空的情况下，则会使用默认值   |
| empty   | 用于判断给定值是否为空，如果为空则返回true                   |
| coalese | 用于扫描一个给定的列表，并返回第一个非空的值                 |
| ternary | 接受两个参数和一个test值，如果test 的布尔值为true，则返回第一个参数的值，否则返回第二个参数的值 |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  values1: {{ eq 2 2}}
  values2: {{ ne 2 2}}
  values3: {{ lt 2 2}}
  values4: {{ le 2 2}}
  values5: {{ gt 2 2}}
  values6: {{ ge 2 2}}
  values7: {{ and 0 0}}
  values8: {{ or 0 1}}
  values9: {{ not 1 | quote}}
  values10: {{ .Values.test | empty }}
  values11: {{ .Values.test | default "haha" }}
  values12: {{ coalesce 0 1 2 }}
  values13: {{ coalesce "" "hello" "world" }}
  values14: {{ ternary "hello" "world" true }}
  values15: {{ ternary "hello" "world" false }}
```

## 字符串函数

1.常用helm3的字符串函数
(1).
(2).
(3).(4)(5)

|                                                 |                                                              |
| ----------------------------------------------- | ------------------------------------------------------------ |
| print，println                                  | 打印，println会在两个非字符串之间加空格，行尾家换行          |
| printf                                          | 按格式打印                                                   |
| trim，trimAll，trimPrefix，trimSufix            | 用来对字符串进行修剪，trimAll去除首尾的                      |
| lower，upper，title，untitle                    | 大小写，首字母处理                                           |
| snakecase，camelcase，kebabcase                 | 驼峰，下划线，中横线转换                                     |
| swapcase                                        | 大写表小写，首字母小写，空格或开头小写转大写，其他小写转大写 |
| substr                                          | 字符串切割                                                   |
| trunc                                           | 截断字符串，正数从左向右，负数从右向左                       |
| abbrev                                          | 保留前n-3个字符，最后三个是...，不满足不变，不够填充...不变  |
| randAlphaNum，randAlpha，randNumeric，randAscii | 生成加密随机字符串                                           |
| contains                                        | 是否有子串                                                   |
| hasPrefix和hasSuffix                            | 前缀后缀判断                                                 |
| repeat                                          | 重复输出指定次数                                             |
| nospace                                         | 去掉空格                                                     |
| initials                                        | 截取字符串首字母                                             |
| wrapWith                                        | 在指定列添加内容                                             |
| quote和squote                                   | 添加双引号单引号                                             |
| cat                                             | 将多个字符串合成一个，用空格分开                             |
| replace                                         | 替换字符串                                                   |
| shuffle                                         | 将字符串顺序打乱                                             |
| indent和nindent                                 | 指定长度缩进指定字符串的所在行，nindent可以在字符串开头添加新行 |
| plural                                          |                                                              |

```yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace }}
data:
  values1: {{ print 2 3 "hello" "haha" }}
  values2: {{ println 2 3 "hello" "haha" }}
  values3: {{ printf "hello %s!" "world"}}
  values4: {{ printf "hello %d" 10 }}
  values5: {{ printf "hello %.2f" 10.12312 }}
  values6: {{ trim "   hello world! " }}
  values7: {{ trimAll "%" "%hello%world!" }}
  values8: {{ trimPrefix "-" "-hello" }}
  values9: {{ trimSuffix "-" "hello-" }}
  values10: {{ lower "HELLO" }}
  values11: {{ upper "hello" }}
  values12: {{ title "hELLO" }}
  values13: {{ untitle "HELLO" }}
  values14: {{ snakecase "UserName" }}
  values15: {{ camelcase "user_name" }}
  values16: {{ kebabcase "UserName" }}
  values17: {{ swapcase "This is A.Test" }}
  values18: {{ substr 2 4 "agjfkal" }}
  values19: {{ trunc 5 "Hello World" }}
  values20: {{ trunc -5 "Hello World" }}
  values21: {{ abbrev 5 "Hello World" }}
  values22: {{ randAlphaNum 10 }}
  values23: {{ randAlpha 10 }}
  values24: {{ randNumeric 10 }}
  values25: {{ randAscii 10 }}
  values26: {{ contains "llo" "Hello" }}
  values27: {{ hasPrefix "He" "Hello" }}
  values28: {{ hasSuffix "lo" "Hello" }}
  values29: {{ repeat 3 "11"}}
  values30: {{ nospace " H  e l l o " }}
  values31: {{ initials "Are You Ok" }}
  values32: {{ wrapWith 5 " " "HelloWorld" }}
  values33: {{ hello | quote }}
  values34: {{ hello | squote }}
  values35: {{ cat "Hello" "World" }}
  values36: {{ "Hello%World" | replace "%" " " }}
  values37: {{ shuffle "abcdefghijklmn" }}
  values38: {{ indent 4 "this is indent" }}
  values39: {{ nindent 4 "this is nindent" }}
  values40: {{ len "a" | plural "one" "many" }}
  values41: {{ len "abc" | plural "one" "many" }}
  values42: {{ len "" | plural "one" "many" }}
```

## 类型转换和正则表达式函数

### 类型转换函数

| 函数         | 含义                                                     |
| ------------ | -------------------------------------------------------- |
| atoi         | 将字符串转换为整数                                       |
| foat64       | 转换成foat64                                             |
| int          | 转换为int类型                                            |
| toString     | 转换成字符串                                             |
| int64        | 转换成int64                                              |
| toDecimal    | 将unix八进制转换成 int64                                 |
| toStrings    | 将列表、切片或数组转换成字符串                           |
| toJson       | 将列表、切片、数组、字典或对象转换成JSON                 |
| toPrettyJson | 将列表、切片、数组、字典或对象转换成格式化JSON           |
| toRawJson    | 将列表、切片、数组、字典或对象转换成ISON(HTML字符不转义) |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  values1: {{ 16 | kindOf }}
  values2: {{ "16" | kindOf }}
  values3: {{ atoi "16" }}
  values4: {{ float64 "16.1" }}
  values5: {{ int "16" }}
  values6: {{ toString "16" }}
```

### 正则表达式函数

| 函数                                               | 含义                                                 |
| -------------------------------------------------- | ---------------------------------------------------- |
| regexFind和mustRegexFind                           | 获取字符串中正则匹配的第一个结果                     |
| regexFindAll和mustRegexFindAll                     | 获取字符串中正则匹配的子串内容，-1表示所有           |
| regexMatch和mustRegexMatch                         | 根据正则表达式匹配                                   |
| regexReplaceAll和mustRegexReplaceAll               | 指定替换字符替换正则表达式匹配的内容                 |
| regexReplaceAllLiteral和mustRegexReplaceAllLiteral | 将正则表达式匹配的内容替换成爱他内容                 |
| regexSplit和mustRegexSplit                         | 指定分隔符(以正则匹配的内容为分隔符)对字符串进行切割 |

在Helm的模板引擎中，`regexReplaceAll`和`regexReplaceAllLiteral`均用于正则表达式替换，但它们在处理替换字符串时存在关键区别：

### 1. `regexReplaceAll`
- **功能**：执行正则替换，并**解析替换字符串中的特殊语法**（如反向引用`$1`、`$name`等）。
- **行为**：
  - 替换字符串中的`$1`、`$2`等会被视为正则表达式的捕获组引用。
  - 支持正则替换的高级语法（如`${group}`）。
- **示例**：
  ```gotemplate
  {{ regexReplaceAll "(foo)(bar)" "foobar" "${1}-${2}" }} 
  # 输出：foo-bar
  ```

### 2. `regexReplaceAllLiteral`
- **功能**：执行正则替换，但**替换字符串中的所有字符均按字面量处理**。
- **行为**：
  - 替换字符串中的`$1`、`\n`等特殊字符不会被解析，直接作为普通文本插入。
  - 适用于需要保留`$`、`\`等字符的场景。
- **示例**：
  ```gotemplate
  {{ regexReplaceAllLiteral "(foo)" "foo" "$$1" }} 
  # 输出：$1（而不是捕获组的内容）
  ```

### 关键区别总结
| 函数                     | 处理替换字符串的方式         | 适用场景                           |
| ------------------------ | ---------------------------- | ---------------------------------- |
| `regexReplaceAll`        | 解析反向引用和特殊语法       | 需要动态引用捕获组内容时（如`$1`） |
| `regexReplaceAllLiteral` | 完全按字面量替换，不解析语法 | 需保留`$`、`\`等字符时             |

### 示例场景
- **使用`regexReplaceAll`**：
  ```gotemplate
  # 输入：Hello 123 World
  {{ regexReplaceAll "([A-Za-z]+) (\d+)" "Hello 123 World" "${2}_${1}" }}
  # 输出：123_Hello World
  ```

- **使用`regexReplaceAllLiteral`**：
  ```gotemplate
  # 输入：Hello 123 World
  {{ regexReplaceAllLiteral "(\d+)" "Hello 123 World" "$$1" }}
  # 输出：Hello $1 World
  ```

根据是否需要处理替换字符串中的正则语法，选择合适的函数以避免意外结果。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ regexFind "[a-zA-z][1-9]" "abcd1234c1" | quote }}
  data2: {{ mustRegexFind "[a-zA-z][1-9]" "abcd1234c1" | quote }}
  data3: {{ regexFindAll "[2,4,6,8]" "123456789" 3 | quote }}
  data4: {{ regexFindAll "[2,4,6,8]" "123456789" -1 | quote }}
  data5: {{ mustRegexFindAll "[2,4,6,8]" "123456789" 3 | quote }}
  data6: {{ mustRegexFindAll "[2,4,6,8]" "123456789" -1 | quote }}
  data7: {{ regexMatch "^[A-Za-z0-9.%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$" "test@xxx.com" | quote }}
  data8: {{ mustRegexMatch "^[A-Za-z0-9.%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$" "test@xxx.com" | quote }}
  data9: {{ regexReplaceAll "ab" "-ab-ab-axxb-" "W" | quote }}
  data10: {{ regexReplaceAll "a(x*)b" "-ab-axxb-" "W" | quote }}
  data11: {{ regexReplaceAll "a(x*)b" "-axb-axxb-" "W" | quote }}
  data12: {{ regexReplaceAll "a(x*)b" "-ab-axxb-" "${1}W" | quote }}
  data13: {{ mustRegexReplaceAll "ab" "-ab-ab-axxb-" "W" | quote }}
  data14: {{ mustRegexReplaceAll "a(x*)b" "-ab-axxb-" "W" | quote }}
  data15: {{ mustRegexReplaceAll "a(x*)b" "-axb-axxb-" "W" | quote }}
  data16: {{ mustRegexReplaceAll "a(x*)b" "-ab-axxb-" "${1}W" | quote }}
  data17: {{ regexReplaceAllLiteral "ab" "-ab-ab-axxb-" "W" | quote }}
  data18: {{ regexReplaceAllLiteral "a(x*)b" "-ab-axxb-" "W" | quote }}
  data19: {{ regexReplaceAllLiteral "a(x*)b" "-axb-axxb-" "W" | quote }}
  data20: {{ regexReplaceAllLiteral "a(x*)b" "-ab-axxb-" "${1}W" | quote }}
  data21: {{ mustRegexReplaceAllLiteral "ab" "-ab-ab-axxb-" "W" | quote }}
  data22: {{ mustRegexReplaceAllLiteral "a(x*)b" "-ab-axxb-" "W" | quote }}
  data23: {{ mustRegexReplaceAllLiteral "a(x*)b" "-axb-axxb-" "W" | quote }}
  data24: {{ mustRegexReplaceAllLiteral "a(x*)b" "-ab-axxb-" "${1}W" | quote }}
  data25: {{ regexSplit "z+" "pizza" -1 | quote }}
  data26: {{ mustRegexSplit "z+" "pizza" -1 | quote }}
```

## 加密函数和编解码函数

- 
  sha1sum
- sha256sum
- adler32sum
- htpasswd
- encryptAES
- decryptAES

helm有以下编码和解码函数:

b64enc编码函数和b64dec解码函数:编码或解码 Base64

b32enc编码函数和b32dec解码函数:编码或解码 Base32

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ sha1sum "Hello World" | quote }}
  data2: {{ sha256sum "Hello World" | quote }}
  data3: {{ adler32sum "Hello World" | quote }}
  data4: {{ htpasswd "user" "1234456" | quote }}
  data5: {{ "test" | b64enc | quote }}
  data6: {{ "dGVzdA==" | b64dec | quote }}
```

## 日期函数

- now
- date
- dateInZone
- duration和durationRound
- unixEpoch
- dateModify和mustDateModify
- toDate和mustToDate

| 函数                        | 含义                                                         |
| --------------------------- | ------------------------------------------------------------ |
| now                         | 用于返回当前日期和时间，通常与其他日期函数共同使用           |
| date                        | 用于将日期信息进行格式化，需指明日期的格式，格式必须使用"2006-01-02"或 "02/01/2006"来标明，否则会出错 |
| dateInZone                  | 用法与date函数基本一致，只不过dataInZone函数可以指定时区返回时间，如:指定UTC时区返回时间示 |
| duration                    | 将给定的秒数转换为Duration类型，例如指定 95秒可以返回1m35s，秒数必须需要使用双引号，否则会返回0s |
| durationRound               | 将给定的日期进行取整，保留最大的单位                         |
| unixEpoch                   | 返回给定时间的时间戳                                         |
| dateModify和must DateModify | 修改时间                                                     |
| toDate和mustToDate          | 将字符串转换成时间                                           |

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ now | quote }}
  data2: {{ now | date "2006-01-02" | quote }}
  data3: {{ dateInZone "2006-01-02" (now) "UTC" | quote }}
  data4: {{ duration "95" | quote }}
  data5: {{ durationRound "2h10m5s" | quote }}
  data6: {{ now | unixEpoch | quote }}
  data7: {{ now | dateModify "-2h" | quote }}
  data8: {{ toDate "2006-01-02" "2025-04-16" | quote }}
  data9: {{ mustToDate "2006-01-02" "2025-04-16" | quote }}
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "2025-04-16 17:06:17.322558 +0800 CST m=+0.155979918"
  data2: "2025-04-16"
  data3: "2025-04-16"
  data4: "1m35s"
  data5: "2h"
  data6: "1744794377"
  data7: "2025-04-16 15:06:17.322581 +0800 CST m=-7199.843996916"
  data8: "2025-04-16 00:00:00 +0800 CST"
  data9: "2025-04-16 00:00:00 +0800 CST"
```

## 字典函数

### dict

| 函数  | 含义             |
| ----- | ---------------- |
| get   | 获取value通过key |
| set   | 设置key，value   |
| unset | 删除key          |

示例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict := dict "1" "haha" "2" "hello"}}
  data1: {{ $myDict | quote }}
  data2: {{ get $myDict "1" | quote }}
  data3: {{ set $myDict "1" "test" | quote }}
  data4: {{ get $myDict "1" | quote }}
  data5: {{ unset $myDict "1" | quote }}
```

结果

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:

  data1: "map[1:haha 2:hello]"
  data2: "haha"
  data3: "map[1:test 2:hello]"
  data4: "test"
  data5: "map[2:hello]"
```

### keys 遍历

示例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello"}}
  {{ $myDict2 := dict "2" "zhangwuji" "3" "zhaoming"}}
  {{ $myDict3 := dict "4" "mary" "5" "mark"}}
  data1: {{ keys $myDict1 $myDict2 $myDict3 | quote }}
  data2: {{ keys $myDict1 $myDict2 $myDict3 | sortAlpha | uniq | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[1 2 2 3 4 5]"
  data2: "[1 2 3 4 5]"
```

### hasKey

示例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict := dict "1" "haha" "2" "hello"}}
  data1: {{ hasKey $myDict "1" | quote }}
  data2: {{ hasKey $myDict "3" | quote }}
```

结果

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "true"
  data2: "false"
```

### pluck

根据一个key获取多个dict的values

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello"}}
  {{ $myDict2 := dict "2" "zhangwuji" "3" "zhaoming"}}
  data1: {{ pluck "1" $myDict1 $myDict2 | quote }}
  data2: {{ pluck "2" $myDict1 $myDict2 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[haha]"
  data2: "[hello zhangwuji]"
```

### merge 

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello"}}
  {{ $myDict2 := dict "2" "zhangwuji" "3" "zhaoming"}}
  data1: {{ merge $myDict1 $myDict2 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "map[1:haha 2:hello 3:zhaoming]"
```

### mergeOverwrite

重复key会覆盖

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello"}}
  {{ $myDict2 := dict "2" "zhangwuji" "3" "zhaoming"}}
  data1: {{ mergeOverwrite $myDict1 $myDict2 | quote }}
  data2: {{ mustMergeOverwrite $myDict1 $myDict2 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "map[1:haha 2:zhangwuji 3:zhaoming]"
  data2: "map[1:haha 2:zhangwuji 3:zhaoming]"
```

### values

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello"}}
  data1: {{ values $myDict1 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[haha hello]"
```

### pick和omit

pick获取指定的key返回dict，omit返回没有被指定的

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello" "3" "hehe"}}
  data1: {{ pick $myDict1 "1" "2" | quote }}
  data2: {{ omit $myDict1 "1" "2" | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:

  data1: "map[1:haha 2:hello]"
  data2: "map[3:hehe]"
```

### deepCopy和mustDeepCopy

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myDict1 := dict "1" "haha" "2" "hello" "3" "hehe"}}
  data1: {{ deepCopy $myDict1 | quote }}
  data2: {{ mustDeepCopy $myDict1 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "map[1:haha 2:hello 3:hehe]"
  data2: "map[1:haha 2:hello 3:hehe]"
```

## 列表函数

### list，first，rest，last，initial

| list    | 生成一个列表                 |
| ------- | ---------------------------- |
| first   | 获取列表第一个               |
| rest    | 获取列表第一个以外的其他的   |
| last    | 获取列表最后一项             |
| initial | 获取列表最后一项以外的其他的 |

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myList := list 1 2 3 "haha" "hello" "hehe"}}
  data1: {{ $myList | quote }}
  data2: {{ first $myList | quote }}
  data3: {{ rest $myList | quote }}
  data4: {{ last $myList | quote }}
  data5: {{ initial $myList | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[1 2 3 haha hello hehe]"
  data2: "1"
  data3: "[2 3 haha hello hehe]"
  data4: "hehe"
  data5: "[1 2 3 haha hello]"
```

### append，prepend，concat，reverse，uniq

| append  | 向后追加   |
| ------- | ---------- |
| prepend | 向前追加   |
| concat  | 合并列表   |
| reverse | 反转列表   |
| uniq    | 去除重复项 |

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myList := list 1 2 3 "haha" "hello" "hehe"}}
  data1: {{ append $myList "test" | quote }}
  data2: {{ prepend $myList "test" | quote }}
  data3: {{ concat $myList (list 4 5 6) (list "zhangwuji" "zhaomin" "xiaozhao") | quote }}
  data4: {{ reverse $myList | quote }}
  data5: {{ $myList | uniq  | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[1 2 3 haha hello hehe test]"
  data2: "[test 1 2 3 haha hello hehe]"
  data3: "[1 2 3 haha hello hehe 4 5 6 zhangwuji zhaomin xiaozhao]"
  data4: "[hehe hello haha 3 2 1]"
  data5: "[1 2 3 haha hello hehe]"
```

### without，has，compact

| 函数    | 含义       |
| ------- | ---------- |
| without | 过滤指定列 |
| has     | 是否包含   |
| compact | 删除空值   |

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ without (list 1 2 3 4) 1 | quote }}
  data2: {{ without (list 1 2 3 4) 1 2 | quote }}
  data3: {{ without (list 1 2 3 4) 5 | quote }}
  data4: {{ has "test" (list 1 2 3 4) | quote }}
  data5: {{ has 2 (list 1 2 3 4) | quote }}
  data6: {{ compact (list "haha" "hello" "") | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[2 3 4]"
  data2: "[3 4]"
  data3: "[1 2 3 4]"
  data4: "false"
  data5: "true"
  data6: "[haha hello]"
```

### slice，until，untilStep，seq

| 函数      | 含义                       |
| --------- | -------------------------- |
| slice     | 对列表进行切割             |
| until     | 构建一个指定整数范围的列表 |
| untilStep | 可以定义整数的开始和步长   |
| seq       | 生成指定范围的整数         |

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ $myList := list 1 2 3 4 5 }}
  data1: {{ slice $myList | quote }}
  data2: {{ slice $myList 3 | quote }}
  data3: {{ slice $myList 1 3 | quote }}
  data4: {{ slice $myList 0 3 | quote }}
  data5: {{ until 5 | quote }}
  data6: {{ untilStep 3 9 2 | quote }}
  data7: {{ seq 5 | quote }}
  data8: {{ seq 0 2 | quote }}
  data9: {{ seq 0 2 10 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "[1 2 3 4 5]"
  data2: "[4 5]"
  data3: "[2 3]"
  data4: "[1 2 3]"
  data5: "[0 1 2 3 4]"
  data6: "[3 5 7]"
  data7: "1 2 3 4 5"
  data8: "0 1 2"
  data9: "0 2 4 6 8 10"
```

## 数学计算函数

### add，sub，mul，div，mod，add1

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ add 1 2 | quote }}
  data2: {{ sub 3 2 | quote }}
  data3: {{ mul 2 3 | quote }}
  data4: {{ div 9 3 | quote }}
  data5: {{ mod 9 2 | quote }}
  data6: {{ add1 2 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "3"
  data2: "1"
  data3: "6"
  data4: "3"
  data5: "1"
  data6: "3"
```

### max，min，round，len，floor，ceil

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ max 1 2 3 3 | quote }}
  data2: {{ min 1 2 3 1 | quote }}
  data3: {{ round 3.141926 3 | quote }}
  data4: {{ len "123123" | quote }}
  data5: {{ floor 3.12312 | quote }}
  data6: {{ ceil 3.12312 | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "3"
  data2: "1"
  data3: "3.142"
  data4: "6"
  data5: "3"
  data6: "4"
```

## 网络，文件路径，类型检查函数

### getHostByName

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ getHostByName "www.baidu.com" | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "110.4242.68.3"
```

### base，dir，ext，isAbs

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ base "/tmp/a/a.txt" | quote }}
  data2: {{ dir "/tmp/a/a.txt" | quote }}
  data3: {{ ext "a.txt" | quote }}
  data4: {{ isAbs "/tmp/a" | quote }}
  data5: {{ isAbs "tmp/a" | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "a.txt"
  data2: "/tmp/a"
  data3: ".txt"
  data4: "true"
  data5: "false"
```

### kindOf，kindIs,deepEqual

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  data1: {{ kindOf (list 1 2 3 ) | quote }}
  data2: {{ kindIs "string" "haha" | quote }}
  data3: {{ deepEqual (list 1 2 3 ) (list 1 2 3 ) | quote }}
  data4: {{ deepEqual (list 1 2 3 ) (list 2 3 4 ) | quote }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  data1: "slice"
  data2: "true"
  data3: "true"
  data4: "false"
```

# 流程控制语句

## if/else 

条件判断

案例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{- if ge (.Values.Person.age | int ) 18  }}
  result: "成年"
  {{- else }}
  result: "未成年"
  {{- end }}
```

结果:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  result: "成年"
```

## with 

控制变量的范围

案例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{- with .Values.people.info }}
  name: {{ .name | quote }}
  age: {{ .age | quote }}
  desc: {{ .desc | quote }}
  {{- end }}
```

结果:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  name: "mary"
  age: "19"
  desc: "haha"
```



## range

遍历集合输出

案例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{ range .Values.address }}
  {{- . | title }}
  {{ end }}
```

结果:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  Beijing
  Shanghai
  Xian
```

# helm3中变量详解

## 作用域

### 声明变量的格式和作用

变量通常是搭配with语句和range语句使用，这样能有效的简化代码，变量的定义格式如下: 

```yaml
$name:=value
```

#### 使用变量解决对象作用域问题

因为with语句里不能调用父级别的变量，所以如果需要调用父级别的变量，需要声明一个变量名，将父级别的变量值赋值给声明的变量在前面关于helm流控制结构的文章中提到过使用with更改当前作用域的用法，当时存在一个问题是在with语句中，无法使用父作用域中的对象，需要使用 $符号或者将语句移到{{- end }}的外面才可以。现在使用变量也可以解决这个问题。示例:

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  {{- $releaseName := .Release.Name }}
  {{- with .Values.people.info }}
  name: {{ .name | quote }}
  age: {{ .age | quote }}
  release: {{ $releaseName | quote }}
  {{- end }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  name: "mary"
  age: "19"
  release: "myconfigmap"
```

## 列表或元组

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  address: |-
    {{- range $index, $val := .Values.address }}
    {{ $index}}: {{ $val }}
    {{- end }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  address: |-
    0: beijing
    1: shanghai
    2: xian
```

## 字典

示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  address: |-
    {{- range $key, $val := .Values.people.info }}
    {{ $key }}: {{ $val }}
    {{- end }}
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
data:
  address: |-
    age: 19
    desc: haha
    name: mary
```

# helm3中子模板定义和使用

## define

### 定义位置

- 主模板
- _helpers.tpl

1. 使用define在主模板中定义子模板的，使用template进行调用子模板
2. 使用define在helpers.tpl文件中定义子模板，使用template进行调用子模板
3. 向子模板中传入对象、使用template进行调用子模板
4. 向子模板中传入对象，使用include进行调用子模板

_helpers.tpl

```yaml
{{- define "mychart.labels" -}}
  labels:
    author: test
    date: {{ now | htmlDate }}
{{- end -}}
```

configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
  {{ template "mychart.labels" }}
data:
  data1: "hello"
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
  labels:
    author: test
    date: 2025-04-21
data:
  data1: "hello "
```

## 

_helpers.tpl

```yaml
{{- define "mychart.labels" -}}
  labels:
    author: test
    name: {{ .people.info.name}}
{{- end -}}
```

configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
  {{ template "mychart.labels" .Values }}
data:
  data1: "hello"
```

结果：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap-configmap
  namespace: default
  labels:
    author: test
    name: mary
data:
  data1: "hello"
```

## 

## include

# helm3中获取其他文件的内容或文件名

## Get 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  token1:
  {{ .Files.Get "config/test.txt" | b64enc | indent 4 }}
  token2: {{ .Files.Get "config/test.txt" | quote }}
```

Glob

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  namespace: {{ .Release.Namespace}}
data:
  token1:
  {{ .Files.Get "config/test.txt" | b64enc | indent 4 }}
  token2: {{ .Files.Get "config/test.txt" | quote }}
```