> ## Documentation Index
> Fetch the complete documentation index at: https://platform.minimaxi.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 视频生成

> 本文档介绍 MiniMax 视频生成服务的使用方法，助力高效创作视频内容。

视频生成服务提供多种功能：

1. 文生视频：根据文本描述直接生成视频
2. 图生视频：基于一张初始图片结合文本描述生成视频
3. 首尾帧生成视频：提供视频开始、结束图片，来生成视频
4. 主体参考生成视频：基于一人脸照片，文本描述生成视频，视频中保持人物特征一致性

## 工作流程

视频生成是一个异步过程，包含以下三个步骤：

1. 创建生成任务：提交一个视频生成请求，获得任务 ID (`task_id`)
2. 查询任务状态：使用 `task_id` 轮询任务状态。任务成功后，会返回一个文件 ID (`file_id`)
3. 获取视频文件：使用 `file_id` 获取视频的下载地址并保存文件

## 功能与代码示例

为了简化代码，我们将轮询和下载的逻辑封装为公共函数，并举例了四种模式下如何创建任务。

```python  theme={null}
import os
import time
import requests

api_key = os.environ["MINIMAX_API_KEY"]
headers = {"Authorization": f"Bearer {api_key}"}


# --- 步骤 1: 发起视频生成任务 ---
# API 支持四种视频生成模式：文生视频、图生视频、首尾帧生成视频、主体参考生成视频。
# 以下四个函数分别对应这四种模式。它们都会发起一个异步的生成任务，并返回一个唯一的 task_id。

def invoke_text_to_video() -> str:
    """（模式一）通过文本描述发起视频生成任务。"""
    url = "https://api.minimaxi.com/v1/video_generation"
    payload = {
        # 'prompt' 是核心参数，用于描述视频的动态内容。
        "prompt": "镜头拍摄一个女性坐在咖啡馆里，女人抬头看着窗外，镜头缓缓移动拍摄到窗外的街道，画面呈现暖色调，色彩浓郁，氛围轻松惬意。",
        "model": "MiniMax-Hailuo-2.3",
        "duration": 6,
        "resolution": "1080P",
    }
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()
    task_id = response.json()["task_id"]
    return task_id


def invoke_image_to_video() -> str:
    """（模式二）通过首帧图像和文本描述发起视频生成任务。"""
    url = "https://api.minimaxi.com/v1/video_generation"
    payload = {
        # 在图生视频模式下，'prompt' 用于描述基于首帧图像的动态变化。
        "prompt": "Contemporary dance，the people in the picture are performing contemporary dance.",
        # 'first_frame_image' 指定了视频的起始画面
        "first_frame_image": "https://filecdn.minimax.chat/public/85c96368-6ead-4eae-af9c-116be878eac3.png",
        "model": "MiniMax-Hailuo-2.3",
        "duration": 6,
        "resolution": "1080P",
    }

    
def invoke_start_end_to_video() -> str:
    """(模式三) 使用首帧图像、尾帧图像和文本描述发起视频生成任务。"""
    url = "https://api.minimaxi.com/v1/video_generation"
    payload = {
    "prompt": "A little girl grow up.",
    # 'first_frame_image' 指定了视频的起始画面
    "first_frame_image": "https://filecdn.minimax.chat/public/fe9d04da-f60e-444d-a2e0-18ae743add33.jpeg",
    # 'last_frame_image' 指定了视频的结束画面
    "last_frame_image": "https://filecdn.minimax.chat/public/97b7cd08-764e-4b8b-a7bf-87a0bd898575.jpeg",
    "model": "MiniMax-Hailuo-02",
    "duration": 6,
    "resolution": "1080P"
    }
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()
    task_id = response.json()["task_id"]
    return task_id


def invoke_subject_reference() -> str:
    """(模式四) 使用人物主体图片和文本描述发起视频生成任务"""
    url = "https://api.minimaxi.com/v1/video_generation"
    payload = {
    "prompt": "On an overcast day, in an ancient cobbled alleyway, the model is dressed in a brown corduroy jacket paired with beige trousers and ankle boots, topped with a vintage beret. The shot starts from over the model's shoulder, following his steps as it captures his swaying figure. Then, the camera moves slightly sideways to the front, showcasing his natural gesture of adjusting the beret with a smile. Next, the shot slightly tilts down, capturing the model's graceful stance as he leans against the wall at a corner. The video concludes with an upward shot, showing the model smiling at the camera. The lighting and colors are natural, giving the footage a cinematic quality.",
    "subject_reference": [
        {
            "type": "character",
            "image": [
                "https://filecdn.minimax.chat/public/54be8fbe-5694-4422-9c95-99cf785eb90e.PNG"
            ],
        }
    ],
    "model": "S2V-01",
    "duration": 6,
    "resolution": "1080P",
}
    
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()
    task_id = response.json()["task_id"]
    return task_id



# --- 步骤 2: 轮询查询任务状态 ---
# 视频生成是一个耗时过程，因此 API 设计为异步模式。
# 提交任务后，需使用 task_id 通过此函数进行轮询，以获取任务的最终状态。
def query_task_status(task_id: str):
    """根据 task_id 轮询任务状态，直至任务成功或失败。"""
    url = "https://api.minimaxi.com/v1/query/video_generation"
    params = {"task_id": task_id}
    while True:
        # 推荐的轮询间隔为 10 秒，以避免对服务器造成不必要的压力。
        time.sleep(10)
        response = requests.get(url, headers=headers, params=params)
        response.raise_for_status()
        response_json = response.json()
        status = response_json["status"]
        print(f"当前任务状态: {status}")
        # 任务成功时，API 会返回一个 'file_id'，用于下一步获取视频文件。
        if status == "Success":
            return response_json["file_id"]
        elif status == "Fail":
            raise Exception(f"视频生成失败: {response_json.get('error_message', '未知错误')}")


# --- 步骤 3: 获取并保存视频文件 ---
# 任务成功后，我们得到的是 file_id 而非直接的下载链接。
# 此函数首先使用 file_id 从文件服务获取下载 URL，然后下载视频内容并保存到本地。
def fetch_video(file_id: str):
    """根据 file_id 获取视频下载链接，并将其保存到本地。"""
    url = "https://api.minimaxi.com/v1/files/retrieve"
    params = {"file_id": file_id}
    response = requests.get(url, headers=headers, params=params)
    response.raise_for_status()
    download_url = response.json()["file"]["download_url"]

    with open("output.mp4", "wb") as f:
        video_response = requests.get(download_url)
        video_response.raise_for_status()
        f.write(video_response.content)
    print("视频已成功保存至 output.mp4")


# --- 主流程: 完整调用示例 ---
# 该部分演示了从发起任务到最终保存视频的完整调用链路。
if __name__ == "__main__":
    # 选择一种方式创建任务
    task_id = invoke_text_to_video()  # 方式一：文生视频
    # task_id = invoke_image_to_video() # 方式二：图生视频
    # task_id = invoke_start_end_to_video() # 方式三: 根据首尾帧生成视频
    # task_id = invoke_subject_reference() # 方式四: 主体参考生成视频

    print(f"视频生成任务已提交，任务 ID: {task_id}")
    file_id = query_task_status(task_id)
    print(f"任务处理成功，文件 ID: {file_id}")
    fetch_video(file_id)
```

## 生成视频结果

### 根据文本生成视频

通过 `prompt` 参数提供一段文本描述，即可生成相应的视频。为了实现对视频内容的精细控制，部分模型支持在 `prompt` 的关键描述后添加 `[运镜]` 指令，来控制镜头。

示例生成结果

<video controls src="https://filecdn.minimax.chat/public/5c6dd9ca-8a10-454e-b9b4-43b135a061ba.mp4" />

### 根据图片生成视频

该功能将 `first_frame_image` 参数指定的图片作为视频的起始帧，并结合 `prompt` 的描述生成后续的动态画面。这使得视频的开场画面完全可控，适合于让静态图片“动起来”的应用场景。

示例生成结果

<video controls src="https://filecdn.minimax.chat/public/002849ca-7d90-4ddb-9be5-1c55b0688180.mp4" />

### 首尾帧生成视频

此模式使用 `first_frame_image` 参数和 `last_frame_image` 参数中指定的图像作为视频的首帧和尾帧。`prompt` 用于描述场景如何从静态图像演变为动态画面。

示例生成结果

<video controls src="https://filecdn.minimax.chat/public/13f1ba45-409d-4e22-bc07-52b7d1d1f411.mp4" />

### Subject Reference

此模式使用 `subject_reference` 参数，将提供的人脸照片作为输入，并结合 `prompt` 描述生成视频，同时确保人物面部特征在整个视频中的一致性。

示例生成结果

<video controls src="https://filecdn.minimax.chat/public/c5d0ce35-a49c-49fd-b6ff-5274a015b6af.mp4" />

## 推荐阅读

<Columns cols={2}>
  <Card title="创建文生视频任务" icon="book-open" href="/api-reference/video-generation-t2v" arrow="true" cta="点击查看">
    使用本接口输入文本内容，创建视频生成任务。
  </Card>

  <Card title="创建图生视频任务" icon="book-open" href="/api-reference/video-generation-i2v" arrow="true" cta="点击查看">
    使用本接口输入图片及文本内容，创建视频生成任务。
  </Card>

  <Card title="产品定价" icon="book-open" href="/guides/pricing-paygo#视频" arrow="true" cta="点击查看">
    各模型的定价说明、计费方式及使用限制。
  </Card>

  <Card title="速率限制" icon="book-open" href="/guides/rate-limits#3%E3%80%81%E6%88%91%E4%BB%AC%E7%9A%84-api-%E7%9A%84%E9%99%90%E9%80%9F%E5%85%B7%E4%BD%93%E6%95%B0%E5%80%BC" arrow="true" cta="点击查看">
    为保证资源的高效使用，引入速率限制，以确保服务的可用性、稳定性。
  </Card>
</Columns>
