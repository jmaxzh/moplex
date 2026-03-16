# 实现 WeightedRoutingStrategy

## Goal
当 Router 使用 weighted 策略时，系统应按权重比例选择 provider/provider_model。

## Contract Context
PRD 第三节 Router；本任务内最小路由契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/router -run '^TestWeightedRoutingStrategy$' -count=1`

### Expected Results
* 1000 次采样下 80/20 命中率分别位于 75%-85% 与 15%-25%。
* 未知模型请求错误返回率 100%。

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
