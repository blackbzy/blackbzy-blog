---
title: 开发_06_容器与部署
description: 自动化相关
date: 2026-03-29
categories:
  - develop
tags:
  - develop
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
image:
  path: tech/docker.png
  alt: docker
---

> 自动化是未来
{: .prompt-info }


## Kubernetes
一个**容器编排平台**，专门用来自动部署、扩展和管理跨主机集群的容器化应用。

## [Docker](https://www.docker.com/)
容器化：中间键的实现和搭建

```
image
container
volume
network

```

### Docker常用命令
1. 容器生命周期管理
   这是最常用的部分，用于启动、停止和查看容器。
- **`docker run`**: 创建并启动一个容器。
  - `docker run -it <image> /bin/bash`: 交互式启动。
  - `docker run -d <image>`: 后台（守护进程）运行。
  - `docker run -p 8080:80 <image>`: 端口映射（宿主机端口:容器端口）。
- **`docker ps`**: 列出运行中的容器。
  - `docker ps -a`: 列出所有容器（包括已停止的）。
- **`docker stop <id/name>`**: 停止运行中的容器。
- **`docker start <id/name>`**: 启动已停止的容器。
- **`docker restart <id/name>`**: 重启容器。
- **`docker rm <id/name>`**: 删除容器（加 `-f` 可强制删除运行中的容器）。

1. 镜像管理

镜像相当于容器的“模板”。
- **`docker images`**: 查看本地所有镜像。
- **`docker pull <image_name>`**: 从仓库拉取镜像（如 `docker pull nginx`）。
- **`docker rmi <image_id>`**: 删除本地镜像。
- **`docker build -t <name>:<tag> .`**: 使用当前目录的 Dockerfile 构建镜像。
- **`docker tag <image_id> <new_name>:<tag>`**: 给镜像打标签。

3. 容器运维与调试

|**命令**|**用途**|
|---|---|
|**`docker exec -it <id> bash`**|**最常用**：进入正在运行的容器终端。|
|**`docker logs -f <id>`**|查看容器日志（`-f` 类似 `tail -f` 持续输出）。|
|**`docker inspect <id>`**|获取容器的元数据（配置、IP 地址等详情）。|
|**`docker cp <path> <id>:<path>`**|在宿主机和容器之间拷贝文件。|
|**`docker stats`**|实时显示容器的资源消耗（CPU、内存、网络）。|
4. 清理命令

Docker 用久了会产生很多垃圾，可以用这些命令“一键瘦身”
- **`docker system prune`**: 删除所有已停止的容器、未使用的网络和悬空镜像。
- **`docker system prune -a`**: 更加彻底，连未被使用的镜像也一起删掉。

5. Docker Compose (多容器编排)

如果你使用 `docker-compose.yml` 部署项目必会的：
- **`docker-compose up -d`**: 后台启动配置文件中定义的所有服务。
- **`docker-compose down`**: 停止并删除容器、网络、卷和镜像。
- **`docker-compose ps`**: 查看当前编排项目的状态。


## [Jenkins](https://www.jenkins.io/)
- pipeline
- CI/CD
### Jenkins操作
1. 基础管理操作
   在正式跑任务之前，要先配置好。
- **安装插件 (Manage Plugins)：** 这是 Jenkins 的灵魂。路径：`Manage Jenkins` -> `Plugins`。必装推荐：_Git, Docker, Pipeline, Blue Ocean, Credentials Binding_。
- **凭据管理 (Credentials)：** 用于存储 GitHub 令牌、服务器 SSH 密码或 Docker 仓库登录信息。
- **全局工具配置 (Global Tool Configuration)：** 配置 JDK、Git、Maven 或 NodeJS 的路径。建议勾选“Install automatically”。
1. 任务（Job）常用操作
- **源码管理：** 填写 Git 仓库地址，选择对应的 Credentials（凭据）。
- **构建触发器：** * `Poll SCM`: 定期检查代码更新。
  - `GitHub hook trigger`: 代码一提交就触发构建（最常用）。
- **构建步骤 (Build Steps)：** * `Execute shell`: 编写 Linux 命令（如 `npm install` 或 `mvn clean package`）。
- **构建后操作：** 发送邮件通知、归档生成的 `.jar` 或 `.war` 包。
1. Pipeline (流水线) 核心语法
   现代 Jenkins 推荐使用 `Jenkinsfile`。这里有两种风格，建议使用 **Declarative（声明式）**，结构更清晰：
   Groovy

```
pipeline {
    agent any // 在任何可用的节点上运行
    
    stages {
        stage('Checkout') { // 拉取代码
            steps {
                git 'https://github.com/user/repo.git'
            }
        }
        stage('Build') { // 编译
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') { // 测试
            steps {
                sh 'make check'
            }
        }
        stage('Deploy') { // 部署
            steps {
                sh './deploy.sh'
            }
        }
    }
    post { // 无论成功失败都会执行
        always {
            echo 'I am always running!'
        }
    }
}
```

4. 运维常用快捷操作
   当 Jenkins 出现卡顿或需要调整时：

| **场景**       | **操作/命令**                            |
| ------------ | ------------------------------------ |
| **安全重启**     | 在 URL 后加上 `/safeRestart`（等待任务结束后重启）。 |
| **强制重新加载配置** | 在 URL 后加上 `/reload`。                 |
| **查看日志**     | `Manage Jenkins` -> `System Log`。    |
| **构建队列管理**   | 在左侧面板查看 Build Queue，点击小红叉可以取消排队的任务。  |
5. 常用环境变量
   在 Shell 脚本或 Pipeline 中，可以直接调用这些变量：
- **`${BUILD_NUMBER}`**: 当前构建的版本号（如：12）。
- **`${JOB_NAME}`**: 项目名称。
- **`${WORKSPACE}`**: 项目存放的绝对路径。
- **`${GIT_BRANCH}`**: 当前正在构建的分支。

避坑指南：
5. **权限问题：** 如果 Jenkins 执行 shell 报错 `Permission denied`，通常是因为 Jenkins 用户（默认是 `jenkins`）没有执行该目录或 Docker 命令的权限。
6. **丢弃旧的构建：** 在 Job 配置里勾选 **"Discard old builds"**
## Linux
![[07-Linux]]

## 云原生
- 主流服务器的配置开发

---
故事未完:96
**Thoughts**:: justdoit.
