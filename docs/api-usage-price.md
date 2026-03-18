# 透明代理接入实施方案（国内/海外双版本）

更新时间：2026-03-18（CST）

## 0. 范围与约束

- 范围仅包含在线调用，不考虑批量、离线、包量。
- 视频能力保持异步透明代理。
- 音频能力只接同步透明代理。
- 按 `PROXY_PROVIDER_REGION=domestic|overseas` 切换上游 `base_url`，`path` 保持原厂定义。
- 透明代理的核心要求是：请求 path、请求体字段、响应体字段、任务状态机尽量与上游一致；上游鉴权由代理侧托管。
- 参考 `seedream` 现有接入方式，代码落点仍在 `api.py`、`router.py`、`service.py`、`client.py`、`proxy_model_config.py`、`providers_catalog.py`、`usage_metering/*`、`raw_traffic/*`。
- 与 `seedream` 的主要差异：
  - `seedream` 走 `/images/generations`，是单次请求计费。
  - 本次视频接入包含异步任务查询链路，计费必须基于终态成功并做 `task_id` 去重。
  - 本次音频接入要求透明代理，不应复用现有 `/audio/speech` 的 OpenAI/Azure 协议转换逻辑。

## 1. 上游接口与协议确认

| 厂商 | 能力 | 代理 path | 国内 `base_url` | 海外 `base_url` | 方法 | 调用模式 | 鉴权 | 协议是否一致 |
|---|---|---|---|---|---|---|---|---|
| MiniMax | 视频创建 | `/v1/video_generation` | `https://api.minimaxi.com` | `https://api.minimax.io` | `POST` | 异步 | `Authorization: Bearer <api_key>` | 一致 |
| MiniMax | 视频查询 | `/v1/query/video_generation` | `https://api.minimaxi.com` | `https://api.minimax.io` | `GET` | 异步轮询 | `Authorization: Bearer <api_key>` | 一致 |
| MiniMax | 文件下载信息 | `/v1/files/retrieve` | `https://api.minimaxi.com` | `https://api.minimax.io` | `GET` | 异步结果读取 | `Authorization: Bearer <api_key>` | 一致 |
| MiniMax | 同步语音合成 HTTP | `/v1/t2a_v2` | `https://api.minimaxi.com` | `https://api.minimax.io` | `POST` | 同步 | `Authorization: Bearer <api_key>` | 一致 |
| Seedance | 任务创建 | `/contents/generations/tasks` | `https://ark.cn-beijing.volces.com/api/v3` | `https://ark.ap-southeast.bytepluses.com/api/v3` | `POST` | 异步 | `Authorization: Bearer <api_key>` | 一致 |
| Seedance | 任务查询 | `/contents/generations/tasks/{task_id}` | `https://ark.cn-beijing.volces.com/api/v3` | `https://ark.ap-southeast.bytepluses.com/api/v3` | `GET` | 异步轮询 | `Authorization: Bearer <api_key>` | 一致 |
| 火山大模型语音合成 2.0 | 同步语音合成 | `/api/v1/tts` | `https://openspeech.bytedance.com` | `https://openspeech.byteoversea.com` | `POST` | 同步 | `Authorization: Bearer;{token}`，同时请求体 `app.appid/app.token` | 一致 |

实施口径说明：

- Seedance 国内文档快照里同时出现过 `operator.las.cn-beijing.volces.com/api/v1` 与 `ark.cn-beijing.volces.com/api/v3` 两套路由。
- 本次实施建议统一采用 `ark.../api/v3`，原因是现有 `bytepluses` provider 和海外 BytePlus ModelArk 已经以 `ark` 为主，能最大化复用 `seedream` 的 region/base_url 配置方式。
- 若后续联调确认国内必须走 `operator.las.../api/v1`，只需要在 provider profile 中替换国内 `base_url`，代理 path 与 service 逻辑可以保持不变。

## 2. 透明代理模型清单与计费

说明：

- 只保留在线按量价格。
- 视频模型按上游原始 `model` 透传，不再设计额外 alias。
- 火山 TTS 原厂协议没有统一 `model` 字段，代理内部使用逻辑模型码 `volc.tts` 做计费。

### 2.1 MiniMax 视频

| 上游模型名 | 计费方式 | 国内价格 | 海外价格 |
|---|---|---|---|
| `MiniMax-Hailuo-2.5` | 按生成时长 | `1.25 元/秒` | `$0.16/秒` |
| `MiniMax-Hailuo-2.3-Fast` | 按生成时长 | `0.5 元/秒` | `$0.06/秒` |
| `MiniMax-Hailuo-2.3` | 按生成时长 | `0.8 元/秒` | `$0.10/秒` |
| `MiniMax-Hailuo-02` | 按生成时长 | `0.5 元/秒` | `$0.06/秒` |

### 2.2 MiniMax 同步语音合成

| 上游模型名 | 计费方式 | 国内价格 | 海外价格 |
|---|---|---|---|
| `speech-2.8-turbo` | 按输入字符 | `2 元/万字符` | `$60/百万字符` |
| `speech-2.6-turbo` | 按输入字符 | `2 元/万字符` | `$60/百万字符` |
| `speech-02-turbo` | 按输入字符 | `2 元/万字符` | `$60/百万字符` |
| `speech-2.8-hd` | 按输入字符 | `3.5 元/万字符` | `$100/百万字符` |
| `speech-2.6-hd` | 按输入字符 | `3.5 元/万字符` | `$100/百万字符` |
| `speech-02-hd` | 按输入字符 | `3.5 元/万字符` | `$100/百万字符` |

补充：

- Turbo 三个模型是同一价格档。
- HD 三个模型是同一价格档。

### 2.3 Seedance 视频

| 上游模型名 | 计费方式 | 国内价格 | 海外价格 |
|---|---|---|---|
| `doubao-seedance-1-5-pro-251215` 有声 | 按输出 token | `16 元/百万 token` | `$2.4/百万 token` |
| `doubao-seedance-1-5-pro-251215` 无声 | 按输出 token | `8 元/百万 token` | `$1.2/百万 token` |
| `doubao-seedance-1-0-pro-fast-251015` | 按输出 token | `4.2 元/百万 token` | `$1/百万 token` |

补充：

- `generate_audio=true|false` 只对 `1.5-pro` 生效，并直接决定有声/无声价格档。
- `1.0-pro-fast` 走同一套任务协议，但按无声视频能力计费。

### 2.4 火山大模型语音合成 2.0

| 逻辑模型码 | 计费方式 | 国内价格 | 海外价格 |
|---|---|---|---|
| `volc.tts` | 按输入字符 | `3 元/万字符` | `$50/百万字符` |

## 3. 用量字段与计费取数结论

| 能力 | 是否能从响应直接拿到计费用量 | 字段路径 | 计费时机 | 备注 |
|---|---|---|---|---|
| MiniMax 视频创建 | 否 | 无 | 不计费 | 创建响应只有任务标识，不应在创建阶段扣费。 |
| MiniMax 视频查询 | 否 | 无标准 `usage` | 首次查询到终态成功时计费 | 计费量来自创建请求里保存的 `model` 和 `duration`，并按 `task_id` 去重。 |
| MiniMax 文件下载信息 | 否 | 无 | 不计费 | 只做结果读取，不重复计费。 |
| MiniMax TTS HTTP | 是 | `extra_info.usage_characters` | 响应返回时计费 | 直接按字符数计费。 |
| Seedance 查询 | 是 | `usage.completion_tokens`，可辅以 `usage.total_tokens` | 首次查询到 `succeeded` 时计费 | 必须按 `task_id` 去重，避免轮询重复扣费。 |
| 火山 TTS 2.0 | 否 | 无计费字符数字段 | 响应返回时计费 | 字符数从请求 `request.text` 统计；`addition.duration` 只用于落日志和对账。 |

实现原则：

- 异步视频的计费必须从“创建时扣费”改为“终态成功时扣费”。
- 为避免重复轮询重复扣费，必须增加异步任务计费台账，最少记录：
  - `provider`
  - `task_id`
  - `request_path`
  - `model`
  - `duration`
  - `resolution`
  - `generate_audio`
  - `status`
  - `billed_at`
  - `points_delta`
- 该台账不能只存在内存里；若服务多实例或重启，内存态会导致重复扣费或漏扣。建议落 DB 或 Redis。

## 4. 实施设计

### 4.0 `seedream` 参考基线

当前 `seedream` 的接入模式已经验证了这套 proxy 架构的主要落点：

- API 入口在 `server/backend/app/modules/proxy/api.py`，通过单独 path 暴露能力。
- 路由分流在 `server/backend/app/modules/proxy/router.py`，负责做 path 约束和上游选择。
- provider/profile/model 配置在 `server/backend/app/modules/proxy/proxy_model_config.py` 与 `server/backend/app/modules/proxy/providers_catalog.py`。
- 上游转发在 `server/backend/app/modules/proxy/service.py` 与 `server/backend/app/modules/proxy/client.py`。
- 计费和积分扣减在 `server/backend/app/modules/proxy/usage_metering/*`。

本次透明代理沿用同一套落点，但要做两处关键调整：

- 不再像 `seedream` 那样通过 `/images/generations` 做单次请求计费，而是把异步视频计费迁移到任务终态成功时。
- 不再复用现有音频兼容层，而是新增原厂 path 的同步透明代理入口。

### 4.1 代理路由

计划新增 7 个透明代理入口：

| 文件 | 新增路由 | 说明 |
|---|---|---|
| `server/backend/app/modules/proxy/api.py` | `POST /v1/video_generation` | MiniMax 视频创建，透明透传。 |
| `server/backend/app/modules/proxy/api.py` | `GET /v1/query/video_generation` | MiniMax 视频任务查询，透明透传。 |
| `server/backend/app/modules/proxy/api.py` | `GET /v1/files/retrieve` | MiniMax 文件下载信息查询，透明透传。 |
| `server/backend/app/modules/proxy/api.py` | `POST /contents/generations/tasks` | Seedance 任务创建，透明透传。 |
| `server/backend/app/modules/proxy/api.py` | `GET /contents/generations/tasks/{task_id}` | Seedance 任务查询，透明透传。 |
| `server/backend/app/modules/proxy/api.py` | `POST /v1/t2a_v2` | MiniMax TTS HTTP，同步透明透传。 |
| `server/backend/app/modules/proxy/api.py` | `POST /api/v1/tts` | 火山大模型语音合成 2.0，同步透明透传。 |

路由约束：

- path 必须与上游 path 保持一致，不再映射到 `/chat/completions` 或 `/audio/speech`。
- 请求体/响应体保持原厂结构，不做协议转换。
- 仅鉴权由代理侧注入；客户端不需要传上游厂商的 API Key、AppID、Token。

### 4.2 router 层

参考 `seedream` 在 `server/backend/app/modules/proxy/router.py` 的模式，新增独立 resolver，而不是把这些透明接口混进现有 OpenAI 兼容路由：

- `resolve_minimax_video_create_upstream`
- `resolve_minimax_video_query_upstream`
- `resolve_minimax_file_retrieve_upstream`
- `resolve_seedance_create_upstream`
- `resolve_seedance_query_upstream`
- `resolve_minimax_tts_upstream`
- `resolve_volc_tts_upstream`

设计要求：

- 透明路由只做“path 白名单 + 模型白名单 + region base_url 选择”。
- MiniMax/Seedance 的 `model` 保持上游原值，不做 alias 改写。
- 火山 TTS 只校验 path，不强依赖请求体里存在统一 `model` 字段。

### 4.3 provider/profile 配置

参考 `seedream` 的 provider/profile 组织方式，扩展：

- `server/backend/app/modules/proxy/proxy_model_config.py`
- `server/backend/app/modules/proxy/providers_catalog.py`
- `server/backend/app/modules/proxy/provider_region.py`

建议新增的 provider 分组：

- `minimax-video`
- `minimax-tts`
- `bytepluses-seedance`
- `volcengine-tts`

建议新增的凭证字段：

- `minimax_api_key`：已存在，可复用视频和 TTS。
- `bytepluses_api_key`：已存在，可复用 Seedance。
- `openspeech_app_id`：国内/海外各一套。
- `openspeech_access_token`：国内/海外各一套。

火山 TTS 的特殊点：

- 请求头需要 `Authorization: Bearer;{token}`。
- 请求体 `app.appid`、`app.token` 也需要由代理注入或覆盖。
- 因此不能直接复用现有 `auth_mode="bearer"` 或 `auth_mode="azure_speech"`，需要新增 `volc_tts` 鉴权模式，或者新增专用发送函数。

### 4.4 client/service 层

主要代码落点：

- `server/backend/app/modules/proxy/client.py`
- `server/backend/app/modules/proxy/service.py`
- `server/backend/app/modules/proxy/schema.py`

实现建议：

1. 在 `client.py` 增加原样 JSON `POST` / `GET` 转发能力，保持 body 不改写。
2. 为火山 TTS 增加专用鉴权头组装逻辑，支持 `Bearer;token`。
3. 在 `service.py` 新增透明代理 handler，避免复用现有 `proxy_audio_speech()` 与 `proxy_request()` 中的协议转换分支。
4. 为异步视频新增任务台账存取逻辑：
   - 创建任务成功后写入台账。
   - 查询任务成功时读取台账、判断是否已计费、提取计费数据并只扣一次。
   - 查询到失败、超时、取消时关闭台账但不扣费。
5. `MiniMax 文件下载信息` 只转发，不计费。

不建议的做法：

- 不建议把透明音频直接塞进现有 `/audio/speech`，因为当前 `proxy_audio_speech()` 会做 Azure/OpenAI 到火山协议转换。
- 不建议把透明视频塞进 `/chat/completions`，因为这会混淆 path 语义，也会让 task 计费链路难以建模。

### 4.5 usage metering 接入

参考 `seedream` 的计费链路，主要落点：

- `server/backend/app/modules/proxy/usage_metering/schema.py`
- `server/backend/app/modules/proxy/usage_metering/extractor.py`
- `server/backend/app/modules/proxy/usage_metering/orchestrator.py`
- `server/backend/app/modules/proxy/usage_metering/proxy_points_config.py`
- `server/backend/app/modules/proxy/usage_metering/points_rate_card.py`

需要补充的内容：

1. 在 `NON_TOKEN_PATHS` 中加入：
   - `/v1/video_generation`
   - `/v1/query/video_generation`
   - `/v1/files/retrieve`
   - `/contents/generations/tasks`
   - `/contents/generations/tasks/{task_id}` 的路径匹配规则
   - `/v1/t2a_v2`
   - `/api/v1/tts`
2. 在 `resolve_product_code()` / `resolve_provider_code()` 中加入新 path 的产品与 provider 归类。
3. 在 `extractor.py` 中新增：
   - MiniMax TTS `extra_info.usage_characters`
   - Seedance `usage.completion_tokens`
   - 火山 TTS `addition.duration` 作为 `meta_json` 辅助字段
4. 在 `proxy_points_config.py` 中补充：
   - MiniMax 视频每秒资源码单价
   - MiniMax TTS Turbo/HD 每字符资源码单价
   - 火山 TTS 每字符资源码单价
   - Seedance 1.5 pro / 1.0 pro fast 双区 token 单价
5. 在 `orchestrator.py` 中接入：
   - 同步音频按响应即时记账
   - 异步视频按任务终态成功记账
   - `task_id` 去重后的唯一扣费

### 4.6 raw traffic 接入

参考现有 raw recorder 的白名单机制，扩展：

- `server/backend/app/modules/proxy/raw_traffic/orchestrator.py`

新增采集 path：

- `/v1/video_generation`
- `/v1/query/video_generation`
- `/v1/files/retrieve`
- `/contents/generations/tasks`
- `/contents/generations/tasks/{task_id}` 的路径匹配规则
- `/v1/t2a_v2`
- `/api/v1/tts`

目的：

- 便于异步任务创建请求与后续查询请求对账。
- 便于核对 MiniMax 视频按时长计费和火山 TTS 按字符计费的落账准确性。

## 5. 模型暴露策略

本次透明代理建议采用“原厂模型名直通”策略：

- MiniMax 视频：客户端直接传 `MiniMax-Hailuo-*`。
- MiniMax TTS：客户端直接传 `speech-*`。
- Seedance：客户端直接传 `doubao-seedance-*`。
- 火山 TTS：客户端按原厂协议传 `voice_type`、`audio`、`request` 等字段，不强制统一 `model` 字段。

原因：

- 这是透明代理最符合预期的做法。
- 避免像 `seedream` 那样再引入一层 alias 到 endpoint 的映射复杂度。
- 避免把“路径相关模型”错误暴露到现有 `/proxy/models/supported` 这种通用模型列表里。

如果后续一定要进入统一“支持模型列表”，建议单独增加透明代理模型注册表，而不是混入现有 OpenAI 兼容模型映射。

## 6. TDD 测试计划

建议先补失败用例，再实现。优先修改这些测试文件：

| 测试文件 | 需要覆盖的点 |
|---|---|
| `server/backend/tests/test_proxy_api.py` | 新 path 路由注册、方法正确、请求与响应透明透传。 |
| `server/backend/tests/test_proxy_router.py` | path 精确匹配、非法 path 拒绝、模型白名单、双区 base_url 切换。 |
| `server/backend/tests/test_proxy_service.py` | `service` 不改写透明请求体、正确调用 `send`/`send_get`、正确注入鉴权。 |
| `server/backend/tests/test_proxy_usage_metering.py` | MiniMax TTS 字符取数、Seedance token 取数、火山 TTS 字符统计、异步任务终态成功才扣费、重复轮询不重复扣费。 |
| `server/backend/tests/modules/proxy/usage_metering/test_points_rate_card.py` | 新模型双区价格表与资源码单价。 |
| `server/backend/tests/modules/proxy/raw_traffic/test_recorder.py` | 新 path 进入 raw 录制白名单。 |

必须新增的关键用例：

- MiniMax 视频创建成功只建台账，不立即扣费。
- MiniMax 视频查询 `Success` 时首次扣费，二次查询不重复扣费。
- MiniMax 视频查询 `Fail`/`expired` 不扣费。
- Seedance 查询拿到 `usage.completion_tokens` 后按模型与 `generate_audio` 价格档扣费。
- Seedance 同一 `task_id` 多次 `succeeded` 查询不重复扣费。
- MiniMax TTS 直接用 `extra_info.usage_characters` 计费。
- 火山 TTS 没有 `usage` 时，按 `request.text` 长度计费，并把 `addition.duration` 写入元信息。
- 国内/海外 `PROXY_PROVIDER_REGION` 切换时，`base_url` 与价格档同步切换。

## 7. 实施顺序

建议拆成 4 步：

1. 先加透明路由与 resolver，不接计费，只打通转发。
2. 再补 provider/profile 和火山 TTS 鉴权模式。
3. 再补异步任务台账与 usage metering。
4. 最后补 points rate card、raw traffic 和完整回归测试。

## 8. 风险与决策

- Seedance 国内 base_url 存在文档口径差异。
  - 当前决策：默认走 `ark.cn-beijing.volces.com/api/v3`。
  - 兜底方案：provider profile 切换到 `operator.las.cn-beijing.volces.com/api/v1`。
- 异步视频若没有持久化任务台账，会出现重复扣费或漏扣费。
  - 当前决策：必须做持久化，不接受仅内存态实现。
- 火山 TTS 2.0 没有直接返回计费字符数字段。
  - 当前决策：按请求文本字符数计费，并用控制台账单做对账校验。
- MiniMax 视频没有标准 `usage` 字段。
  - 当前决策：以创建请求中的 `model + duration` 为主计费依据，在查询终态成功时结算。

## 9. 主要依据

- `insight/model-api-docs/minimax-video.md`
- `insight/model-api-docs/minimax-tts.md`
- `insight/model-api-docs/seedance.md`
- `insight/model-api-docs/doubao-tts.md`
- `server/backend/app/modules/proxy/api.py`
- `server/backend/app/modules/proxy/router.py`
- `server/backend/app/modules/proxy/service.py`
- `server/backend/app/modules/proxy/client.py`
- `server/backend/app/modules/proxy/proxy_model_config.py`
- `server/backend/app/modules/proxy/providers_catalog.py`
- `server/backend/app/modules/proxy/provider_region.py`
- `server/backend/app/modules/proxy/usage_metering/orchestrator.py`
- `server/backend/app/modules/proxy/usage_metering/extractor.py`
- `server/backend/app/modules/proxy/usage_metering/schema.py`
- `server/backend/app/modules/proxy/raw_traffic/orchestrator.py`
