# 实现 Gin 鉴权中间件

## Goal
当请求进入 Gin 网关时，系统应校验 Authorization 并对非法请求返回 401。

## Contract Context
PRD 第三节 Edge Gateway 鉴权；本任务内 TokenValidator 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/edge -run '^TestAuthMiddleware$' -count=1`

### Expected Results
* 至少 4 个鉴权场景通过。
* 非法请求 100% 返回 401，合法请求 100% 放行。

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
