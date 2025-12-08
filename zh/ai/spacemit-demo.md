---
sidebar_position: 5
---

# SpacemiT AI Demo 示例库

## 简介

SpacemiT AI Demo 示例库是基于进迭时空 K 系列芯片适配的 AI 应用示例集合。该项目为开发者提供了丰富的 AI 模型部署教程和完整的示例代码，涵盖了计算机视觉（CV）、自然语言处理（NLP）、语音处理等多个 AI 领域。

**源码仓库**: [https://gitee.com/bianbu/spacemit-demo.git](https://gitee.com/bianbu/spacemit-demo.git)

## 项目特点

- **开箱即用**: 提供完整的模型下载、量化、部署流程
- **多语言支持**: 同时提供 Python 和 C++ 版本的示例代码  
- **性能优化**: 针对 SpacemiT K1 芯片深度优化，提供详细的性能数据
- **完整文档**: 每个示例都包含详细的 README 文档和使用说明

## 目录结构

```
spacemit_demo/
├── examples/
│   ├── CV/                           # 计算机视觉示例
│   │   ├── yolov5/                  # 目标检测模型
│   │   ├── yolov6/                  # 目标检测模型
│   │   ├── yolov8/                  # 目标检测模型  
│   │   ├── yolov8-pose/             # 姿态检测模型
│   │   ├── yolov11/                 # 目标检测模型
│   │   ├── yolov5-face/             # 人脸检测模型
│   │   ├── yolo-world/              # 开放词汇检测模型
│   │   ├── resnet/                  # 图像分类模型
│   │   ├── efficientnet/            # 图像分类模型
│   │   ├── mobilenet_v2/            # 轻量级分类模型
│   │   ├── inception_v1/            # 图像分类模型
│   │   ├── inception_v3/            # 图像分类模型
│   │   ├── swin-tiny_16xb64_in1k/   # 视觉 Transformer 模型
│   │   ├── fcn/                     # 语义分割模型
│   │   ├── unet/                    # 语义分割模型
│   │   ├── SAM/                     # 图像分割模型
│   │   ├── arcface/                 # 人脸识别模型
│   │   ├── nanotrack/               # 目标跟踪模型
│   │   └── CLIP/                    # 多模态模型
│   └── NLP/                         # 自然语言处理示例
│       ├── spacemit_asr/            # 语音识别模块
│       ├── spacemit_llm/            # 大语言模型模块
│       ├── spacemit_tts/            # 文本转语音模块
│       ├── spacemit_audio/          # 音频处理模块
│       └── *.py                     # 各种 AI 功能演示脚本
└── README.md                        # 项目总览
```

## 示例分类

### 计算机视觉 (CV)

计算机视觉模块包含了主流的 CV 模型示例，涵盖图像分类、目标检测、语义分割、人脸识别等任务：

#### 图像分类
- **ResNet50**: 经典的深度残差网络，适用于图像分类任务
- **EfficientNet**: 高效的卷积神经网络，在精度和效率间取得良好平衡  
- **MobileNetv2**: 专为移动设备优化的轻量级网络
- **Inception**: Google 提出的多尺度特征提取网络
- **Swin Transformer**: 基于窗口注意力的视觉 Transformer

#### 目标检测
- **YOLOv5/v8/v11**: 最新的 YOLO 系列目标检测算法
- **YOLOv6**: 美团提出的高效目标检测算法
- **YOLOv8-pose**: 基于 YOLOv8 的人体姿态检测模型
- **YOLO-World**: 支持开放词汇的目标检测模型

#### 语义分割  
- **FCN**: 全卷积网络，语义分割的经典方法
- **U-Net**: 医学图像分割领域的经典网络
- **SAM**: Meta 提出的分割一切模型

#### 人脸相关
- **ArcFace**: 基于角度边际的人脸识别算法
- **YOLOv5-face**: 专门用于人脸检测的 YOLO 变体

#### 其他
- **NanoTrack**: 轻量级目标跟踪算法
- **CLIP**: OpenAI 的图文多模态理解模型

### 自然语言处理 (NLP)

NLP 模块提供了完整的语音和文本处理解决方案，支持从语音输入到智能回复的全流程：

#### 核心功能
- **语音识别 (ASR)**: 将语音转换为文本，支持实时识别
- **大语言模型 (LLM)**: 支持本地部署 Qwen、DeepSeek 等主流模型
- **文本转语音 (TTS)**: 高质量的语音合成功能
- **函数调用**: LLM 自动选择和调用外部函数的能力
- **视觉语言模型**: 图像理解和文本生成的多模态功能

#### 完整应用场景
- **语音助手**: 语音输入 → 语音识别 → 大模型推理 → 语音合成输出
- **智能对话**: 支持上下文理解的对话系统
- **图像问答**: 基于图像内容进行智能问答

## 性能数据

注：以下性能数据均为最新仓库 README 提供的数据。CV 性能不包含前后处理时间；ASR 为 2 核测试，其余如未特殊说明均为 4 核测试。

### CV 模型性能 (4核)

|   模型大类   |       具体模型        |   输入大小    | 数据类型 | 帧率(4核) | 
| :----------: | :-------------------: | :-----------: | :------: | :-------: |
| EfficientNet |    EfficientNet_b1    | [1,3,224,224] |   int8   |    23     |
|  Inception   |     Inception_v1      | [1,3,224,224] |   int8   |    18     |
|              |     Inception_v3      | [1,3,229,229] |   int8   |    18     |
|  MobileNet   |      MobileNetv2      | [1,3,224,224] |   int8   |    67     |
|    ResNet    |       ResNet50        | [1,3,224,224] |   int8   |    25     |
|    YOLOv5    |        YOLOv5n        | [1,3,640,640] |   int8   |     8     |
|    YOLOv6    |        YOLOv6n        | [1,3,320,320] |   int8   |    54     |
|    YOLOv8    |        YOLOv8n        | [1,3,320,320] |   int8   |    73     |
|              |        YOLOv8n        | [1,3,192,320] |   int8   |    106    |
|   YOLOv11    |       YOLOv11n        | [1,3,320,320] |   int8   |    35     |
|  NanoTrack   |       NanoTrack       | [1,3,255,255] |   int8   |    55     |
|   ArcFace    | Arcface_mobilefacenet | [1,3,320,320] |   int8   |    27     |
| YOLOv5-face  |     YOLOv5n-face      | [1,3,320,320] |   int8   |    21     |


### LLM 模型性能 (4核)

| 模型 | 参数量 | 模型格式 | 量化格式 | 存储大小 | 内存 | 推理速度（4核） | 作用 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Qwen2.5-0.5B | 0.5b | gguf | q4_0 | 336MB | 480MB | 11tokens/s | 对话 |
| Deepseek R1-1.5B | 1.5b | gguf | q4_0 | 1017MB | 1.26GB | 4.5tokens/s | 对话 |
| Qwen2.5-0.5B-f16-agv-fc | 0.5b | gguf | q4_0 | 271MB | 480MB | 11tokens/s | 函数调用 |
| Qwen2.5-0.5B-f16-elephant-fc | 0.5b | gguf | q4_0 | 271MB | 480MB | 11tokens/s | 函数调用 |
| smollm2:135m | 135m | gguf | fp16 | 271MB | 490MB | 15tokens/s | 对话 |
| smollm2-135m-f16-agv-fc | 135m | gguf | fp16 | 270MB | 480MB | 15tokens/s | 函数调用 |
| smollm2-135m-q40-agv-fc | 135m | gguf | q4_0 | 77MB | 320MB | 30tokens/s | 函数调用 |

### ASR 模型性能

| 模型 | 参数量 | 模型格式 | 量化格式 | 存储大小 | 内存 | 推理速度（2核） |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| sensevoice-small（python） | 330m | onnx | 动态量化 | 229MB | 430MB | RTF=0.38 |
| sensevoice-small（c++） | 330m | onnx | 动态量化 | 229MB | 360MB | RTF=0.3 |

### TTS 模型性能

| 模型 | 模型格式 | 量化格式 | 存储大小 | 内存 | 推理速度（4核） |
| :---: | :---: | :---: | :---: | :---: | :---: |
| melotts | onnx | 动态量化 | 74MB | 100MB | RTF: 2-4 |
| matchtts | onnx | model-steps-3动态量化 | 73MB | 300MB | RTF: 0.64-0.75 |

## 快速开始

### 1. 克隆仓库
```bash
git clone https://gitee.com/bianbu/spacemit-demo.git
cd spacemit-demo
```

### 2. 选择示例类型

**计算机视觉示例**:
```bash
cd examples/CV/yolov5
# 下载模型和数据
cd model && sh download_model.sh
cd ../data && sh download_data.sh
# 运行 Python 示例
cd ../python && python test_yolov5.py
```

**自然语言处理示例**:
```bash
cd examples/NLP
# 安装依赖
pip install -r requirements.txt
# 运行语音识别示例
python 03_asr_demo.py
```

## 适用场景

- **AI 学习**: 为初学者提供完整的 AI 模型部署学习资料
- **产品原型**: 快速构建 AI 功能原型和 MVP
- **性能评估**: 评估不同模型在 SpacemiT K1 上的性能表现
- **应用开发**: 为实际项目提供可参考的代码模板

## 技术支持

- **官方文档**: 详细的使用说明和 API 文档
- **问题反馈**: [Gitee Issues](https://gitee.com/bianbu/spacemit-demo/issues)
- **社区交流**: SpacemiT 开发者社区

立即访问 [SpacemiT AI Demo 源码仓库](https://gitee.com/bianbu/spacemit-demo.git) 开始您的 AI 开发之旅！
