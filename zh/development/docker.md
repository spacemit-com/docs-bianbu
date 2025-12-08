---
sidebar_position: 5
---

# Docker 使用指南

## 简介

Docker 是一个开源的平台，旨在简化应用程序的开发、部署和运行。Docker 使用容器技术来实现这一目标，容器是一个轻量级、独立的运行环境。

## 安装 Docker

```bash
sudo apt-get update
sudo apt-get install -y docker.io docker-buildx
```

## 基本命令

### 检查 Docker 版本

```bash
sudo docker --version
```

### 拉取镜像

```bash
sudo docker pull <镜像名>
```

### 运行容器

```bash
sudo docker run -it <镜像名>
```

### 列出运行中的容器

```bash
sudo docker ps
```

### 停止容器

```bash
sudo docker stop <容器ID>
```

## 构建镜像

### 创建 Dockerfile

```Dockerfile
# Dockerfile
FROM harbor.spacemit.com/bianbu/bianbu:latest
RUN apt-get update && apt-get install -y python3
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]
```

[Dockerfile语法详细介绍](https://docs.docker.com/reference/dockerfile/)

请确保在项目目录中包含一个名为 `app.py` 的文件，该文件将作为容器启动时执行的入口点。

```python
# app.py
def main():
    print("Hello, Docker!")

if __name__ == "__main__":
    main()
```

### 构建镜像

```bash
sudo docker buildx build -t myapp --load -f Dockerfile . 
```

### 运行

```bash
sudo docker run --rm myapp
```

可以看到输出了
`Hello, Docker!`

## 管理容器

### 查看所有容器

```bash
docker ps -a
```

### 删除容器

```bash
docker rm <容器ID>
```

### 删除镜像

```bash
docker rmi <镜像ID>
```

## 常见问题

### 如何清理未使用的镜像和容器？

```bash
docker system prune
```

### 如何查看容器日志？

```bash
docker logs <容器ID>
```

### 如何进入运行中的容器？

```bash
docker exec -it <容器ID> /bin/bash
```
