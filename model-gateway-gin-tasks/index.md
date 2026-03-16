# Task Index

## Goal
当需要将 `prd.md` 拆分为可执行开发计划时，系统应输出一组满足原子化原则的子任务清单，覆盖模型代理网关（Golang + Gin）的契约、网关、适配、路由、计量、计费与可观测能力，并确保每个子任务都具备可独立执行与可验证的验收标准。

## Planning Protocol

1. 仅允许一个任务为 **in-progress**。
2. 若任务 `Verdict=fail`，禁止标记为 **done**，必须安排修复与复验。
3. 有依赖任务按依赖拓扑顺序推进，无依赖任务可并行处理。

## Status Enum
`pending | in-progress | blocked | done`

## Tasks

### TASK-001
**标题**: 定义 `CanonicalRequest` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-001.md

### TASK-002
**标题**: 定义 `CanonicalResponse` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-002.md

### TASK-003
**标题**: 定义 `ProviderResponse` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-003.md

### TASK-004
**标题**: 定义 `Usage` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-004.md

### TASK-005
**标题**: 定义 `UsageEvent` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-005.md

### TASK-006
**标题**: 定义 `BillingEvent` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-006.md

### TASK-007
**标题**: 定义 `ObservabilityEvent` 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-007.md

### TASK-008
**标题**: 定义 OpenAI 请求 DTO 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-008.md

### TASK-009
**标题**: 定义 OpenAI 响应 DTO 最小契约  
**状态**: pending  
**依赖**: none  
**File**: task-009.md

### TASK-010
**标题**: 实现 Gin 鉴权中间件  
**状态**: pending  
**依赖**: none  
**File**: task-010.md

### TASK-011
**标题**: 实现 Gin 限流中间件  
**状态**: pending  
**依赖**: none  
**File**: task-011.md

### TASK-012
**标题**: 实现 Gin `request_id` 中间件  
**状态**: pending  
**依赖**: none  
**File**: task-012.md

### TASK-013
**标题**: 实现 OpenAI 入站请求适配器  
**状态**: pending  
**依赖**: none  
**File**: task-013.md

### TASK-014
**标题**: 实现 OpenAI 出站响应适配器  
**状态**: pending  
**依赖**: none  
**File**: task-014.md

### TASK-015
**标题**: 实现路由配置加载器  
**状态**: pending  
**依赖**: none  
**File**: task-015.md

### TASK-016
**标题**: 实现 `WeightedRoutingStrategy`  
**状态**: pending  
**依赖**: none  
**File**: task-016.md

### TASK-017
**标题**: 实现 OpenAI Provider 请求构造器  
**状态**: pending  
**依赖**: none  
**File**: task-017.md

### TASK-018
**标题**: 实现 OpenAI Provider 响应解析器  
**状态**: pending  
**依赖**: none  
**File**: task-018.md

### TASK-019
**标题**: 实现 OpenAI Provider HTTP 调用器  
**状态**: pending  
**依赖**: none  
**File**: task-019.md

### TASK-020
**标题**: 实现 OpenAI `UsageExtractor`  
**状态**: pending  
**依赖**: none  
**File**: task-020.md

### TASK-021
**标题**: 实现 `TokenPricingStrategy`  
**状态**: pending  
**依赖**: none  
**File**: task-021.md

### TASK-022
**标题**: 实现 `UsageEvent` 异步发射器  
**状态**: pending  
**依赖**: none  
**File**: task-022.md

### TASK-023
**标题**: 实现 `BillingEvent` 异步发射器  
**状态**: pending  
**依赖**: none  
**File**: task-023.md

### TASK-024
**标题**: 实现 Gin Observability 异步发射中间件  
**状态**: pending  
**依赖**: none  
**File**: task-024.md
