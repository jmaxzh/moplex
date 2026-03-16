# 实现 OpenAI 入站请求适配器

## Goal
当系统收到 OpenAI 请求 DTO 时，系统应转换为内部 CanonicalRequest。

## Contract Context
本任务内最小 OpenAIRequestDTO 与 CanonicalRequest 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/protocol/openai -run '^TestToCanonicalRequest$' -count=1`

### Expected Results
* 至少 6 个映射场景通过。
* 字段断言不少于 20 条且通过率 100%。

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
