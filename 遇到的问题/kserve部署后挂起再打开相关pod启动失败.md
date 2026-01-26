# KServe 部署故障复盘与解决方案日志

## 1. 故障现象 (The Problem)
**环境背景：**

- **OS:** Debian 13 (Running on VMware)
- **Cluster:** Kubernetes v1.34.3
- **CNI:** Calico
- **Component:** KServe v0.15.0 (Standard/RawDeployment Mode)
- **触发场景：** 虚拟机成功部署后挂起（Suspend）4天，唤醒后出现故障。
**具体表现：**

```plaintext
Warning  FailedCreatePodSandBox  ...  kubelet
Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "...": 
plugin type="calico" failed (add): error getting ClusterInformation: connection is unauthorized: Unauthorized

```

---

## 2. 根本原因 (The Root Cause)
**核心原因：虚拟机挂起导致的时间与认证凭证失效 (Time Skew & Token Expiration)**

1. **时间冻结效应：**
虚拟机挂起期间，内部系统时钟停止。外部世界（Kubernetes API Server 的逻辑时钟）已过了4天，但虚拟机内部认为时间并未流逝。这导致了严重的时间不同步。
2. **CNI 凭证过期：**
  - Calico CNI 插件在宿主机磁盘上（`/etc/cni/net.d/`）保存了用于连接 K8s API Server 的配置文件和 Token。
  - 由于长时间的挂起，这些**静态保存的 Token 已经超过了 Kubernetes API Server 认可的生命周期（TTL）**，或者服务端因长时间无心跳已断开了连接会话。
3. **死锁循环：**
  - Kubelet 尝试启动 KServe Pod -> 调用磁盘上的 Calico 插件 -> 插件使用过期 Token 访问 API -> API 返回 `Unauthorized` -> 网络创建失败 -> Pod 卡在 `ContainerCreating`。
  - 由于网络插件无法工作，`Terminating` 的 Pod 也无法清理其网络资源，导致卡在删除过程中。

---

## 3. 解决方案 (The Solution)
按以下顺序执行操作，成功恢复了集群状态。

### 步骤一：清理“僵尸” Pod
强制删除那些因网络层故障而无法正常终结的 Pod，清除脏数据。

```bash
# 清理 Cert-Manager
kubectl get pods -n cert-manager | grep -E 'Terminating|ContainerCreating' | awk '{print $1}' | xargs kubectl delete pod -n cert-manager --force --grace-period=0

# 清理 KServe
kubectl get pods -n kserve | grep -E 'Terminating|ContainerCreating' | awk '{print $1}' | xargs kubectl delete pod -n kserve --force --grace-period=0

# 清理 Istio
kubectl get pods -n istio-system | grep -E 'Terminating|ContainerCreating' | awk '{print $1}' | xargs kubectl delete pod -n istio-system --force --grace-period=0

```

### 步骤二：强制刷新 CNI 凭证 (关键修复)
重启 Calico 的 DaemonSet。这会触发 Calico 初始化容器重新运行，向 API Server 申请**新的、有效的 Token** 并覆盖宿主机上的旧配置文件。

```bash
# 在 Master 节点执行
kubectl rollout restart daemonset calico-node -n kube-system

```
*执行后等待约 30-60 秒，确保 kube-system 下所有的 calico-node Pod 变为 Running 状态。*

### 步骤三：验证恢复
观察 KServe 相关的 Pod，确认它们能够成功分配网络 IP 并启动。

```bash
kubectl get pods -n kserve -w

```
*结果：Cert-Manager 成功签发证书，KServe Controller 成功启动，服务恢复正常。*

---

## 4. 后续避坑指南 (Best Practices)
在虚拟机环境下运行 Kubernetes，若需长时间挂起/休眠，建议执行以下操作以避免类似问题：

- **1.恢复后第一时间同步时间**： Linux 虚拟机唤醒后，时间往往不会立刻同步。这会导致证书校验失败（比如现在是 2026 年，虚拟机醒来以为是 2022 年，证书就会被认为“尚未生效”）
```bash
# 示例 (根据具体时间同步工具调整)
systemctl restart systemd-timesyncd
# 或
ntpdate pool.ntp.org

```

- **2.主动重启网络组件**： 挂起数天后，TCP 连接肯定断了。不要等它报错，醒来后直接刷新一下关键组件：
```bash
- # 刷新节点状态
systemctl restart kubelet
# 刷新网络插件 (可选，如果发现 Pending 再做)
kubectl rollout restart ds calico-node -n kube-system
```


# 问：为什么其它pod没有受到影响？ 只有kserve相关pod被影响了
# 答
简单来说：**“活着的（Running）”Pod 不需要再通过安检，只有“刚出生（Pending）”的 Pod 才会被拦在门外。**
下面是详细的各种原因分析：

## 1. 核心机制：CNI 插件只在“创建时”工作
你遇到的报错 `plugin type="calico" failed (add)`，关键词是 **(add)**。

- **CNI 的工作时机：** Kubernetes 的网络插件（CNI）**不是**每时每刻都在工作的。它只在两个时间点被 Kubelet 调用：
  1. **Pod 创建时 (ADD)：** 为 Pod 分配 IP，创建虚拟网卡，设置路由。
  2. **Pod 销毁时 (DEL)：** 回收 IP，拆除虚拟网卡。
- **存量 Pod (Other Pods)：** 那些在你挂起虚拟机之前就已经处于 `Running` 状态，并且在挂起期间没有崩溃的 Pod，它们的网络环境（虚拟网线、IP地址、路由表）在4天前就已经配置好了。只要宿主机不重启，Linux 内核就会一直维持这些网络连接。因此，它们**不需要**再次调用 CNI 插件，也就不会触发 Token 认证，当然就不会受影响。

## 2. 为什么 KServe/Cert-Manager 会“撞在枪口上”？
虽然其他业务 Pod 可能没挂，但 KServe 及其依赖组件（Cert-Manager, Istio）往往比较“敏感”或“活跃”，导致它们在唤醒后**发生了重启**，从而触发了上述的 CNI 调用流程：

- **Cert-Manager 的敏感性：**
Cert-Manager 负责证书轮换。虚拟机挂起4天，唤醒后时间发生跳变，Cert-Manager 内部的某些定时器（Timer）或证书检查逻辑可能会因为时间偏差判定“状态异常”或“超时”，导致进程退出并尝试重启。一旦它尝试重启，就必须走“Pod 创建 -> 调用 CNI”的流程，于是就挂了。
- **Liveness Probe（存活探针）超时：**
KServe 和 Istio 的组件通常配置了健康检查（Liveness Probe）。虚拟机唤醒瞬间，系统负载极高（因为所有进程都在抢 CPU 追赶时间），导致这些组件无法在规定时间内响应 Kubelet 的健康检查。Kubelet 判定它们“死了”，于是杀掉旧 Pod，试图拉起新 Pod —— **只要一拉新 Pod，就会撞上 CNI 认证失败的墙。**
- **依赖链崩塌：**
KServe 依赖 Istio 和 Cert-Manager。如果 Cert-Manager 的 Webhook Pod 挂了（处于 Terminating/ContainerCreating），KServe 的控制器可能因为连不上 Webhook 而报错退出，触发重启循环。