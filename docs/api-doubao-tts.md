<span id="da88c110"></span>
# **算子介绍**
<span id="28dbc73d"></span>
## 描述
**语音合成模块 - 基于豆包语音大模型的文本转音频解决方案**
<span id="15e450bd"></span>
## **核心功能**

* 接入火山引擎大模型（`volc.tts`）的语音合成接口
* 支持多种参数配置，如音色、情绪、编码格式、语速、采样率等
* 并发处理多个文本输入，输出 Base64 编码音频及原始响应
* 适合用于语音播报、虚拟人声音生成、听力内容制作等场景

<span id="fe765f54"></span>
## **使用场景**

* 智能客服、语音助理内容生成
* 多语言文本播报系统
* 内容平台、短视频配音
* 虚拟人语音驱动

<span id="e8005be9"></span>
## **企业接入说明**
当前该能力仅对通过企业认证的用户开放，如需测试或正式接入，请先完成火山引擎企业认证流程。
<span id="ac365cda"></span>
# **Daft 调用**
<span id="88d42ca4"></span>
## 算子参数
<span id="b602c14a"></span>
### 输入

| | | \
|输入列名 |说明 |
|---|---|
| | | \
|texts |输入文本的数组，每个元素为字符串类型 |

<span id="38accf80"></span>
### 输出
字符串数组，文本生成的音频结果；
若识别失败或输入为空字符串，则对应元素为 None
<span id="ecbb3cff"></span>
### 参数
如参数没有默认值，则为必填参数

| | | | | \
|参数名称 |类型 |默认值 |描述 |
|---|---|---|---|
| | | | | \
|appid |str |None |火山引擎控制台获取的 AppID |
| | | | | \
|token |str |None |火山引擎控制台获取的 Access Token |
| | | | | \
|uid |str |None |用户标识，用于接口调用追踪 |
| | | | | \
|cluster |str |volcano_tts |接入服务所使用的服务集群名 默认值："volcano_tts" |
| | | | | \
|voice_type |str |zh_female_wanqudashu_moon_bigtts |音色类型 默认值："zh_female_wanqudashu_moon_bigtts" |
| | | | | \
|encoding |str |mp3 |音频编码格式（如 "mp3", "pcm"） 默认值："mp3" |
| | | | | \
|speed_ratio |float |1.0 |语速调整比例 默认值：1.0 |
| | | | | \
|rate |int |24000 |采样率 默认值：24000 |
| | | | | \
|bitrate |int |160 |音频比特率（单位 kbps） 默认值：160 |
| | | | | \
|enable_emotion |bool |False |是否启用情绪音色 默认值：False |
| | | | | \
|emotion |str |happy |情绪类型，如 "happy", "angry" 默认值："happy" |
| | | | | \
|emotion_scale |int |4 |情绪强度，范围 1~5 默认值：4 |
| | | | | \
|extra_audio_params |dict or None |None |其他音频参数（可选，传入字典进行补充） 默认值：None |
| | | | | \
|extra_request_params |dict or None |None |其他请求参数（可选，传入字典进行补充） 默认值：None |
| | | | | \
|timeout |int |60 |单个请求的超时时间（秒） 默认值：60 |
| | | | | \
|num_coroutines |int |1 |并发请求数量控制 默认值：1 |

<span id="d9f6d4fb"></span>
## 调用示例
下面的代码展示了如何使用 daft 运行算子将文字转换为语音。
```Python
from __future__ import annotations

import os

import daft
from daft import col
from daft.las.functions.audio.audio_tts_doubao import AudioTtsDoubao
from daft.las.functions.udf import las_udf

if __name__ == "__main__":
    # 从环境变量中读取豆包语音服务的鉴权信息：
    # - OPENSPEECH_APPID：应用 ID，用于标识调用方
    # - OPENSPEECH_TOKEN：访问令牌，用于接口鉴权
    appid = os.getenv("OPENSPEECH_APPID")
    token = os.getenv("OPENSPEECH_TOKEN")

    if os.getenv("DAFT_RUNNER", "native") == "ray":
        import logging

        import ray

        def configure_logging():
            logging.basicConfig(
                level=logging.INFO,
                format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
                datefmt="%Y-%m-%d %H:%M:%S",
            )
            logging.getLogger("tracing.span").setLevel(logging.WARNING)
            logging.getLogger("daft_io.stats").setLevel(logging.WARNING)
            logging.getLogger("DaftStatisticsManager").setLevel(logging.WARNING)
            logging.getLogger("DaftFlotillaScheduler").setLevel(logging.WARNING)
            logging.getLogger("DaftFlotillaDispatcher").setLevel(logging.WARNING)

        ray.init(dashboard_host="0.0.0.0", runtime_env={"worker_process_setup_hook": configure_logging})
        daft.context.set_runner_ray()

    daft.set_execution_config(actor_udf_ready_timeout=600)
    daft.set_execution_config(min_cpu_per_task=0)

    samples = {
        "text_input": [
            "今天天气真好，适合出去走走。",
        ]
    }

    df = daft.from_pydict(samples)
    df = df.with_column(
        "tts_audio",
        las_udf(
            AudioTtsDoubao,
            construct_args={
                "appid": appid,
                "token": token,
                "uid": "test",
                "concurrency": 1,
                "timeout": 60,
            },
            num_cpus=1,
            batch_size=1,
            concurrency=1,
        )(col("text_input")),
    )

    df.show()
    # ╭─────────────────────────────────────────────┬────────────────────────────────╮
    # │ text_input                                  ┆ tts_audio                      │
    # │ ---                                         ┆ ---                            │
    # │ Utf8                                        ┆ Binary                         │
    # ╞═════════════════════════════════════════════╪════════════════════════════════╡
    # │ 欢迎使用豆包语音大模型，这是一个演示示例。…        ┆ b"\xff\xf3\xe4\xc4\x00\x00\x0… │
    # ╰─────────────────────────────────────────────┴────────────────────────────────╯
```
