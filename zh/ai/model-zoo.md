---
sidebar_position: 4
---

# Model Zoo

## 项目简介

SpacemiT Model Zoo 是面向进迭时空 K 系列芯片适配的 AI 应用集合，提供：

- **计算机视觉（vision）**：检测/分类/分割/跟踪/人脸/姿态等，覆盖示例包括 `resnet`、`yolov8`、`yolov11`、`yolov8_seg`、`yolov8_pose`、`bytetrack`、`ocsort`、`yolov5-face`、`arcface`、`emotion` 等
- **语音**：`VAD`（语音活动检测）、`ASR`（语音识别）、`TTS`（语音合成），提供可直接运行的 demo，便于单模块验证与联调
- **自然语言（LLM）**：OpenAI 兼容接口对接（如 `llama-server`），提供 `llm_chat` 等示例便于快速体验与集成
- **端到端应用示例**：`omni_agent`（VAD -> ASR -> LLM -> TTS，可选 MCP 工具调用），可一键构建后直接跑通

![](../static/model-zoo-arch.png)


**Model Zoo 关键目录速览：**
- **`model_zoo/vision`**：计算机视觉 SDK + `examples/`（检测/分类/分割/跟踪/人脸/姿态等）
- **`model_zoo/asr`**：语音识别 SDK（统一 API + 示例）
- **`model_zoo/tts`**：语音合成 SDK（统一 API + 示例）
- **`model_zoo/vad`**：语音活动检测 SDK（统一 API + 示例）
- **`model_zoo/llm`**：大模型 SDK（OpenAI 兼容服务对接 + 示例）
- **`omni_agent`**：端到端语音对话示例（VAD → ASR → LLM → TTS，可选 MCP 工具调用）


Model Zoo 的每个独立组件都封装了一层通用 API 接口，用于屏蔽底层复杂业务细节，让用户专注于上层应用开发。

**各组件对外头文件（C++ API 入口）：**

- **vision**：`model_zoo/vision/include/vision_service.h`
- **ASR**：`model_zoo/asr/include/asr_service.h`
- **TTS**：`model_zoo/tts/include/tts_service.h`
- **VAD**：`model_zoo/vad/include/vad_service.h`
- **LLM**：`model_zoo/llm/include/llm_service.h`

> **说明**：本文档从「顶层应用」视角提供 Model Zoo 的导航与快速上手。模型下载与准备、依赖安装、API 说明及详细参数请参阅各组件仓库中的 README。各组件仍在持续迭代，文档会随之更新，敬请关注。

## 构建编译

### 获取代码


```bash
sudo apt update
sudo apt install repo

# 中国大陆地区若repo慢，可切换repo为清华镜像：
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'

# 方式一：使用GitHub
repo init -u https://github.com/spacemit-robotics/manifest.git -b main -m omni-agent/model_zoo.xml
repo sync -j4
repo start main --all

# 方式二：使用Gitee：
repo init -u https://gitee.com/spacemit-robotics/manifest.git -b main -m omni-agent/model_zoo.xml
repo sync -j4
repo start main --all
```

### 整体编译

```bash
# source环境变量
source build/envsetup.sh

# 选择编译组件
lunch kx-generic-omni_agent

# 一键编译应用
m
```

## 示例运行

> 说明：以下示例默认你已经按上文完成 `source build/envsetup.sh`、`lunch ...` 并执行过一次 `m`（示例可执行文件已生成并安装到 `output/staging`）。

### 计算机视觉

**步骤1：下载模型**

```bash
# 下载所有 vision 示例模型（会放到 ~/.cache/models/vision/ 下面）
bash components/model_zoo/vision/scripts/download_all_models.sh
```

**步骤2：下载资源文件（图片/视频）**

```bash
# 下载示例用图片/视频资源（会放到 ~/.cache/assets/ 下面）
bash components/model_zoo/vision/scripts/download_assets.sh
```

默认目录：

- 模型：`~/.cache/models/vision/`
- 资源：`~/.cache/assets/`

**步骤3：运行示例**

以下为 `components/model_zoo/vision/examples/` 下**全部示例**的运行命令（C++，执行过 `m` 后可直接运行）：

```bash
# 人脸识别（相似度）
arcface components/model_zoo/vision/examples/arcface/config/arcface.yaml

# 人脸检测
yolov5-face components/model_zoo/vision/examples/yolov5-face/config/yolov5-face.yaml

# 手势检测
yolov5_gesture components/model_zoo/vision/examples/yolov5_gesture/config/yolov5_gesture.yaml

# 目标检测
yolov8 components/model_zoo/vision/examples/yolov8/config/yolov8.yaml
yolov11 components/model_zoo/vision/examples/yolov11/config/yolov11.yaml

# 姿态估计
yolov8_pose components/model_zoo/vision/examples/yolov8_pose/config/yolov8_pose.yaml

# 实例分割
yolov8_seg components/model_zoo/vision/examples/yolov8_seg/config/yolov8_seg.yaml

# 图像分类 / 情绪识别
resnet components/model_zoo/vision/examples/resnet/config/resnet50.yaml
emotion components/model_zoo/vision/examples/emotion/config/emotion.yaml

# 多目标跟踪（视频/摄像头）
bytetrack components/model_zoo/vision/examples/bytetrack/config/bytetrack.yaml
ocsort components/model_zoo/vision/examples/ocsort/config/ocsort.yaml
```

每个示例的可选参数（如 `--image`、`--video`、`--use-camera`、`--output`、阈值等）请参考对应目录的 README（`components/model_zoo/vision/examples/*/README.md`）。

### ASR

**步骤1：下载音频资源（示例音频）**

```bash
mkdir -p ~/.cache/models/assets/audio
cd ~/.cache/models/assets/audio
wget https://archive.spacemit.com/spacemit-ai/model_zoo/assets/audio/001_zh_daily_weather.wav
```

更多音频资源可在 `https://archive.spacemit.com/spacemit-ai/model_zoo/assets/audio` 按需下载。

**步骤2：运行示例**

```bash
# 识别 wav 文件（示例）
asr_file_demo ~/.cache/models/assets/audio/001_zh_daily_weather.wav
```

### TTS

**运行示例**

```bash
# 简单合成（默认参数）
tts_file_demo

# 指定文本与后端（示例）
tts_file_demo -p "你好世界" -l matcha:zh

# 流式合成（示例）
tts_stream_demo -p "自定义文本"
```

### VAD

**运行示例**

```bash
# 内置模拟音频测试
vad_simple_demo
```


### LLM

**步骤1：下载模型（GGUF 示例）**

```bash
mkdir -p ~/.cache/models/llm
cd ~/.cache/models/llm
wget https://archive.spacemit.com/spacemit-ai/model_zoo/llm/qwen2.5-0.5b-instruct-q4_0.gguf
```

如需体验更多模型，可查看 [LLM 模型目录](https://archive.spacemit.com/spacemit-ai/model_zoo/llm) 按需下载。

**步骤2：启动 OpenAI 兼容服务并调用示例**

```bash
# 启动服务（8080）
llama-server -m ~/.cache/models/llm/qwen2.5-0.5b-instruct-q4_0.gguf -t 8 --port 8080 &

# 调用 SDK 示例程序
llm_chat "你好" "http://localhost:8080/v1" "qwen2.5-0.5b" "You are a helpful assistant." 256
```

## 应用示例

### omni_agent

`omni_agent` 示例程序为 `voice_chat`（可选 `voice_chat_aec`）。跑通它需要：

- VAD/ASR/TTS 对应模型已按上文下载到默认缓存目录
- LLM 服务已启动（参考上文 `LLM`）

**步骤1：启动大模型**

```bash
# 启动服务（8080）
llama-server -m ~/.cache/models/llm/qwen2.5-0.5b-instruct-q4_0.gguf -t 8 --port 8080 &
```

**步骤2：列出音频设备**

```bash
voice_chat --list-devices
```

**步骤3：启动对话（最简）**

将 `-i/-o` 替换成你机器上实际的音频输入/输出设备 ID：

```bash
voice_chat -i 0 -o 0 --llm-url http://localhost:8080
```

**步骤4：（可选）启用 MCP 工具调用**

```bash
pip install mcp starlette uvicorn psutil
./components/smart_voice/mcp/examples/start_all_services.sh

voice_chat --llm-url http://localhost:8080 --mcp-config ./components/smart_voice/mcp/examples/config_registry.json
```

## 应用开发

Model Zoo 各组件面向应用侧提供 **稳定的 C++ 头文件入口**（多数为 PIMPL 设计，便于集成与二进制发布），并提供对应的 Python 绑定与示例。应用侧建议先在 SDK workspace 中编译产物到 `output/staging`，再基于 staging 进行联调与部署。

- **vision**：`vision_service.h`（详见 [vision/README.md](https://github.com/spacemit-robotics/model-zoo-vision/blob/main/README.md) 的「应用开发」章节）
- **ASR**：`asr_service.h`（详见 [asr/README.md](https://github.com/spacemit-robotics/model-zoo-asr/blob/main/README.md) 的「应用开发」章节）
- **TTS**：`tts_service.h`（详见 [tts/README.md](https://github.com/spacemit-robotics/model-zoo-tts/blob/main/README.md) 的「应用开发」章节）
- **VAD**：`vad_service.h`（详见 [vad/README.md](https://github.com/spacemit-robotics/model-zoo-vad/blob/main/README.md) 的「应用开发」章节）
- **LLM**：`llm_service.h`（详见 [llm/README.md](https://github.com/spacemit-robotics/model-zoo-llm/blob/main/README.md) 的「应用开发」章节）

如果你的目标是做“对话式应用”，建议直接从 `omni_agent` 入手：先跑通 `voice_chat`，再按需替换/裁剪 ASR、TTS、LLM 后端或接入 MCP 工具。

## 附表：性能数据

性能数据通常随模型、线程数、平台（K3）与编译选项变化较大。以下为当前 `model_zoo` 各组件 README 中提供的阶段性实测数据汇总（均基于 K3 平台，具体测试方法见各组件 README）。

### Vision

> 说明：以下数据基于 K3 平台（4线程推理）包含前后处理（数据截至 2026/3/10，持续迭代中，敬请关注）：

|  模型大类   |       具体模型        |   输入大小    | 数据类型 | 帧率(4核) | 帧率(8核) |
| :---------: | :-------------------: | :-----------: | :------: | :-------: | :-------: |
|   resnet    |       resnet50        | [1,3,224,224] |   int8   |   108.7   |           |
|   arcface   | arcface_mobilefacenet | [1,3,112,112] |   int8   |    49     |           |
| yolov5-face |     yolov5n-face      | [1,3,640,640] |   int8   |   23.0    |           |
|   yolov8    |        yolov8n        | [1,3,640,640] |   int8   |   37.9    |           |
|             |        yolov8s        | [1,3,640,640] |   int8   |   27.5    |           |
|             |        yolov8m        | [1,3,640,640] |   int8   |   16.8    |           |
| yolov8-pose |     yolov8n-pose      | [1,3,640,640] |   int8   |   39.8    |           |
|             |     yolov8s-pose      | [1,3,640,640] |   int8   |   28.5    |           |
|             |     yolov8m-pose      | [1,3,640,640] |   int8   |   17.2    |           |
| yolov8-seg  |      yolov8n-seg      | [1,3,640,640] |   int8   |    4.7    |           |
|             |      yolov8s-seg      | [1,3,640,640] |   int8   |    3.5    |           |
|             |      yolov8m-seg      | [1,3,640,640] |   int8   |    3.4    |           |
|   yolo11    |        yolo11n        | [1,3,640,640] |   int8   |    9.5    |           |
|             |        yolo11s        | [1,3,640,640] |   int8   |    7.1    |           |
|             |        yolo11m        | [1,3,640,640] |   int8   |    4.0    |           |

### ASR

> 说明：以下数据基于 K3 平台（2 线程推理）实测（数据截至 2026/3/10，持续迭代中，敬请关注）：

| 模型 | 量化 | 音频时长 | 处理时间 | RTF |
|------|------|----------|----------|-----|
| SenseVoice (model_quant_optimized.onnx) | INT8 | 41s | 14.3s | 0.35 |

### TTS

> 说明：以下数据基于 K3 平台（2 线程推理）实测（测试数据截至 2026/3/10，持续迭代中，敬请关注）：

| 后端 | 测试文本 | 音频时长 | 处理时间 | RTF |
|------|----------|----------|----------|-----|
| MATCHA_ZH | "这是一个语音合成测试" | 2426ms | 1183ms | 0.49 |
| MATCHA_EN | "Hello, world" | 731ms | 440ms | 0.60 |
| MATCHA_ZH_EN | "今天学Python" | 1920ms | 715ms | 0.37 |
| KOKORO | "你好" | 2075ms | 13814ms | 6.66 |

### LLM

> 说明：以下性能数据是基于 K3 平台按照组件 README 中步骤的实测数据（数据截至 2026/3/10，持续迭代中，敬请关注）。

| 模型                  | 参数量 | 量化参数 | first token latency (ms) | token per second (tokens/s) | E2E latency (s) |
|-----------------------|--------|----------|--------------------------|-----------------------------|-----------------|
| Qwen2.5-0.5B-Instruct | 0.5B   | Q4_0     | 184                      | 47.8                        | 2.8             |
| Qwen3-0.6B            | 0.6B   | Q4_K_M   | 250                      | 36.4                        | 3.7             |
| LFM2.5-1.2B-Instruct  | 1.2B   | Q4_0     | 406                      | 25.4                        | 5.3             |
| Qwen2.5-1.5B-Instruct | 1.5B   | Q4_0     | 398                      | 20.0                        | 6.8             |
| Deepseek R1-1.5B      | 1.5B   | Q4_0     | 463                      | 19.8                        | 6.8             |
| glm-edge-1.5b-chat    | 1.5B   | Q4_0     | 367                      | 21.8                        | 6.2             |
| Qwen3-1.7B            | 1.7B   | Q8_0     | 635                      | 11.6                        | 11.5            |
| Qwen2.5-3B-Instruct   | 3B     | Q4_0     | 806                      | 11.6                        | 12.5            |
| SmallThinker-4B-A0.6B | 4B     | Q4_0     | 345                      | 29.5                        | 4.6             |
| Qwen3-4B              | 4B     | Q4_K_M   | 1477                     | 7.4                         | 18.5            |
| Qwen3-30B-A3B         | 30B    | Q4_0     | 1495                     | 10.0                        | 14.0            |

更详细的复现脚本、测试命令与备注请参考：

- [model-zoo-vision/README.md](https://github.com/spacemit-robotics/model-zoo-vision/blob/main/README.md)（附录：模型性能）
- [model-zoo-asr/README.md](https://github.com/spacemit-robotics/model-zoo-asr/blob/main/README.md)（附录：性能指标）
- [model-zoo-tts/README.md](https://github.com/spacemit-robotics/model-zoo-tts/blob/main/README.md)（附录：性能指标）
- [model-zoo-llm/README.md](https://github.com/spacemit-robotics/model-zoo-llm/blob/main/README.md)（附录：模型性能）
