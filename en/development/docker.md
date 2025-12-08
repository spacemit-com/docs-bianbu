---
sidebar_position: 5
---

# Docker User Guide

## Introduction

Docker is an open-source platform designed to simplify the development, deployment, and running of applications. Docker uses container technology to achieve this goal. A container is a lightweight, standalone runtime environment.

## Installing Docker

```bash
sudo apt-get update
sudo apt-get install -y docker.io docker-buildx
```

## Basic Commands

### Check Docker Version

```bash
sudo docker --version
```

### Pull an Image

```bash
sudo docker pull <image_name>
```

### Run a Container

```bash
sudo docker run -it <image_name>
```

### List Running Containers

```bash
sudo docker ps
```

### Stop a Container

```bash
sudo docker stop <container_id>
```

## Building an Image

### Create a Dockerfile

```Dockerfile
# Dockerfile
FROM harbor.spacemit.com/bianbu/bianbu:latest
RUN apt-get update && apt-get install -y python3
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]
```

[Detailed Dockerfile Syntax](https://docs.docker.com/reference/dockerfile/)

Make sure to include a file named `app.py` in your project directory, which will be executed as the entry point when the container starts.

```python
# app.py
def main():
    print("Hello, Docker!")

if __name__ == "__main__":
    main()
```

### Build the Image

```bash
sudo docker buildx build -t myapp --load -f Dockerfile . 
```

### Run

```bash
sudo docker run --rm myapp
```

You should see the output
`Hello, Docker!`

## Managing Containers

### View All Containers

```bash
docker ps -a
```

### Remove a Container

```bash
docker rm <container_id>
```

### Remove an Image

```bash
docker rmi <image_id>
```

## Common Issues

### How to Clean Up Unused Images and Containers?

```bash
docker system prune
```

### How to View Container Logs?

```bash
docker logs <container_id>
```

### How to Enter a Running Container?

```bash
docker exec -it <container_id> /bin/bash
```
