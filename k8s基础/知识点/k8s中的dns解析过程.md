从pod中发出请求到外部DNS服务器 大体经过三层DNS解析：k8s的CoreDns解析→宿主机的DNS网关解析→外部DNS服务器

### CoreDNS、宿主机 DNS 与外部 DNS 的分工

| 组件                | 作用范围                  | 核心职责                                                                 |
|---------------------|--------------------------|--------------------------------------------------------------------------|
| **CoreDNS**         | Kubernetes 集群内部       | - 解析集群内服务域名（如 `service.namespace.svc.cluster.local`）<br>- 缓存 DNS 查询结果<br>- 通过 `forward` 插件将外部域名请求转发至宿主机 DNS |
| **宿主机 DNS**      | 节点（宿主机）及容器网络  | - 处理非集群域名查询（如内网域名 `company.local`）<br>- 将外部域名请求转发至更上游的公共 DNS 服务器 |
| **外部 DNS 服务器** | 互联网                    | - 解析公共域名（如 `google.com`）<br>- 提供递归查询服务（如 `8.8.8.8`、`1.1.1.1`） |


如果pod不能访问外部网络，但是在容器外部却可以随意访问，那么首先考虑CoreDNS失效，可以查看pod日志文件    

**CoreDNS失效：**   
如果Pod 容器内的文件/etc/resolv.conf指向 CoreDNS（如 nameserver 10.96.0.10） 由于pod请求先经过CoreDNS进行处理转发，因此请求不能到达外部无服务器  

---------

宿主机的DNS配置通常在/etc/sysconfig/network-scripts/ifcfg-ens33，也可在/etc/resolv.conf进行查看（注意是容器外）  
CoreDNS以pod形式部署在kube-sysytem命名空间  
