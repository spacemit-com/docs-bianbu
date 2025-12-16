---
sidebar_position: 6
---

# LocalAI

## Introduction

LocalAI is a complete AI stack for running AI models locally. Designed to be simple, efficient, and accessible, it provides an alternative to OpenAI's API. Users can run large language models (LLMs), image generation, speech transcription, and other AI tasks on consumer-grade hardware (including CPU environments) while maintaining data privacy and security.

This document details how to compile, install, and use LocalAI from source on our Bianbu platform, along with adding custom inference backends.

## Compilation and Installation Steps

### Install System Dependencies

```bash
sudo apt update
sudo apt install cmake golang libgrpc-dev make protobuf-compiler-grpc python3-grpc-tools

# Uninstall existing protobuf
sudo apt-get remove --purge protobuf-compiler libprotobuf-dev
sudo apt-get autoremove
sudo rm /usr/local/bin/protoc          # Remove executable
sudo rm -rf /usr/local/include/google  # Remove headers
sudo rm -rf /usr/local/lib/libproto*   # Remove libraries
sudo rm -rf /usr/lib/protoc            # Other possible paths

sudo apt-get install autoconf automake libtool curl make gcc-14 g++-14 unzip

# Switch to /usr/bin, delete symlinks for:
# gcc, g++, gcc-ar, gcc-nm, gcc-ranlib,
# riscv64-linux-gnu-gcc, riscv64-linux-gnu-gcc-ar, riscv64-linux-gnu-gcc-nm,
# riscv64-linux-gnu-gcc-ranlib, riscv64-linux-gnu-g++
# Then create new symlinks pointing to version 14
# Example:
# sudo rm /usr/bin/gcc
# sudo ln -s /usr/bin/gcc-14 /usr/bin/gcc

# Compile and install protobuf from source
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.20.3/protobuf-cpp-3.20.3.tar.gz
tar xvzf protobuf-cpp-3.20.3.tar.gz 
cd protobuf-3.20.3/cmake
cmake -DCMAKE_INSTALL_PREFIX=/usr/local .
cmake --build . --parallel 8
ctest --verbose
sudo cmake --install .
sudo ldconfig
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
cd ../../

sudo apt install libgrpc++-dev
```

### Compile LocalAI

Download our source code and compile:

```bash
wget https://archive.spacemit.com/spacemit-ai/localai/localai.tar.gz
tar xvzf localai.tar.gz

# Navigate to project directory
cd localai

# Add Go binary path to environment
export PATH=$PATH:$(go env GOPATH)/bin

# Use Aliyun proxy for faster module downloads
export GOPROXY=https://mirrors.aliyun.com/goproxy/,direct

# Compile
make build

# If errors occur, run make clean before rebuilding
```

## Adding Custom Inference Backends

### Add RISC-V Accelerated llama.cpp Backend

Our modify llama.cpp with RISC-V acceleration and package it as a grpc-server binary. Deploy using:

```bash
cd backend/cpp/spacemit-llama-cpp
bash install.sh
```

The script downloads the RISC-V accelerated llama-cpp-grpc-server binary and quantized models, configures directories, and generates config files.
Run **./local-ai --debug** from the project root to start LocalAI.
Test via browser at <http://localhost:8080/chat/>.

If encountering shared library error

    stderr llama-cpp-riscv-spacemit: error while loading shared libraries: libabsl_synchronization.so.20220623: No such file.

Fix with:

```bash
sudo apt install libabsl-dev
sudo ln -s /usr/lib/riscv64-linux-gnu/libabsl_synchronization.so /usr/lib/riscv64-linux-gnu/libabsl_synchronization.so.20220623
```

To use other LLM models with our accelerated backend, you can

- Download models from <https://archive.spacemit.com/spacemit-ai/gguf/>
- Get corresponding modelfiles from <https://archive.spacemit.com/spacemit-ai/modelfile/>
- Create a new config file (e.g., models/spacemit-qwen2.5-0.5b-instruct.yaml), adjusting model name, stop words, and template based on the modelfile.

### Add RISC-V Accelerated ASR Backend

```bash
cd backend/cpp/spacemit-asr-cpp
bash build.sh

cd ../../../
./local-ai --debug
```

Test with:

```bash
# Prepare audio file test.wav
curl -X POST http://localhost:8080/v1/audio/transcriptions \
    -H "Content-Type: multipart/form-data" \
    -F "file=@test.wav" \
    -F "model=sensevoicesmall-cpp"
```

### Add C++ TTS Backend

```bash
cd backend/cpp/matcha-tts-cpp
bash build.sh

cd ../../../
./local-ai --debug
```

Test with:

```bash
curl -X POST "http://localhost:8080/tts" \
    -H "Content-Type: application/json" \
    -d '{"input":"Hello, how is the weather today","model":"matcha-tts-cpp"}' \
    -o output.wav 
```
