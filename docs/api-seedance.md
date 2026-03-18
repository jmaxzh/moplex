<span id="f65be34b"></span>
# **算子介绍**
<span id="1bb1c0a1"></span>
## 描述
Seedance是字节跳动豆包大模型团队最新推出的视频生成基础模型，可根据用户输入的文本、图片等内容，快速生成优质的视频片段
<span id="47499b27"></span>
## 使用限制

* 暂只支持文生视频和图生视频
* 不支持调整帧数
* 视频生成为异步接口，您需要先创建视频生成任务，再通过视频生成任务的 ID 去查询视频生成结果

<span id="28b1c2af"></span>
## 核心功能

* 视频生产

<span id="54643028"></span>
# 注意与前提

| | | \
|细分项 |注意与前提 |
|---|---|
| | | \
|费用 |调用算子前，您需先了解使用算子时的模型调用费用，详情请参见[大模型调用计费](https://www.volcengine.com/docs/6492/1544808)。 |
| | | \
|鉴权（API Key） |调用算子前，您需要先生成算子调用的API Key，并建议将API Key配置为环境变量，便于更安全地调用算子，详情请参见[获取 API Key 并配置](/docs/6492/2191994)。 |
| | | \
|BaseURL |调用算子前，您需要先根据您当前使用的LAS服务所在地域，了解算子调用的BaseURL，用于配置算子调用路径参数取值。 |\
| |详情请参见[获取 Base URL](/docs/6492/2191993)，下文中的调用示例仅作为参考，实际调用时需替换为您对应地域的路径取值。 |



<span id="1122633d"></span>
# **API 调用**
<span id="5a594008"></span>
## Generate
`POST https://operator.las.cn-beijing.volces.com/api/v1/contents/generations/tasks`
<span id="4489c352"></span>
### 接口说明
调用 seedance模型 进行视频生成。
<span id="4a78059b"></span>
### 请求参数

| | | | | | \
|参数 |类型 |必填 |示例值 |说明 |
|---|---|---|---|---|
| | | | | | \
|model |\
| |string |\
| | |是 |doubao-seedance-1-5-pro-251215 |您需要调用的模型的 ID （Model ID）。支持： |\
| | | | | |\
| | | | |* doubao-seedance-1-5-pro-251215,  |\
| | | | |* doubao-seedance-1-0-pro-fast-251015  |\
| | | | |* doubao-seedance-1-0-pro-250528,  |\
| | | | |* doubao-seedance-1-0-lite-t2v-250428 |
| | | | | | \
|content |\
| |list of object |\
| | |是 | |输入给模型，生成视频的信息，支持文本信息、图片信息和样片信息。参考 |\
| | | | |[创建视频生成任务 API](https://www.volcengine.com/docs/82379/1520757?lang=zh)中的 `content`参数。 |
| | | | | | \
|callback_url |\
| |string |\
| | |否 |http://xxx/callback |填写本次生成任务结果的回调通知地址。当视频生成任务有状态变化时，方舟将向此地址推送 POST 请求。 |\
| | | | | |\
| | | | |* 回调请求内容结构与查询任务API的返回体一致。 |\
| | | | |* 回调返回的 status 包括以下状态： |\
| | | | |   * queued：排队中。 |\
| | | | |   * running：任务运行中。 |\
| | | | |   * succeeded： 任务成功。 |\
| | | | |   * failed：任务失败。（如发送失败，即5秒内没有接收到成功发送的信息，回调三次） |\
| | | | |   * expired：任务超时，即任务处于运行中或排队中状态超过过期时间。可通过 execution_expires_after 字段设置过期时间。 |
| | | | | | \
|return_last_frame |\
| |boolean |\
| | |否 |false |是否返回视频最后一帧的图像，默认值为 false。 |\
| | | | | |\
| | | | |* true：返回生成视频的尾帧图像。设置为 true 后，可通过 查询视频生成任务接口 获取视频的尾帧图像。 |\
| | | | |   * 尾帧图像的格式为 png，宽、高、像素值与生成的视频保持一致，无水印。 |\
| | | | |   * 使用该参数可实现生成多个连续视频：以上一个生成视频的尾帧作为下一个视频任务的首帧，快速生成多个连续视频。 |\
| | | | |* false：不返回生成视频的尾帧图像。 |
| | | | | | \
|execution_expires_after |integer |\
| | |否 |172800 |任务超时阈值。指定任务提交后的过期时间（单位：秒），从 **created at** 时间戳开始计算。默认值 172800 秒，即 48 小时。取值范围：[3600，259200] |
| | | | | | \
|generate_audio |boolean |否 |true |默认为true，控制生成的视频是否包含与画面同步的声音。 |\
| | | | |:::warning |\
| | | | |仅 doubao-seedance 1.5 pro 支持。 |\
| | | | |::: |\
| | | | | |\
| | | | |* true：模型输出的视频包含同步音频。doubao-seedance 1.5 pro 能够基于文本提示词与视觉内容，自动生成与之匹配的人声、音效及背景音乐。 |\
| | | | |   * 建议将对话部分置于双引号内，以优化音频生成效果。 |\
| | | | |   * 例如：男人叫住女人说：“你记住，以后不可以用手指指月亮。” |\
| | | | |* false：模型输出的视频为无声视频。 |
| | | | | | \
|draft |boolean |否 | |默认false，控制是否开启样片模式 |
| | | | | | \
|resolution |string |否 |720p |视频分辨率，枚举值： |\
| | | | | |\
| | | | |* `480p` |\
| | | | |* `720p` |\
| | | | |* `1080p`：参考图场景不支持 |\
| | | | | |\
| | | | |:::warning |\
| | | | |* doubao-seedance 1.5 pro、doubao-seedance 1.0 lite 默认值 `720p` |\
| | | | |* doubao-seedance 1.0 pro&pro-fast 默认值：`1080p` |\
| | | | |::: |
| | | | | | \
|ratio |string |否 |16:9 |生成视频的宽高比例。 |\
| | | | | |\
| | | | |* 16:9 |\
| | | | |* 4:3 |\
| | | | |* 1:1 |\
| | | | |* 3:4 |\
| | | | |* 9:16 |\
| | | | |* 21:9 - adaptive: 根据输入自动选择最合适的宽高比。 |\
| | | | | |\
| | | | |:::tip |\
| | | | |* 文生视频：默认 `16:9`(doubao-seedance 1.5 Pro 默认值为 `adaptive`） |\
| | | | |* 图文视频：默认值 `adaptive`（参考图文视频场景默认值为 `16:9`） |\
| | | | |::: |
| | | | | | \
|duration |integer |否 |5 |> duration 和 frames 二选一即可，frames 的优先级高于 duration。如果您希望生成整数秒的视频，建议指定 duration。 |\
| | | | | |\
| | | | |生成视频时长，单位：秒。支持 2~12 秒。 |
| | | | | | \
|frames |\
| |integer |否 | |> doubao-seedance 1.5 pro 暂不支持。duration 和 frames 二选一即可，frames 的优先级高于 duration。如果您希望生成小数秒的视频，建议指定 frames。 |\
| | | | | |\
| | | | |生成视频的帧数。通过指定帧数，可以灵活控制生成视频的长度，生成小数秒的视频。 |\
| | | | |由于 frames 的取值限制，仅能支持有限小数秒，您需要根据公式推算最接近的帧数。 |\
| | | | | |\
| | | | |* 计算公式：帧数 = 时长 × 帧率（24）。 |\
| | | | |* 取值范围：支持 [29, 289] 区间内所有满足 `25 + 4n` 格式的整数值，其中 n 为正整数。 |\
| | | | | |\
| | | | |例如：假设需要生成 2.4 秒的视频，帧数=2.4×24=57.6。由于 frames 不支持 57.6，此时您只能选择一个最接近的值。根据 25+4n 计算出最接近的帧数为 57，实际生成的视频为 57/24=2.375 秒。 |
| | | | | | \
|seed |integer |否 |-1 |种子整数，用于控制生成内容的随机性。取值范围：[-1, 2^32-1]之间的整数。 |
| | | | | | \
|camera_fixed |boolean |否 |false |> 参考图场景不支持 |\
| | | | | |\
| | | | |是否固定摄像头。枚举值： |\
| | | | | |\
| | | | |* true：固定摄像头。平台会在用户提示词中追加固定摄像头，实际效果不保证。 |\
| | | | |* false：不固定摄像头。 |\
| | | | | |\
| | | | |默认值为 `false`。 |
| | | | | | \
|watermark |boolean |否 |false |生成视频是否包含水印。枚举值： |\
| | | | | |\
| | | | |* false: 不含水印。 |\
| | | | |* true: 含有水印。 |

<span id="a44db6f6"></span>
### 返回数据

| | | | \
|参数 |类型 |说明 |
|---|---|---|
| | | | \
|id |string |任务ID |

<span id="d89b2d80"></span>
### 示例
<span id="0d4a79da"></span>
#### 请求示例
```Bash
curl --location "https://operator.las.cn-beijing.volces.com/api/v1/contents/generations/tasks" \
--header "Content-Type: application/json" \
--header "Authorization: Bearer $LAS_API_KEY" \
--data '{
    "model": "doubao-seedance-1-0-pro-250528",
    "content": [
        {
            "type": "text",
            "text": "写实风格，晴朗的蓝天之下，一大片白色的雏菊花田，镜头逐渐拉近，最终定格在一朵雏菊花的特写上，花瓣上有几颗晶莹的露珠"
        }
    ],
    "ratio": "16:9",
    "duration": 5,
    "watermark": false
}'
```

<span id="a23a0ee2"></span>
#### 返回示例
```JSON
{
    "id": "cgt-20251125163544-qrj4f"
}
```

<span id="3ef92e42"></span>
## Task
`GET https://operator.las.cn-beijing.volces.com/api/v1/contents/generations/tasks/{id}`
<span id="c6f6178a"></span>
### 接口说明
查询视频生成任务状态。
<span id="3aafb3ff"></span>
### 请求参数

| | | | | | \
|参数 |类型 |必填 |示例值 |说明 |
|---|---|---|---|---|
| | | | | | \
|id |string |是 |cgt-2025******-**** |您需要查询的视频生成任务的 ID 。 |\
| | | | |本请求参数为Query String Parameters，在URL String中传入。 |

<span id="51b1f09f"></span>
### 返回数据

```mixin-react
const properties = ({
  "columns": [
    { "title": "参数", "dataIndex": "column_0", "width": 300 },
    { "title": "类型", "dataIndex": "column_1", "width": 150 },
    { "title": "示例值", "dataIndex": "column_2", "width": 150 },
    { "title": "说明", "dataIndex": "column_3" }
  ],
  "data": [
    {
      "key": 0,
      "children": [],
      "column_0": "id",
      "column_1": "string",
      "column_2": "cgt-2025******-****",
      "column_3": "视频生成任务 ID"
    },
    {
      "key": 1,
      "children": [],
      "column_0": "model",
      "column_1": "string",
      "column_2": "",
      "column_3": "任务使用的模型名称和版本，模型名称-版本"
    },
    {
      "key": 2,
      "children": [],
      "column_0": "status",
      "column_1": "string",
      "column_2": "succeeded",
      "column_3": <div>模型状态以及相关的信息：<ul><li>queued: 排队中。</li><li>running：任务运行中。</li><li>cancelled: 取消任务，取消状态24h自动删除（只支持排队中状态的任务被取消）。</li><li>succeeded: 任务成功。</li><li>failed: 任务失败。</li><li>expired: 任务超时。</li></ul></div>
    },
    {
      "key": 3,
      "children": [
        {
          "key": 4,
          "children": [],
          "column_0": "code",
          "column_1": "string",
          "column_2": "",
          "column_3": "错误码。"
        },
        {
          "key": 5,
          "children": [],
          "column_0": "message",
          "column_1": "string",
          "column_2": "",
          "column_3": "错误提示信息。"
        }
      ],
      "column_0": "error",
      "column_1": "error",
      "column_2": "",
      "column_3": <div>错误提示信息，任务成功返回null，任务失败时返回错误数据，错误信息具体参见 <a target="_blank" href="https://www.volcengine.com/docs/6492/131872#%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86">错误处理</a>。</div>
    },
    {
      "key": 6,
      "children": [],
      "column_0": "created_at",
      "column_1": "integer",
      "column_2": "1764059744",
      "column_3": "创建时间"
    },
    {
      "key": 7,
      "children": [],
      "column_0": "updated_at",
      "column_1": "integer",
      "column_2": "1764059744",
      "column_3": "更新时间"
    },
    {
      "key": 8,
      "children": [
        {
          "key": 9,
          "children": [],
          "column_0": "video_url",
          "column_1": "string",
          "column_2": "",
          "column_3": "生成视频的 URL，格式为 mp4。为保障信息安全，生成的视频会在24小时后被清理，请及时转存。"
        },
        {
          "key": 10,
          "children": [],
          "column_0": "last_frame_url",
          "column_1": "string",
          "column_2": "",
          "column_3": <div>视频的尾帧图像 URL。有效期为 24小时，请及时转存。<br/>说明：<a target="_blank" href="https://www.volcengine.com/docs/6492/2165104#%E5%88%9B%E5%BB%BA%E8%A7%86%E9%A2%91%E7%94%9F%E6%88%90%E4%BB%BB%E5%8A%A1">创建视频生成任务</a> 时设置 "return_last_frame": true 时，会返回该参数。</div>
        },
        {
            "key": 11,
            "children": [],
            "column_0": "file_url",
            "column_1": "string",
            "column_2": "",
            "column_3": "视频生成结果文件 URL（如 flv 等非 mp4 格式）。"
        }
      ],
      "column_0": "content",
      "column_1": "content",
      "column_2": "",
      "column_3": "视频生成任务的输出内容。"
    },
    {
      "key": 12,
      "children": [],
      "column_0": "seed",
      "column_1": "integer",
      "column_2": "1345",
      "column_3": "本次请求使用的种子整数值。"
    },
    {
      "key": 13,
      "children": [],
      "column_0": "resolution",
      "column_1": "string",
      "column_2": "1080p",
      "column_3": "生成视频的分辨率。"
    },
    {
      "key": 14,
      "children": [],
      "column_0": "ratio",
      "column_1": "string",
      "column_2": "9:16",
      "column_3": "生成视频的宽高比。"
    },
    {
      "key": 15,
      "children": [],
      "column_0": "duration",
      "column_1": "integer",
      "column_2": "10",
      "column_3": <div>生成视频的时长，单位：秒。<br/>说明：duration 和 frames 参数只会返回一个。创建视频生成任务时未指定 frames，会返回 duration。</div>
    },
    {
      "key": 16,
      "children": [],
      "column_0": "frames",
      "column_1": "integer",
      "column_2": "",
      "column_3": <div>生成视频的帧数。<br/>说明：duration 和 frames 参数只会返回一个。创建视频生成任务时指定了 frames，会返回 frames。</div>
    },
    {
      "key": 17,
      "children": [],
      "column_0": "framespersecond",
      "column_1": "integer",
      "column_2": "30",
      "column_3": "生成视频的帧率。"
    },
    {
      "key": 18,
      "children": [],
      "column_0": "generate_audio",
      "column_1": "boolean",
      "column_2": "false",
      "column_3": <div>生成的视频是否包含与画面同步的声音。仅 Seedance 1.5 pro 会返回该参数。<ul><li>true: 模型输出的视频包含同步音频。</li><li>false: 模型输出的视频为无声视频。</li></ul></div>
    },
    {
      "key": 19,
      "children": [],
      "column_0": "draft",
      "column_1": "boolean",
      "column_2": "false",
      "column_3": <div>生成的视频是否为 Draft 视频。仅 Seedance 1.5 pro 会返回该参数。<ul><li>true: 表示当前输出为 Draft 视频。</li><li>false: 表示当前输出为正常视频。</li></ul></div>
    },
    {
      "key": 20,
      "children": [],
      "column_0": "draft_task_id",
      "column_1": "string",
      "column_2": "",
      "column_3": "Draft 视频任务 ID。基于 Draft 视频生成正式视频时，会返回该参数。"
    },
    {
      "key": 21,
      "children": [],
      "column_0": "execution_expires_after",
      "column_1": "integer",
      "column_2": "",
      "column_3": "任务超时阈值，单位：秒。"
    },
    {
      "key": 22,
      "children": [
        {
          "key": 23,
          "children": [],
          "column_0": "completion_tokens",
          "column_1": "integer",
          "column_2": "13987",
          "column_3": "输出内容 token 数量。"
        },
        {
          "key": 24,
          "children": [],
          "column_0": "total_tokens",
          "column_1": "integer",
          "column_2": "15000",
          "column_3": "本次请求消耗的总 token 数量。视频生成模型不统计输入 token，输入 token 为 0，故 total_tokens=completion_tokens。"
        }
      ],
      "column_0": "usage",
      "column_1": "token_usage",
      "column_2": "",
      "column_3": "视频生成token使用量"
    }
  ]
}); 
return <Table border={{ cell: true, wrapper: true }} pagination={false} {...properties} />;
```

<span id="83c01e83"></span>
### 示例
<span id="521bc663"></span>
#### 请求示例
```Bash
curl --location "https://operator.las.cn-beijing.volces.com/api/v1/contents/generations/tasks/cgt-xxxxxx" \
--header "Content-Type: application/json" \
--header "Authorization: Bearer $LAS_API_KEY"
```

<span id="e31dd27b"></span>
#### 返回示例
```JSON
{
    "framesPerSecond": 24,
    "id": "cgt-20251119202422-jcfm2",
    "model": "doubao-seedance-1-0-pro-250528",
    "status": "succeeded",
    "content": {
        "video_url": "https://ark-content-generation-cn-beijing.tos-cn-beijing.volces.com/xxx.mp4",
        "last_frame_url": "https://ark-content-generation-cn-beijing.tos-cn-beijing.volces.com/xxx.png",
        "file_url": null
    },
    "usage": {
        "completion_tokens": 295800
    },
    "frames": 145,
    "framespersecond": 24,
    "created_at": 1763555062,
    "updated_at": 1763555155,
    "seed": 655,
    "ratio": "9:16",
    "resolution": "1080p"
}
```

<span id="db39c554"></span>
# 错误码

| | | | | \
|HttpCode |错误码 |错误信息 |说明 |
|---|---|---|---|
| | | | | \
|401 |ApiKey.Invalid |The api key is invalid. |API不合法 |
