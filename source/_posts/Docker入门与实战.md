---
title: Docker 入门与实战：打造你的第一个容器化应用
date: 2024-03-01
tags:
  - Docker
  - 容器
  - DevOps
cover: https://images.unsplash.com/photo-1605745341112-85968b19335b?w=800
---

Docker 已经成为现代软件开发的标配。本文从零开始，带你快速掌握 Docker 的核心概念和实战技巧。

## 一、Docker 核心概念

### 1. 镜像（Image）

镜像是一个只读的模板，用来创建容器。可以理解为面向对象的"类"。

```bash
# 拉取镜像
docker pull nginx:latest

# 查看镜像
docker images

# 删除镜像
docker rmi nginx:latest
```

### 2. 容器（Container）

容器是镜像的运行实例。可以理解为面向对象的"对象"。

```bash
# 创建并运行容器
docker run -d -p 8080:80 --name mynginx nginx

# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 停止容器
docker stop mynginx

# 删除容器
docker rm mynginx
```

### 3. 仓库（Repository）

存放镜像的地方，最常用的是 **Docker Hub**。

```bash
# 登录 Docker Hub
docker login

# 推送镜像
docker tag myapp:latest username/myapp:v1.0
docker push username/myapp:v1.0
```

## 二、Dockerfile 最佳实践

### 基础 Dockerfile

```dockerfile
# 使用官方镜像作为基础
FROM node:18-alpine

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装依赖
RUN npm install

# 复制源代码
COPY . .

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["node", "server.js"]
```

### 优化技巧

#### 1. 减少镜像层数

```dockerfile
# ❌ 不好：多个 RUN 命令创建多层
RUN apt-get update
RUN apt-get install -y nginx
RUN rm -rf /var/lib/apt/lists/*

# ✅ 好：合并成一个 RUN
RUN apt-get update && \
    apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/*
```

#### 2. 使用 .dockerignore

```
node_modules
.git
.env
README.md
```

#### 3. 多阶段构建

```dockerfile
# 构建阶段
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm run build

# 运行阶段
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

## 三、Docker Compose 编排

对于复杂应用，需要多个容器配合。使用 Docker Compose：

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - redis
      - db

  redis:
    image: redis:alpine
    volumes:
      - redis-data:/data

  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret

volumes:
  redis-data:
  db-data:
```

常用命令：

```bash
# 启动所有服务
docker compose up -d

# 查看日志
docker compose logs -f

# 停止所有服务
docker compose down
```

## 四、网络配置

### 1. 端口映射

```bash
# 主机端口:容器端口
docker run -p 8080:80 nginx
```

### 2. 网络模式

```bash
# bridge（默认）
docker run --network bridge nginx

# host（共享主机网络）
docker run --network host nginx

# none（无网络）
docker run --network none nginx
```

### 3. 自定义网络

```bash
# 创建网络
docker network create mynetwork

# 使用网络
docker run --network mynetwork -d nginx
```

## 五、数据持久化

### 1. 匿名卷

```bash
docker run -v /data nginx
```

### 2. 命名卷

```bash
# 创建卷
docker volume create mydata

# 使用卷
docker run -v mydata:/data nginx
```

### 3. 绑定挂载

```bash
# 挂载主机目录
docker run -v $(pwd):/app nginx
```

## 六、实战：部署 Node.js 应用

### 1. 项目结构

```
myapp/
├── Dockerfile
├── package.json
├── server.js
└── .dockerignore
```

### 2. Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

### 3. docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb

volumes:
  pgdata:
```

### 4. 启动

```bash
docker compose up -d --build
```

## 七、总结

| 概念 | 说明 |
|------|------|
| 镜像 | 模板，类 |
| 容器 | 实例，对象 |
| Dockerfile | 构建脚本 |
| Docker Compose | 编排工具 |

**核心命令**：
- `docker run` - 运行容器
- `docker build` - 构建镜像
- `docker compose` - 编排服务

---

*容器化是现代部署的标配，掌握 Docker 是每个后端开发者的必备技能。*
