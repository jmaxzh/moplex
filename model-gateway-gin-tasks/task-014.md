# 实现 OpenAI 出站响应适配器

## Goal
当系统得到内部 CanonicalResponse 时，系统应转换为 OpenAI 响应 DTO。

## Contract Context
本任务内最小 CanonicalResponse 与 OpenAIResponseDTO 契约。

## Execution Protocol
* step1，仅实现当前子任务的 Goal。
* step2，完成实现后立即执行 Validation。
* step3，Validation 通过后执行 Subagent Acceptance。
* step4，若 `Verdict=fail`，撰写`Acceptance Report`，然后回到 step1。
* step5，若 `Verdict=pass`，撰写`Acceptance Report`，更新任务状态为 `done`。

## Validation

### Commands
* `go test ./internal/protocol/openai -run '^TestFromCanonicalResponse$' -count=1`

### Expected Results
* 至少 5 个场景全部通过。
* 关键字段断言不少于 16 条且通过率 100%。

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
