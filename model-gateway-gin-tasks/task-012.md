# 实现 Gin request_id 中间件

## Goal
当请求未携带 X-Request-ID 时，系统应生成唯一 ID 并透传到响应头。

## Contract Context
PRD 第三节 Edge Gateway request_id；本任务内 IDGenerator 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/edge -run '^TestRequestIDMiddleware$' -count=1`

### Expected Results
* 并发 20 请求的 request_id 唯一率 100%。
* 已有 X-Request-ID 请求 100% 保持原值。

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
