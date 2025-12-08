---
sidebar_position: 3
---

# Ollama

**‌Ollama‌** is an open-source, cross-platform tool for local deployment of large language models (LLMs), engineered to simplify the execution, management, and inference workflows of LLMs in on-premises environments. It empowers users to deploy and invoke pre-trained models (e.g., LLaMA, DeepSeek) directly on personal devices (including PCs and edge servers) via simple CLI commands, eliminating dependencies on cloud services or high-end GPU hardware.

## Installation

```shell
sudo apt update
sudo apt install spacemit-ollama-toolkit
```

Verify installation:

```shell
ollama list
```

The final output of `NAME  ID  SIZE  MODIFIED` indicates a successful installation.

Verify version (ensure version 0.0.8 or above):

```shell
sudo apt show spacemit-ollama-toolkit
```

Confirm the version is 0.0.8 or above to support new model formats and direct pulling functionality.

## Pull models

### Method 1: Direct Pull (Recommended, requires version 0.0.8+)

Starting from spacemit-ollama-toolkit version 0.0.8, new support for **q4_K_M** and **q4_1** model formats has been added. You can directly use the `ollama pull` command to fetch q4_K_M format models from the official Ollama website with acceleration features:

```shell
# Directly pull q4_K_M format models (recommended)
ollama pull qwen3:0.6b
```

### Method 2: Manual Model Creation

To ensure maximum performance efficiency on the K1 development board, **q4_0** quantized models also perform very well. You can choose to manually download and create models by downloading GGUF-format models with q4_0 quantization precision from platforms like ModelScope or Hugging Face and transferring them to your K1 board or MuseBook device.

Below is a model creation example demonstrating the production workflow:

```shell
sudo apt install wget
wget https://huggingface.co/second-state/Qwen2.5-0.5B-Instruct-GGUF/blob/main/Qwen2.5-0.5B-Instruct-Q4_0.gguf ~/
wget https://archive.spacemit.com/spacemit-ai/modelfile/qwen2.5:0.5b.modelfile ~/
cd ~/
ollama create qwen2.5:0.5b -f qwen2.5:0.5b.modelfile
```

## Usage

```shell
# Run directly pulled models
ollama run qwen3:0.6b

# Or run manually created models
ollama run qwen2.5:0.5b
```
