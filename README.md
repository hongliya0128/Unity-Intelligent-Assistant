# Unity Intelligent Assistant

本仓库是课程项目 **Unity开发智能适配助手** 的主仓库。

当前仓库已经按项目分层整理，避免前端源码、后端适配层和项目文档全部堆在同一层目录。

## 目录结构

```text
.
├─ frontend/
│  └─ open-webui/          # 基于 Open WebUI 的前端与原始源码定制
├─ backend/                # 后续放 Dify 适配层或项目自研后端
├─ docs/                   # 部署文档、设计说明、接口说明
└─ README.md
```

## 当前进度

- 已完成 Open WebUI 本地化品牌定制
- 已完成 Unity 场景化首页改造
- 已补充部署文档，方便组员从 0 开始在本地运行
- 已预留 `backend/` 用于后续接入 Dify 兼容适配层

## 快速开始

如果你是第一次拉取项目，建议按下面顺序看：

1. 阅读 [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md)
2. 进入前端目录：

```powershell
cd frontend/open-webui
```

3. 按部署文档启动：
- Docker 后端
- 前端开发环境

## 说明

### 前端源码位置

当前实际运行和修改的前端代码在：

```text
frontend/open-webui
```

### 后端适配层位置

如果后续需要接：

- Dify 工作流
- OpenAI 兼容代理
- 项目自己的 API

建议统一放在：

```text
backend
```

### 文档位置

部署、汇报、接口设计等说明建议统一放在：

```text
docs
```
