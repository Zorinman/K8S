**参考**:https://blog.csdn.net/qq_40189664/article/details/147150255

### 以下所有操作只需要在master节点上完成即可

**1.查看证书看是否过期**
` kubeadm certs check-expiration`
**2、备份证书(可选)**
cp -rp /etc/kubernetes /etc/kubernetes.bak

**3.更新证书文件**
`kubeadm certs renew all`

----------

这个命令会更新`/etc/kubernetes/pki/`下所有的证书文件延期一年
（**不会更新/var/lib/kubelet/pki/下的证书**，kubelet/pki的证书文件在 kubelet 完成 TLS bootstrapping 后自动生成，此证书是由 controller manager 签署的）

- `/etc/kubernetes/pki/`: 主要存储控制平面组件（API Server、Controller Manager、Scheduler 等）的证书，包含了集群的根 CA 证书、API Server 证书、控制平面组件证书等。

- `/var/lib/kubelet/pki/`: 主要存储 kubelet 相关的证书，用于与控制平面 API Server 进行通信。此目录的证书与集群节点的身份验证相关。

**4.再次查看证书**
` kubeadm certs check-expiration`
发现天数增加了

**5.初始化所有配置文件**
`kubeadm init phase kubeconfig all`

这个命令会自动重新初始化集群的以下文件，以便重新加载证书内容：

- /etc/kubernetes/admin.conf：管理员的 Kubeconfig 配置文件。

- /etc/kubernetes/controller-manager.conf：控制器管理器的 Kubeconfig 文件。

- /etc/kubernetes/scheduler.conf：调度器的 Kubeconfig 文件。

- /etc/kubernetes/kubelet.conf：Kubelet 的 Kubeconfig 文件。


**6.重启节点**
`reroot` 或者手动重启


### 附
- 如果从节点之前也需要使用kubectl命令操控集群 则需要把完成以上操作之后`master`节点上的`/etc/kubernetes/admin.conf`文件拷贝到从节点存放旧admin.conf的对应位置并覆盖


- 查看kubelet/pki的证书期限
`openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -text | grep Not`
