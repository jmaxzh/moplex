# 实现 Gin 限流中间件

## Goal
当单客户端请求超过阈值时，系统应返回 429 并阻止进入后续处理链。

## Contract Context
PRD 第三节 Edge Gateway 限流；本任务内 LimiterStore/Clock 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/edge -run '^TestRateLimitMiddleware$' -count=1`

### Expected Results
* 阈值=5 时第 6 次请求 100% 返回 429。
* 窗口重置后首请求 100% 返回 200。

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
