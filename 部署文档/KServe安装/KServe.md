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

## 安装流程概览与对比

无论是标准模式还是 Serverless 模式，KServe 的安装都遵循相似的逻辑路径，但在网络层和核心依赖上有所分叉。以下是通用化后的安装流程总结：

### 1. 基础环境准备 (Common Foundation)
所有模式均需要的通用步骤：
*   **Kubernetes Cluster**: 确保集群版本符合要求 (v1.32+)。
*   **Cert Manager**: 安装 Cert Manager (v1.15.0+) 用于自动化管理 Webhook 证书，这是生产环境的基础安全组件。

### 2. 分歧路径 (Divergent Paths)

根据您的业务需求（生成式 AI vs 预测性 AI），安装流程在此分为两条路径：

#### 路径 A：Standard Kubernetes Deployment (推荐用于 LLM/GenAI)
*专注于简化架构与长连接性能*

1.  **安装网络控制器 (Network Controller)**
    *   **核心动作**：部署 **Gateway API** CRDs。
    *   **实现选择**：安装 **Envoy Gateway** 或 **Istio** 作为 Gateway 的具体实现。
    *   **配置**：创建 `GatewayClass` 和 `Gateway` 资源以暴露服务端口。
2.  **安装 KServe (Standard Mode)**
    *   **关键配置**：在 Helm Chart 或安装参数中，显式设置 `deploymentMode` 为 `Standard`。
    *   **网关启用**：开启 `enableGatewayApi` 选项。

#### 路径 B：Knative Deployment (推荐用于传统预测模型)
*专注于资源利用率与弹性伸缩*

1.  **安装 Knative Serving**
    *   **核心动作**：安装 Knative Serving 核心组件与 CRDs。
2.  **安装网络层 (Networking Layer)**
    *   **核心动作**：安装 **Istio** (推荐) 作为 Knative 的网络层网关。
    *   **配置**：配置 Knative使用 Istio 进行流量路由。
3.  **安装 KServe (Serverless Mode)**
    *   **关键配置**：保持默认配置安装即可（KServe 默认为 Serverless 模式）。
    *   **模型运行**：通常配合 `ClusterServingRuntimes` 一起安装。

