# 实现 TokenPricingStrategy

## Goal
当系统得到 Usage 指标时，系统应按 token 单价计算成本。

## Contract Context
PRD 第三节 PricingStrategy；本任务内最小 Usage 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/billing -run '^TestCalculateTokenCost$' -count=1`

### Expected Results
* 至少 5 个定价场景通过。
* 成本误差绝对值不超过 1e-6。

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
