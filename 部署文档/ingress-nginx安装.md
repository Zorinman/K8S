Ingress是一种抽象概念，而ingress-nginx是其中一种实现
问：k8s问什么创建service,deployment等资源时可以通过yaml文件直接创建，但是创建ingress资源时需要先安装ingress?
答：
```
在 Kubernetes 中，创建 Deployment、Service 等资源时可以直接通过 YAML 文件创建，而创建 Ingress 资源需要先安装 Ingress 控制器
因为service,deployment等资源属于k8s的核心功能,Kubernetes 控制平面（如 kube-controller-manager）内置了这些资源的控制器,Deployment 由 deployment-controller 管理，负责创建和管理 Pod。
Service 由 kube-proxy 管理，负责实现服务发现和负载均衡。

Ingress 是扩展功能,Kubernetes 核心组件中没有内置 Ingress 控制器，需要用户自行安装第三方实现
Ingress 资源本身只是一个路由规则的声明,Ingress 资源需要由独立的 Ingress 控制器监听并实现具体的流量路由逻辑
可以将 Kubernetes 的 Ingress 资源比作“路由规则说明书”，而 Ingress 控制器则是“实际执行路由的工程师”
Ingress 控制器是一个独立的组件，负责以下任务：

监听 Ingress 资源：实时监控集群中的 Ingress 对象变化。

配置反向代理：根据 Ingress 规则动态配置负载均衡器（如 Nginx、Envoy 等）。

处理外部流量：将外部请求路由到集群内部的 Service 或 Pod。

Kubernetes 采用这种设计是为了保持核心功能的轻量化和灵活性：

核心功能开箱即用：Deployment、Service 等是大多数应用的基础需求，因此由 Kubernetes 直接支持。

扩展功能按需选择：不同的场景可能需要不同的 Ingress 实现（如云厂商的负载均衡器、自建 Nginx 等），因此 Kubernetes 将具体实现交给用户选择。
```

## 安装 ingress-nginx(这里使用helm包管理器安装）
### 1.[安装helm 自行查阅](helm安装.md)
### 2.添加ingress-nginx到helm仓库
```
# 添加仓库
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# 查看仓库列表
helm repo list

# 搜索 ingress-nginx
helm search repo ingress-nginx
```
### 3.下载安装包 
 
`helm pull ingress-nginx/ingress-nginx`
### 4.参数配置
将下载好的安装包解压  
`tar xf ingress-nginx-xxx.tgz`

 解压后，进入解压完成的目录  
`cd ingress-nginx`

### 修改 values.yaml
```
#修改镜像地址：修改为国内镜像（Linux以及全局代理加速了可以不用修改）
registry: registry.cn-hangzhou.aliyuncs.com
image: google_containers/nginx-ingress-controller
image: google_containers/kube-webhook-certgen
tag: v1.3.0
--------------------------------------------------
#修改配置为以下内容：
hostNetwork: true 

#将 Pod 的网络模式设置为使用宿主机的网络栈，而不是 Kubernetes 默认的虚拟网络

dnsPolicy: ClusterFirstWithHostNet 

#为了使Pod能够监听宿主机的IP和端口需要 hostNetwork: true ，但这时候默认的 DNS 策略（ClusterFirst）会失效，Pod会使用宿主机DNS 配置使得不能解析 Kubernetes 集群内部的 Service。
ClusterFirstWithHostNet是对 hostNetwork: true的补充，会强制 Pod 使用 Kubernetes 集群的 DNS 服务（如 CoreDNS），这样 Pod 使用 hostNetwork: true 时就仍然能够解析 Kubernetes 集群内部的 Service。


 kind: DaemonSet #找不到DaemonSet就将Deployment修改为DaemonSet
    nodeSelector:
            ingress: "true" # 增加选择器，如果 node 上有 ingress=true 就部署

service:
	type:ClusterIP
#将 service 中的 type 由 LoadBalancer 修改为 ClusterIP，
如果服务器是云平台才用 LoadBalancer

admissionWebhooks:
	enabled: false
#将 admissionWebhooks.enabled 修改为 false
```
### 5.为 ingress 专门创建一个 namespace
`kubectl create ns ingress-nginx`  
### 6.安装
```
# 为需要部署 ingress 的节点上加标签(如果在maser上会有污点，需要先去污点）
$kubectl label node <node-name> ingress=true  

# 安装 ingress-nginx
#进入ingress-nginx文件夹执行命令
$helm install ingress-nginx ./ -n ingress-nginx

#命令解析：
#helm install：Helm 的安装命令。
#ingress-nginx：安装的 Release 名称（可以自定义）。
#./：Chart 的路径，表示当前目录下
#-n ingress-nginx：指定安装到的 Kubernetes 命名空间（Namespace）为 ingress-nginx。
``````
