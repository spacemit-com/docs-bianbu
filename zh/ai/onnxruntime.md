---
sidebar_position: 1
---

# ONNX Runtime

[ONNX Runtime](https://onnxruntime.ai/) 是一个跨平台的推理和训练机器学习加速器。为了让应用获得更快的推理速度，我们开发了 SpaceMIT Execution Provider，它会使用扩展 AI 指令进行加速，只需在开发应用时指定 `SpaceMITExecutionProvider`。

## 安装

```shell
sudo apt-get update
sudo apt-get install -y onnxruntime python3-spacemit-ort libopencv-dev
```

验证是否安装成功：

```shell
python3
>>> import onnxruntime as ort
>>> import spacemit_ort
>>> available_providers = ort.get_available_providers()
>>> print(available_providers)
['SpaceMITExecutionProvider', 'CPUExecutionProvider']
```

最后输出 `['SpaceMITExecutionProvider', 'CPUExecutionProvider']` 表示安装成功。

## 使用 Python API

### 安装依赖

```shell
sudo apt-get install -y python3-onnx python3-pillow python3-matplotlib python3-opencv
```

### 下载模型、标签和图片

我们提供量化好的开源模型，可从 [ModelZoo](https://archive.spacemit.com/spacemit-ai/ModelZoo/) 下载。理论上，[ONNX Model Zoo] 的模型都可以推理，但性能比我们量化的差点。

```shell
wget https://archive.spacemit.com/spacemit-ai/ModelZoo/classification.tar.gz
tar -zxf classification.tar.gz
wget -P classification https://archive.spacemit.com/spacemit-ai/ModelZoo/classification/kitten.jpg
wget -P classification https://archive.spacemit.com/spacemit-ai/ModelZoo/classification/synset.txt
```

### 导入依赖

确保下面依赖都安装，如果没有错误则继续，警告可忽略。

```python
import onnx
import numpy as np
import onnxruntime as ort
import spacemit_ort
from PIL import Image
import cv2
import matplotlib.pyplot as plt
import time
```

### 读图片

```python
def get_image(path, show=False):
    with Image.open(path) as img:
        img = np.array(img.convert('RGB'))
    if show:
        plt.imshow(img)
        plt.axis('off')
    return img
```

### 图片预处理

预处理推理图像 -> 缩放到0~1，调整大小到256x256，取224x224的中心裁剪，标准化图像，转置为NCHW格式，转换为float32并添加一个维度以批处理图像。

```python
def preprocess(img):
    img = img / 255.
    img = cv2.resize(img, (256, 256))
    h, w = img.shape[0], img.shape[1]
    y0 = (h - 224) // 2
    x0 = (w - 224) // 2
    img = img[y0 : y0+224, x0 : x0+224, :]
    img = (img - [0.485, 0.456, 0.406]) / [0.229, 0.224, 0.225]
    img = np.transpose(img, axes=[2, 0, 1])
    img = img.astype(np.float32)
    img = np.expand_dims(img, axis=0)
    return img
```

### 预测

```python
def predict(path):
    img = get_image(path, show=True)
    img = preprocess(img)
    ort_inputs = {session.get_inputs()[0].name: img}
    start = time.time()
    preds = session.run(None, ort_inputs)[0]
    end = time.time()
    preds = np.squeeze(preds)
    a = np.argsort(preds)[::-1]
    print('time=%.2fms; class=%s; probability=%f' % (round((end-start) * 1000, 2), labels[a[0]], preds[a[0]]))
```

### 导入模型、标签并推理

```python
with open('classification/synset.txt', 'r') as f:
    labels = [l.rstrip() for l in f]

# Set the number of inference threads
session_options = ort.SessionOptions()
session_options.intra_op_num_threads = 2

# IMPORTANT: Specified SpaceMITExecutionProvider providers
session = ort.InferenceSession('classification/resnet50/resnet50.q.onnx', sess_options=session_options, providers=["SpaceMITExecutionProvider"])

img_path = 'classification/kitten.jpg'
predict(img_path)
```

结果：

```shell
time=95.15ms; class=n02123045 tabby, tabby cat; probability=13.498201
```

### API Overview

更多 API 请查阅官方 [API Overview](https://onnxruntime.ai/docs/api/python/api_summary.html)。
