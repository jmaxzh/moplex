# 实现 OpenAI Provider 请求构造器

## Goal
当 Provider 层收到内部请求时，系统应生成可发送到 OpenAI 上游的请求 payload。

## Contract Context
本任务内最小 CanonicalRequest 与 OpenAIUpstreamRequest 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/provider/openai -run '^TestBuildOpenAIRequestPayload$' -count=1`

### Expected Results
* 至少 5 个映射场景通过。
* model/messages/stream 断言不少于 12 条且全部通过。

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
