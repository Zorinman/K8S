[Ingress安装](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/ingress-nginx%E5%AE%89%E8%A3%85.md)  

Ingress 提供从集群外部到集群内服务的 HTTP 和 HTTPS 路由，流量Ingress-controller根据Ingress的yaml 资源所定义的路由规则来找到对应集群内的服务  

Ingress的安装指的是前者，而 Ingress的yaml 资源通常根据对应的服务来创建，且**Ingress资源必须跟要关联的服务在同一命名空间下**  

**1安装部署的Ingress**  
![image](https://github.com/user-attachments/assets/f1ed0c61-3606-4b07-9887-35c2546232f3)  

**2为对应服务定义的Ingress资源作为路由规则**  
![image](https://github.com/user-attachments/assets/d9678c5d-18bb-40da-af63-47a258f05e3a)  

-----
![image](https://github.com/user-attachments/assets/cf1971b8-3ed1-4daf-b509-2583df5f54db)

如图当集群外部客户端像访问集群内服务时，外部流量从入口Ingress-controller进入，Ingress-controller会根据定义的Ingress资源（通过yaml定义)的路由规则来找到对应的服务及Pod  

**Ingress资源模板(用于关联到prometheus的服务）**"
```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: monitoring
  name: prometheus-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.zorin.cn  # 访问 Grafana 域名流量转发到对应服务prometheus-k8s 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
  - host: prometheus.zorin.cn  # 访问 Prometheus 域名流量转发到对应服务prometheus-k8s 
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


