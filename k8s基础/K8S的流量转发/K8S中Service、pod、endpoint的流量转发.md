## Endpoint、Service、pod的关系
流量过程  kubectl命令 →apiserver→service→Endpoint→通过iptable找到对应节点的kube-proxy→该节点上对应的Pod以及容器如图
![alt text](图片\image.png)

每个Service有一个对应的Endpoint 该Endpoint管理着对应Service所关联的所有节点Pod的ip地址以及Targetport（ Pod 中容器的监听端口）    


## 集群内部Pod互相访问的流量传输：
### 背景前提
所有 Pod 都在同一个集群内。

每个 Pod 都有一个唯一的 IP（由 CNI 插件分配）。

集群内部网络是扁平的（即 Pod IP 是可以互通的）。


![alt text](图片\image1.png)

具体解释如下：

### Pod 之间通信的传输过程


 **Pod A 发起请求**
Pod A 里的应用通过某种方式（例如访问 IP 或 DNS 名称）访问另一个 Pod B，例如 curl http://10.1.2.3:8080 或 curl http://svc-name.default.svc.cluster.local。


**流量路由选择路径**
(1)直接访问 Pod IP：

流量直接发送到 Pod B 的 IP（比如 10.1.2.3:8080）。

路由表由 CNI 插件管理（例如 Calico、Flannel、Cilium），负责转发到目标 Pod 的网络命名空间。

如果跨 Node，会经过节点之间的 VXLAN、BGP 或其他封装机制（由 CNI 实现）。

(2)通过 Service 访问：

流量首先被发往 Service 的 ClusterIP。

iptables 或 IPVS（由 kube-proxy 管理）将流量转发到后端 Pod（Pod B）。

kube-proxy 在每个 Node 上维护转发规则，实现负载均衡。

数据包被转发到某个符合 selector 的 Pod IP。

4. 网络传输
如果两个 Pod 在同一 Node 上：

流量通过虚拟网卡、veth pair、本地 bridge 网络进行转发。

如果两个 Pod 在不同 Node 上：

流量会先到 Node 的网络设备，然后经由 CNI 插件配置的隧道（VXLAN/GRE 等）或 BGP 等方式转发到目标 Node。

目标 Node 接收后，再转发到对应的 Pod。

5. 响应返回
Pod B 的应用处理请求后将响应数据通过同样的网络路径返回给 Pod A。


## 通过Ingress实现外部访问内部集群的pod

[通过Ingress实现外部访问内部集群的pod](../知识点/Ingress.md)