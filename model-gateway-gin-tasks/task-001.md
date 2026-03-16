# 定义 CanonicalRequest 最小契约

## Goal
当系统接收任意入站请求时，系统应使用统一 CanonicalRequest 结构并保证 JSON 编解码稳定。

## Contract Context
PRD 第四节 CanonicalRequest。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/canonical -run '^TestCanonicalRequestContract$' -count=1 -cover`

### Expected Results
* 至少 6 个用例通过，失败数为 0。
* 覆盖率不低于 85%。

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
