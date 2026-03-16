## 高层次架构文档：模型代理网关（基于设计模式的可扩展架构）

### 一、设计原则

1. 模块数量保持精简，每个模块职责单一。
2. 通过**设计模式进行抽象扩展**（Adapter、Strategy 等），而不是运行时插件或热插拔机制。
3. 内部统一使用 **Canonical 协议模型** 作为系统内部数据交换结构。
4. 厂商差异、协议差异、计费差异均通过抽象接口 + 多实现方式隔离。
5. **用量统计与计费**、**可观测指标**均以旁路事件方式输出，不参与主流程控制，不影响主流程延迟。
6. 系统不假定任何持久化存储能力，所有统计数据均以事件形式抛出，由下游系统决定如何消费。

---

### 二、核心模块

系统仅包含六个核心模块：

1. Edge Gateway
2. Protocol Adapter
3. Router
4. Provider Adapter
5. Usage & Billing Event Emitter（旁路）
6. Observability Event Emitter（旁路）

模块之间通过明确的数据模型与接口进行协作。

---

### 三、模块职责

#### 1. Edge Gateway

职责：

* 接收客户端请求
* 执行鉴权与限流
* 统一生成 request_id
* 记录基础请求上下文

输出：

CanonicalRequest

Edge Gateway 不关心模型厂商和具体协议，仅负责入口控制。

#### 2. Protocol Adapter

职责：

将客户端协议转换为内部 Canonical 数据结构，并将内部响应转换为客户端协议。

不同客户端协议通过 **Adapter 模式** 实现。

示例实现：

* OpenAICompatibleAdapter
* AnthropicMessagesAdapter
* InternalSDKAdapter

系统内部只处理 CanonicalRequest / CanonicalResponse。

#### 3. Router

职责：

* 模型 ID 映射
* 上游厂商选择
* 流量分流

Router 通过 **Strategy 模式** 支持不同路由策略。

可能的策略实现：

* WeightedRoutingStrategy
* FailoverRoutingStrategy
* LatencyBasedRoutingStrategy
* CostBasedRoutingStrategy

Router 输出：

ProviderCandidate

包含：

* provider
* provider_model

#### 4. Provider Adapter

职责：

将 CanonicalRequest 转换为具体厂商请求，并将厂商响应转换为 ProviderResponse。

不同厂商通过 **Adapter 模式** 实现。

示例实现：

* OpenAIProviderAdapter
* AnthropicProviderAdapter
* GeminiProviderAdapter
* CustomProviderAdapter

Provider Adapter 负责：

* 请求构造
* 协议转换
* 流式响应处理
* 返回 ProviderResponse

#### 5. Usage & Billing Event Emitter（旁路）

职责：

* 从 ProviderResponse 中提取用量
* 根据用量计算成本
* 将结果作为事件输出

该模块**不参与主流程控制**，仅订阅主流程产生的数据并生成事件。

系统通过两类抽象进行扩展：

**UsageExtractor（Strategy）**

不同厂商解析不同字段。

示例：

* OpenAIUsageExtractor
* AnthropicUsageExtractor
* GeminiUsageExtractor

**PricingStrategy（Strategy）**

根据 Usage 计算成本。

示例：

* TokenPricingStrategy
* ImagePricingStrategy
* EmbeddingPricingStrategy

输出事件：

UsageEvent
BillingEvent

#### 6. Observability Event Emitter（旁路）

职责：

收集请求生命周期中的关键指标，并以事件形式输出。

该模块同样为旁路模块，不影响主流程。

生成事件：

ObservabilityEvent

---

### 四、Canonical 数据模型

#### CanonicalRequest

```
{
  "request_id": "...",
  "client": "sdk|http|agent",
  "model": "gemini3-pro",
  "modality": "text|image|video|audio|embedding",
  "input": {...},
  "params": {...},
  "stream": true|false,
  "metadata": {...}
}
```

#### CanonicalResponse

```
{
  "request_id":"...",
  "outputs":[...],
  "finish_reason":"...",
  "provider_raw": {...}
}
```

---

### 五、用量事件结构

Usage 统一抽象结构：

```
{
  "input_tokens": 0,
  "output_tokens": 0,
  "cache_hit_tokens": 0,
  "cache_write_tokens": 0,
  "image_count": 0,
  "image_sizes": {"1024x1024": 2},
  "embedding_count": 0
}
```

UsageEvent：

```
{
  "request_id": "...",
  "provider": "...",
  "model": "...",
  "usage": {...}
}
```

BillingEvent：

```
{
  "request_id": "...",
  "provider": "...",
  "model": "...",
  "cost": 0
}
```

---

### 六、可观测事件结构

ObservabilityEvent：

```
{
  "request_id":"...",
  "model":"...",
  "provider":"...",
  "client":"...",
  "modality":"...",
  "stream":true,
  "latency_total_ms":0,
  "ttfb_ms":0,
  "request_bytes":0,
  "response_bytes":0,
  "error_code":null
}
```

---

### 七、主流程数据流

```
Client Request
     │
     ▼
Edge Gateway
     │
     ▼
Protocol Adapter
     │
     ▼
CanonicalRequest
     │
     ▼
Router (model alias + routing strategy)
     │
     ▼
Provider Adapter
     │
     ▼
CanonicalResponse
     │
     ▼
Protocol Adapter
     │
     ▼
Client Response
```

旁路流程：

```
ProviderResponse
      │
      ▼
UsageExtractor
      │
      ▼
PricingStrategy
      │
      ▼
UsageEvent / BillingEvent

RequestContext + ResponseContext
      │
      ▼
ObservabilityEvent
```

---

### 八、模型路由配置示例

```
models:
  gemini3-pro:
    alias: gemini3-pro-preview
    routes:
      - provider: google
        provider_model: gemini3-pro-preview
        weight: 100

  gpt4:
    routes:
      - provider: openai
        provider_model: gpt-4.1
        weight: 80

      - provider: azure
        provider_model: gpt-4
        weight: 20

routing:
  default_strategy: weighted
```

---

### 九、扩展方式

系统扩展通过新增实现类完成，而不需要修改核心流程。

示例：

新增厂商：

* 实现 ProviderAdapter
* 实现 UsageExtractor

新增协议：

* 实现 ProtocolAdapter

新增路由策略：

* 实现 RoutingStrategy

新增计费方式：

* 实现 PricingStrategy

核心系统只依赖抽象接口，新增实现即可扩展能力。

