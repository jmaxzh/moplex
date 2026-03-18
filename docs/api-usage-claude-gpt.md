# Claude/GPT 用量协议与解析备份

本文档用于备份当前代理服务对 Claude（Anthropic Messages）与 GPT（Azure Responses API）两类模型的原始用量结构、解析逻辑与落库映射。

## 1. 模型范围

### 1.1 Claude（`/v1/messages`）

- `claude.opus-4.5`
- `claude.opus-4.6`（GCP upstream）
- `claude.sonnet-4.5`
- `claude.sonnet-4.6`（GCP upstream）
- `claude.haiku-4.5`

### 1.2 GPT（`/responses`）

- `openai.gpt-5.4`
- `openai.gpt-5.4-pro`
- `openai.gpt-5.2`
- `openai.gpt-5.2-codex`
- `openai.gpt-5.3-codex`

## 2. 四类 token 字段口径

- 输入 token：`input_tokens`
- 输出 token：`output_tokens`
- 输入缓存命中 token：`cache_read_input_tokens`（统一内部字段）
- 输入缓存写入 token：`cache_creation_input_tokens`（统一内部字段）

数据库映射：

- `input_tokens -> input_tokens`
- `output_tokens -> output_tokens`
- `cache_read_input_tokens -> cached_input_tokens`
- `cache_creation_input_tokens -> cached_creation_tokens`

## 3. Claude 原始协议与解析

### 3.1 非流式（`/v1/messages`）

原始 usage 示例：

```json
{
  "usage": {
    "input_tokens": 15,
    "output_tokens": 7,
    "cache_creation_input_tokens": 3,
    "cache_read_input_tokens": 2
  }
}
```

解析规则（EARS）：

- WHEN `usage.input_tokens` 存在 THEN 解析为 `input_tokens`
- WHEN `usage.output_tokens` 存在 THEN 解析为 `output_tokens`
- WHEN `usage.cache_read_input_tokens` 存在 THEN 解析为 `cache_read_input_tokens` 且同步写 `cached_input_tokens`
- WHEN `usage.cache_creation_input_tokens` 存在 THEN 解析为 `cache_creation_input_tokens` 且同步写 `cached_creation_tokens`

### 3.2 流式（SSE）

原始事件示例：

```json
{
  "type": "message_start",
  "message": {
    "usage": {
      "input_tokens": 12,
      "output_tokens": 0
    }
  }
}
```

```json
{
  "type": "message_delta",
  "usage": {
    "output_tokens": 13,
    "cache_creation_input_tokens": 3,
    "cache_read_input_tokens": 2
  }
}
```

解析规则（EARS）：

- WHEN 事件类型是 `message_start` 且 `message.usage` 存在 THEN 提取 usage 字段
- WHEN 事件类型是 `message_delta` 且 `usage` 或 `delta.usage` 存在 THEN 提取 usage 字段并覆盖同名键
- WHEN 聚合结束 THEN 将 `cache_read_input_tokens` 同步为 `cached_input_tokens`，将 `cache_creation_input_tokens` 同步为 `cached_creation_tokens`

## 4. GPT（Azure Responses API）原始协议与解析

### 4.1 非流式（`/responses`）

原始 usage 示例：

```json
{
  "usage": {
    "input_tokens": 21,
    "output_tokens": 8,
    "total_tokens": 29,
    "input_tokens_details": {
      "cached_tokens": 5
    }
  }
}
```

解析规则（EARS）：

- WHEN `usage.input_tokens` 存在 THEN 解析为 `input_tokens`
- WHEN `usage.output_tokens` 存在 THEN 解析为 `output_tokens`
- WHEN `usage.total_tokens` 存在 THEN 解析为 `total_tokens`
- WHEN `usage.input_tokens_details.cached_tokens` 存在 THEN 解析为 `cache_read_input_tokens` 且同步写 `cached_input_tokens`
- WHEN 协议未提供 `cache_creation_input_tokens` THEN `cache_creation_input_tokens` 与 `cached_creation_tokens` 记为空（不估算）

### 4.2 流式（SSE）

原始 completed 事件示例：

```json
{
  "type": "response.completed",
  "response": {
    "usage": {
      "input_tokens": 30,
      "output_tokens": 12,
      "total_tokens": 42,
      "input_tokens_details": {
        "cached_tokens": 7
      }
    }
  }
}
```

解析规则（EARS）：

- WHEN `type=response.completed` 且 `response.usage` 存在 THEN 解析 usage
- WHEN `response.usage.input_tokens_details.cached_tokens` 存在 THEN 解析为 `cache_read_input_tokens` 且同步 `cached_input_tokens`
- WHEN 流式结束且无 completed usage THEN 保持现有异常标签策略（`responses_completed_missing_usage` / `responses_stream_no_completed_event`）

## 5. 计费参与规则（当前实现）

解析后参与 points 计算的 token 字段：

- `input_tokens`
- `output_tokens`
- `cache_read_input_tokens`
- `cache_creation_input_tokens`

EARS 规则：

- WHEN `cache_read_input_tokens > 0` THEN 按 cache-read 比例加价
- WHEN `cache_creation_input_tokens > 0` THEN 按 cache-creation 比例加价
- WHEN 字段为空 THEN 按 0 处理，不参与该项计费

对 GPT（Azure Responses API）的结论：

- 协议提供“缓存命中”（`cached_tokens`），可计费
- 协议未提供“缓存写入”，当前不估算，不计该项
