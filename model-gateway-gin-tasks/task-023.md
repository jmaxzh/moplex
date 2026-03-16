# 实现 BillingEvent 异步发射器

## Goal
当系统收到成本数据时，系统应异步发射 1 条 BillingEvent 且保证成本值精度。

## Contract Context
PRD 第五节 BillingEvent；本任务内 EventSink 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/events -run '^TestEmitBillingEventAsync$' -count=1`

### Expected Results
* 每次调用精确发射 1 条 BillingEvent。
* cost 精度误差绝对值不超过 1e-6。

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
