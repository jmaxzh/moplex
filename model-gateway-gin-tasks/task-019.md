# 实现 OpenAI Provider HTTP 调用器

## Goal
当 Provider 调用 OpenAI 上游时，系统应完成 HTTP 请求、超时控制与错误状态映射。

## Contract Context
PRD 第三节 Provider Adapter；本任务内调用接口契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/provider/openai -run '^TestInvokeOpenAIHTTP$' -count=1`

### Expected Results
* 200/5xx/timeout 三类场景通过率 100%。
* timeout 场景在 200ms 内返回并匹配预期错误码。

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
