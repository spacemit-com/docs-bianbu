---
sidebar_position: 5
---

# SpacemiT AI Demo Repository

## Introduction

The SpacemiT AI Demo Repository is a comprehensive collection of AI application examples adapted for SpacemiT K-series chips. This project provides developers with rich AI model deployment tutorials and complete sample code, covering multiple AI domains including Computer Vision (CV), Natural Language Processing (NLP), and speech processing.

**Source Repository**: [https://gitee.com/bianbu/spacemit-demo.git](https://gitee.com/bianbu/spacemit-demo.git)

## Key Features

- **Ready-to-Use**: Complete model download, quantization, and deployment workflow
- **Multi-Language Support**: Provides both Python and C++ example code
- **Performance Optimized**: Deeply optimized for SpacemiT K1 chips with detailed performance metrics
- **Complete Documentation**: Each example includes detailed README documentation and usage instructions

## Directory Structure

```
spacemit_demo/
├── examples/
│   ├── CV/                           # Computer Vision Examples
│   │   ├── yolov5/                  # Object Detection Model
│   │   ├── yolov6/                  # Object Detection Model
│   │   ├── yolov8/                  # Object Detection Model  
│   │   ├── yolov8-pose/             # Pose Detection Model
│   │   ├── yolov11/                 # Object Detection Model
│   │   ├── yolov5-face/             # Face Detection Model
│   │   ├── yolo-world/              # Open-Vocabulary Detection Model
│   │   ├── resnet/                  # Image Classification Model
│   │   ├── efficientnet/            # Image Classification Model
│   │   ├── mobilenet_v2/            # Lightweight Classification Model
│   │   ├── inception_v1/            # Image Classification Model
│   │   ├── inception_v3/            # Image Classification Model
│   │   ├── swin-tiny_16xb64_in1k/   # Vision Transformer Model
│   │   ├── fcn/                     # Semantic Segmentation Model
│   │   ├── unet/                    # Semantic Segmentation Model
│   │   ├── SAM/                     # Image Segmentation Model
│   │   ├── arcface/                 # Face Recognition Model
│   │   ├── nanotrack/               # Object Tracking Model
│   │   └── CLIP/                    # Multimodal Model
│   └── NLP/                         # Natural Language Processing Examples
│       ├── spacemit_asr/            # Automatic Speech Recognition Module
│       ├── spacemit_llm/            # Large Language Model Module
│       ├── spacemit_tts/            # Text-to-Speech Module
│       ├── spacemit_audio/          # Audio Processing Module
│       └── *.py                     # Various AI Function Demo Scripts
└── README.md                        # Project Overview
```

## Example Categories

### Computer Vision (CV)

The computer vision module contains mainstream CV model examples, covering image classification, object detection, semantic segmentation, face recognition, and other tasks:

#### Image Classification
- **ResNet50**: Classic deep residual network for image classification tasks
- **EfficientNet**: Efficient convolutional neural network with good balance between accuracy and efficiency
- **MobileNetv2**: Lightweight network optimized for mobile devices
- **Inception**: Google's multi-scale feature extraction network
- **Swin Transformer**: Vision Transformer based on windowed attention

#### Object Detection
- **YOLOv5/v8/v11**: Latest YOLO series object detection algorithms
- **YOLOv6**: Efficient object detection algorithm proposed by Meituan
- **YOLOv8-pose**: Human pose detection model based on YOLOv8
- **YOLO-World**: Open-vocabulary object detection model

#### Semantic Segmentation
- **FCN**: Fully Convolutional Network, classic method for semantic segmentation
- **U-Net**: Classic network in medical image segmentation
- **SAM**: Segment Anything Model proposed by Meta

#### Face-Related
- **ArcFace**: Face recognition algorithm based on angular margin
- **YOLOv5-face**: YOLO variant specifically for face detection

#### Others
- **NanoTrack**: Lightweight object tracking algorithm
- **CLIP**: OpenAI's image-text multimodal understanding model

### Natural Language Processing (NLP)

The NLP module provides complete speech and text processing solutions, supporting the full workflow from speech input to intelligent responses:

#### Core Functions
- **Automatic Speech Recognition (ASR)**: Convert speech to text with real-time recognition support
- **Large Language Models (LLM)**: Support local deployment of mainstream models like Qwen, DeepSeek
- **Text-to-Speech (TTS)**: High-quality speech synthesis functionality
- **Function Calling**: LLM's ability to automatically select and call external functions
- **Vision Language Models**: Multimodal functionality for image understanding and text generation

#### Complete Application Scenarios
- **Voice Assistant**: Speech Input → Speech Recognition → LLM Inference → Speech Synthesis Output
- **Intelligent Dialogue**: Dialogue system with contextual understanding support
- **Image Q&A**: Intelligent Q&A based on image content

## Performance Metrics

Note: The following metrics are from the latest repository README. CV metrics exclude pre/post-processing time. ASR is tested on 2 cores; others use 4 cores unless otherwise specified.

### CV Model Performance (4 cores)

| Category | Model | Input Size | Data Type | FPS (4 cores) |
| :--: | :--: | :--: | :--: | :--: |
| EfficientNet | EfficientNet_b1 | [1,3,224,224] | int8 | 23 |
| Inception | Inception_v1 | [1,3,224,224] | int8 | 18 |
|  | Inception_v3 | [1,3,229,229] | int8 | 18 |
| MobileNet | MobileNetv2 | [1,3,224,224] | int8 | 67 |
| ResNet | ResNet50 | [1,3,224,224] | int8 | 25 |
| YOLOv5 | YOLOv5n | [1,3,640,640] | int8 | 8 |
| YOLOv6 | YOLOv6n | [1,3,320,320] | int8 | 54 |
| YOLOv8 | YOLOv8n | [1,3,320,320] | int8 | 73 |
|  | YOLOv8n | [1,3,192,320] | int8 | 106 |
| YOLOv11 | YOLOv11n | [1,3,320,320] | int8 | 35 |
| NanoTrack | NanoTrack | [1,3,255,255] | int8 | 55 |
| ArcFace | arcface_mobilefacenet | [1,3,320,320] | int8 | 27 |
| YOLOv5-face | YOLOv5n-face | [1,3,320,320] | int8 | 21 |

### LLM Model Performance (4 cores)

| Model | Parameters | Model Format | Quantization | Storage | Memory | Inference Speed (4 cores) | Purpose |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Qwen2.5-0.5B | 0.5b | gguf | q4_0 | 336MB | 480MB | 11 tokens/s | Chat |
| Deepseek R1-1.5B | 1.5b | gguf | q4_0 | 1017MB | 1.26GB | 4.5 tokens/s | Chat |
| Qwen2.5-0.5B-f16-agv-fc | 0.5b | gguf | q4_0 | 271MB | 480MB | 11 tokens/s | Function Calling |
| Qwen2.5-0.5B-f16-elephant-fc | 0.5b | gguf | q4_0 | 271MB | 480MB | 11 tokens/s | Function Calling |
| smollm2:135m | 135m | gguf | fp16 | 271MB | 490MB | 15 tokens/s | Chat |
| smollm2-135m-f16-agv-fc | 135m | gguf | fp16 | 270MB | 480MB | 15 tokens/s | Function Calling |
| smollm2-135m-q40-agv-fc | 135m | gguf | q4_0 | 77MB | 320MB | 30 tokens/s | Function Calling |

### ASR Model Performance

| Model | Parameters | Model Format | Quantization | Storage | Memory | Inference Speed (2 cores) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| sensevoice-small (python) | 330m | onnx | dynamic | 229MB | 430MB | RTF=0.38 |
| sensevoice-small (c++) | 330m | onnx | dynamic | 229MB | 360MB | RTF=0.3 |

### TTS Model Performance

| Model | Model Format | Quantization | Storage | Memory | Inference Speed (4 cores) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| melotts | onnx | dynamic | 74MB | 100MB |  RTF: 2-4 |
| matchtts | onnx | model-steps-3 dynamic | 73MB | 300MB | RTF: 0.64-0.75 |

## Quick Start

### 1. Clone Repository
```bash
git clone https://gitee.com/bianbu/spacemit-demo.git
cd spacemit-demo
```

### 2. Choose Example Type

**Computer Vision Examples**:
```bash
cd examples/CV/yolov5
# Download model and data
cd model && sh download_model.sh
cd ../data && sh download_data.sh
# Run Python example
cd ../python && python test_yolov5.py
```

**Natural Language Processing Examples**:
```bash
cd examples/NLP
# Install dependencies
pip install -r requirements.txt
# Run speech recognition example
python 03_asr_demo.py
```

## Use Cases

- **AI Learning**: Provides complete AI model deployment learning materials for beginners
- **Product Prototyping**: Quickly build AI feature prototypes and MVPs
- **Performance Evaluation**: Evaluate performance of different models on SpacemiT K1
- **Application Development**: Provides referenceable code templates for actual projects

## Technical Support

- **Official Documentation**: Detailed usage instructions and API documentation
- **Issue Reporting**: [Gitee Issues](https://gitee.com/bianbu/spacemit-demo/issues)
- **Community Discussion**: SpacemiT Developer Community

Visit the [SpacemiT AI Demo Source Repository](https://gitee.com/bianbu/spacemit-demo.git) now to start your AI development journey!
