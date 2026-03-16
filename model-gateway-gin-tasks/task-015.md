# 实现路由配置加载器

## Goal
当服务读取 YAML 路由配置时，系统应解析并校验模型与 routes 的基本合法性。

## Contract Context
PRD 第八节；本任务内 RoutingConfig 最小契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/router -run '^TestLoadRoutingConfig$' -count=1`

### Expected Results
* 合法配置解析出至少 2 个模型与 3 条 route。
* 至少 4 类非法配置全部返回错误。

## Subagent Acceptance

### Gate Rules
* 仅当 `Verdict=pass`，任务可标记 `done`。

### Checklist
* 输出满足 Goal（EARS）
* Validation 可复现
* 无严重副作用

### Verdict
Enum: `pass | fail`

## Status
pending

## Acceptance Report
（待执行阶段填写）
