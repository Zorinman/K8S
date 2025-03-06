

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

#当 hostNetwork: true 时，默认的 DNS 策略（ClusterFirst）会失效，因为 Pod 会直接使用宿主机的 DNS 配置。
ClusterFirstWithHostNet 会强制 Pod 使用 Kubernetes 集群的 DNS 服务（如 CoreDNS），而不是宿主机的 DNS。


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
