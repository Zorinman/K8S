[ingress解释](https://github.com/Zorinman/K8S/blob/main/k8s%E5%9F%BA%E7%A1%80/%E6%9C%AF%E8%AF%AD%E8%A7%A3%E6%9E%90/Ingress.md)  
下面prometheus-ingress.yaml关联的三个Service即使Type为ClusterIP也不影响外部访问，因为会通过ingress的控制器作为流量入口再从集群内发现对应Service

--------
⭐⭐这里ingress-nginx控制器的Service的Type为ClusterIP外部也依然能够访问,这是因为在之前部署Ingress时在value.yaml文件中定义了hostNetwork: true
```
当 ingress-nginx-controller 的 Pod 配置了 hostNetwork: true 时：

Pod 会直接使用宿主机（Pod部署在的那个节点）的网络栈，而不是 Kubernetes 的虚拟网络。

Pod 会监听宿主机的 IP 地址和端口,而不是 Kubernetes 分配的 Pod IP

外部网络可以通过节点的 IP 地址访问服务
```
这里ingress-nginx-controller 部署在节点 node1 上（Service和Pod以及DaemonSet），IP 地址为 192.168.219.148。 Pod 配置了 hostNetwork: true，它会监听 192.168.219.148:80 和 192.168.219.148:443。  

------
**1 提前安装Ingress-nginx**

 
 
**2 下载kube-prometheus**  
下载kube-prometheus:https://github.com/prometheus-operator/kube-prometheus
注意查看K8S版本然后查看对应可用的kube-prometheus版本
`unzip <压缩名称>`


**3 进入manifests目录中创建prometheus-ingress.yaml文件，为之后使用ingress反向代理提供流量转发转发规则，使得访问能到达对应的内部Service**
```yaml
# 创建 prometheus-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: monitoring
  name: prometheus-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.wolfcode.cn  # 访问 Grafana 域名流量转发到对应服务prometheus-k8s 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
  - host: prometheus.wolfcode.cn  # 访问 Prometheus 域名流量转发到对应服务prometheus-k8s 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-k8s 
            port:
              number: 9090
  - host: alertmanager.zorin.cn  # 外部访问 alertmanager 域名流量转发到对应服务alertmanager-main
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: alertmanager-main
            port:
              number: 9093

```




**4.以 Server-Side Apply 方式应用 manifests/setup 目录中的资源**  
`kubectl apply --server-side -f setup`

等待所有 CRD 资源 变为 Established 状态，确保它们被 Kubernetes 正确注册  
```commandline
kubectl wait \  
	--for condition=Established \  
	--all CustomResourceDefinition \
	--namespace=monitoring`  
```



部署 manifests目录下的 的其他组件  
`kubectl apply -f manifests/`

**安装之后所有资源都在命名空间monitoring下运行**




**5 在物理机以下目录配置host文件：C:\Windows\System32\drivers\etc**  
配置host之后，浏览器访问相应域名就会解析到对应IP地址，访问流量从ingress进行集群  
添加以下内容(192.168.219.148为ingress-nginx所在的节点，可用通过ingress-nginx的pod查看)  
```commandline
192.168.219.148 grafana.zorin.cn
192.168.219.148 prometheus.zorin.cn
192.168.219.148 alertmanager.zorin.cn
```
**6.通过浏览器访问上面对应域名即可(记得关梯子）**  
grafana的默认账号和密码都是admin
