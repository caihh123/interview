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

# 流程控制语句

## if/else 

## with 

## range

# helm3中变量详解

## 声明变量的格式和作用

变量通常是搭配with语句和range语句使用，这样能有效的简化代码，变量的定义格式如下: 

```yaml
$name:=value
```

#### 使用变量解决对象作用域问题

因为with语句里不能调用父级别的变量，所以如果需要调用父级别的变量，需要声明一个变量名，将父级别的变量值赋值给声明的变量在前面关于helm流控制结构的文章中提到过使用with更改当前作用域的用法，当时存在一个问题是在with语句中，无法使用父作用域中的对象，需要使用 $符号或者将语句移到{{- end }}的外面才可以。现在使用变量也可以解决这个问题。示例:

values.yaml

```yaml
people:
	info:
		name: test
		age: 18
		sex: boy
```

xxx.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: {{ .Release.Name }}-configmap
data:
	{{ -$releaseName := .Release.Name }}
	{{ -with .Values.people.info }}
	name: {{ .name }}
	age: {f .age }}
	release3: {{ .Release.Name }}
```




## 作用域

## 列表或元组

## 字典

# helm3中子模板定义和使用

## define

## template

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