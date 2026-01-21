https://blog.csdn.net/gitblog_00811/article/details/154934399
https://kserve.github.io/website/docs/intro
**DEMO ：**
[Guideline](https://devopscube.com/deploy-ml-model-kubernetes-kserve/)  
[Github Repo](https://github.com/devopscube/predictor-model)
# 架构图
![alt text](imgs/image.png)

## KServe 部署模式特性对比

| 特性           | **Knative Deployment** (Serverless Mode)                              | **Standard Deployment** (Raw Mode)                                              |
| :------------- | :-------------------------------------------------------------------- | :------------------------------------------------------------------------------ |
| **核心机制**   | 基于 Knative 的 Serverless 架构                                       | 基于 Kubernetes 原生资源 (Deployment)                                           |
| **弹性伸缩**   | **Scale-to-Zero (缩容至零)** <br> 基于请求数 (Request-based) 自动扩缩 | **手动或 HPA** <br> 直接资源控制 (Direct Resource Control)                      |
| **适用场景**   | **Predictive AI (预测性 AI)** <br> 突发流量、低频调用、传统小模型     | **Generative AI (生成式 AI / LLM)** <br> 稳定流量、长连接、流式响应 (Streaming) |
| **依赖复杂度** | **高** (需维护 Knative Serving 全套组件)                              | **低** (无 Knative 依赖，更轻量)                                                |
| **网络层**     | 通常结合 Istio / Kourier                                              | 推荐使用 **Gateway API** (Envoy/Istio)                                          |

## 核心组件依赖概览 (Component Requirement Matrix)

下表清晰展示了两种部署模式下各组件的**共用性**与**差异性**：

| 组件名称 (Component)     | 功能简述 (Role)    | Standard Mode (Raw)         | Serverless Mode (Knative)     | 依赖类型            |
| :----------------------- | :----------------- | :-------------------------- | :---------------------------- | :------------------ |
| **Kubernetes Cluster**   | 基础设施           | ✅ **必须**                  | ✅ **必须**                    | 🔵 **共用 (Shared)** |
| **Cert Manager**         | 证书/Webhook 安全  | ✅ **必须**                  | ✅ **必须**                    | 🔵 **共用 (Shared)** |
| **KServe Control Plane** | 控制器与核心逻辑   | ✅ **必须** (Mode: Standard) | ✅ **必须** (Mode: Serverless) | 🔶 **差异 (Unique)** |
| **Gateway API CRDs**     | 新一代流量路由标准 | ✅ **必须**                  | ❌ (通常不需要)                | 🔶 **差异 (Unique)** |
| **Knative Serving**      | Serverless 引擎    | ❌ (不需要)                  | ✅ **核心组件**                | 🔶 **差异 (Unique)** |
| **Istio** (用户当前选择) | 网络/流量网关      | ✅ **作为 Gateway 实现**     | ✅ **作为 Knative 网关**       | 🔵 **共用 (Shared)** |

> **注:** 
> *   🔵 **共用**: 无论哪种模式都必须安装的基础组件。
> *   🔶 **差异**: 决定了是走“轻量级标准模式”还是“全功能 Serverless 模式”的关键分歧点。

### 组件协作关系与流量拓扑 (Component Roles & Flows)

为便于理解，以下将各组件比作一个协作团队：

| 组件             | 角色定位                  | 核心职责与协作关系                                                                                                                                      |
| :--------------- | :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Kubernetes**   | **基座 (OS)**             | 提供容器运行的底层资源。                                                                                                                                |
| **Cert Manager** | **安保 (Security)**       | 自动签发证书，保障 Webhook 通信加密。KServe 强依赖它来验证自身组件的合法性。                                                                            |
| **Gateway API**  | **交通规则 (Standard)**   | **定义接口 (YAML)**。它不干活，只制定规则。它解耦了 KServe (上层) 与 Istio (下层)，让 KServe 只需生成 Gateway 资源，而无需关心底层是 Istio 还是 Envoy。 |
| **Istio**        | **交警/大门 (Ingress)**   | **执行动作 (Data Plane)**。它监听 Gateway API 的规则，负责真正的流量拦截、路由转发及 mTLS 加密。**它是 Gateway API 的“肉身”。**                         |
| **KServe**       | **指挥官 (Orchestrator)** | **定义服务**。它负责抽象模型（InferenceService），并自动生成底层的 K8s Deployment 和 Gateway 路由规则，指挥 Istio 如何转发流量。                        |
| **Knative**      | **调度员 (Autoscaler)**   | **(仅 Serverless)** 接管 KServe 的部署请求。它在 Istio 和 Pod 之间充当“中间人”，拦截流量以判断是否需要扩容（0->1）或缩容（1->0）。                      |

#### 模式 A: Standard Mode 
**特征**: 架构扁平，适合长连接 (LLM)。
> **流量流向**: `用户` -> `Istio (Gateway)` -> `Model Pod (由 KServe 直接管理)`
*   **协作**: KServe 定义服务 -> 通过 Gateway API 告知 Istio -> Istio 直接将流量导向模型 Pod。

#### 模式 B: Serverless Mode 
**特征**: 链路较长，支持缩容至零。
> **流量流向**: `用户` -> `Istio (Gateway)` -> `Knative (Activator)` -> `Model Pod`
*   **协作**: KServe 委托 Knative 管理 -> Knative 负责拉起/销毁 Pod -> Istio 负责将流量路由给 Knative 或 Pod。

## 安装流程概览与对比

无论是标准模式还是 Serverless 模式，KServe 的安装都遵循相似的逻辑路径，但在网络层和核心依赖上有所分叉。以下是通用化后的安装流程总结：

### 1. 基础环境准备 (Common Foundation)
所有模式均需要的通用步骤：
*   **Kubernetes Cluster**: 确保集群版本符合要求 (v1.32+)。
*   **Cert Manager**: 安装 Cert Manager (v1.15.0+) 用于自动化管理 Webhook 证书，这是生产环境的基础安全组件。
    > **原理解析**: 它的核心作用是在 Kubernetes 集群内部自动签发和轮转证书（如 TLS 证书）。在 KServe 中，它主要用于保障 Admission Webhook 的安全通信，确保组件间的交互是加密且受信任的。
*   **Istio**：
    > 官方文档中省略了具体安装步骤，以下是 **Istio** 的安装简述：
    
    **Istio 简介**: 
    Istio 是一个开源的服务网格，提供流量管理、安全性及可观测性。在 KServe Standard 模式中，它主要作为 **Gateway API 的数据面实现**，负责真正的流量接收与路由。
    
    **安装步骤**:
     >只需在master节点安装，K8s 会自动调度 istio的Pod。
    1. **下载安装工具到本地**: 在master节点下载 `istioctl`。
    ```bash
    curl -L https://istio.io/downloadIstio | sh -
    ```
    2. **进入下载到本地的文件包中执行安装**:
       ```bash
       cd <folder name>
       ```
        ```bash
        istioctl install --set profile=default -y
        ```
       此命令会在集群中安装 `istiod` (控制平面) 和 `istio-ingressgateway` (网关)两个Pod
  
### 2. 分歧路径 (Divergent Paths)

根据您的业务需求（生成式 AI vs 预测性 AI），安装流程在此分为两条路径：

#### 路径 A：Standard Kubernetes Deployment (推荐用于 LLM/GenAI)
*专注于简化架构与长连接性能*

1.  **安装网络控制器 (Network Controller)**
    *   **核心动作**：部署 **Gateway API** CRDs。
        > **原理解析**: 这是 Kubernetes 新一代的网络流量管理标准，旨在替代传统的 Ingress。它提供了更丰富的流量路由能力（基于 header、权重等）。在 KServe Standard 模式下，它负责将外部推理请求通过标准化的接口转发给后端的模型服务。
    *   **实现选择（需要安装，官方文档省略了）**：安装 **Istio（我使用的）**或**Envoy Gateway** 作为 Gateway 的具体实现。
        > **原理解析**: 只有 API 定义是不够的，需要具体的实现者。Envoy Gateway 或 Istio 充当“数据面”，真正处理进出的网络流量。它们负责负载均衡、请求验证及将流量导向正确的模型 Pod。Envoy 以高性能著称，适合高并发的 LLM 推理场景。
    *   **配置**：创建 `GatewayClass` 和 `Gateway` 资源以暴露服务端口。
2.  **安装 KServe (Standard Mode)**
    *   **关键配置**：在 Helm Chart 或安装参数中，显式设置 `deploymentMode` 为 `RawDeployment`。
    *   **网关启用**：开启 `enableGatewayApi` 选项。

#### 路径 B：Knative Deployment (推荐用于传统预测模型)
*专注于资源利用率与弹性伸缩*

1.  **安装 Knative Serving**
    *   **核心动作**：安装 Knative Serving 核心组件与 CRDs。
        > **原理解析**: 这是 Serverless 架构的核心引擎。它负责管理无服务器工作负载的生命周期，包括自动伸缩（从 0 到 N 再回到 0）、版本控制（Revisions）以及流量切分。它让模型服务只需在有请求时才消耗资源。
2.  **安装网络层 (Networking Layer)**
    *   **核心动作**：安装 **Istio** (推荐) 作为 Knative 的网络层网关。
        > **原理解析**: 在 Knative 模式下，Istio 作为强大的 Ingress 网关，不仅处理南北向流量（进出集群），还支持复杂的流量治理（如金丝雀发布、熔断）。KServe 依赖它来实现推理请求的精确路由。
    *   **配置**：配置 Knative使用 Istio 进行流量路由。
3.  **安装 KServe (Serverless Mode)**
    *   **关键配置**：保持默认配置安装即可（KServe 默认为 Serverless 模式）。
    *   **模型运行**：通常配合 `ClusterServingRuntimes` 一起安装。
        > **原理解析**: 定义了特定机器学习框架（如 PyTorch, TensorFlow, vLLM）运行所需的容器镜像和启动命令模板。它让 KServe 知道如何启动一个具体的模型服务。





# 实操
## 路径A部署：Standard Kubernetes Deployment

### 安装 Cert Manager (v1.15.0+):
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.2/cert-manager.yaml

```
### 安装Istio

  > 官方文档中省略了具体安装步骤，以下是 **Istio** 的安装简述：
  **安装步骤**:
    >只需在master节点安装，K8s 会自动调度 istio的Pod。
  1. **下载安装工具到本地**: 在master节点下载 `istioctl`。
  ```bash
  curl -L https://istio.io/downloadIstio | sh -
  ```
  2. **进入下载到本地的文件包中执行安装**:
      ```bash
      cd <folder name>
      ```
      ```bash
      istioctl install --set profile=default -y
      ```
      此命令会在集群中安装 `istiod` (控制平面) 和 `istio-ingressgateway` (网关)两个Pod
### 安装网络控制器
#### 部署 **Gateway API** CRDs
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/standard-install.yaml
```
#### 创建 `GatewayClass` 和 `Gateway` Yaml资源以暴露服务端口(使用的是istio)
**建议先创建命名空间（如果尚未创建）**
```bash
kubectl create namespace kserve
```
**创建yaml：**
```yaml
touch GatewayClass-istio.yaml
touch Gateway.yaml
```
**GatewayClass：**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: istio
spec:
  controllerName: istio.io/gateway-controller
```
**Gateway**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: kserve-ingress-gateway
  namespace: kserve
spec:
  gatewayClassName: istio  # 这里指向上面定义的 Istio GatewayClass
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
  - name: https  # 如果不需要 HTTPS 可以移除
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: kserve-ingress-gateway-certs # 确保您有相应的证书 Secret
        namespace: kserve
    allowedRoutes:
      namespaces:
        from: All
```
**应用:**
```bash
kubectl apply -f GatewayClass-istio.yaml 
kubectl apply -f Gateway.yaml 
```
### 安装 KServe (Standard Mode-使用helm)
**安装 KServe CRD**
```bash
helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd --version v0.15.0
```
**安装 KServe 核心组件**
 在 KServe v0.15.0 中，使用Standard模式设置`deploymentMode=RawDeployment`即可
```bash
helm install kserve oci://ghcr.io/kserve/charts/kserve --version v0.15.0 \
  --namespace kserve \
  --create-namespace \
  --set kserve.controller.deploymentMode=RawDeployment \
  --set kserve.controller.gateway.ingressGateway.enableGatewayApi=true \
  --set kserve.controller.gateway.ingressGateway.kserveGateway=kserve/kserve-ingress-gateway
```

**查看pods是否成功启动**
```bash
kubectl get pods -n kserve
```