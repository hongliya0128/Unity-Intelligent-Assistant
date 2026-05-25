# Unity开发智能适配助手部署说明

本项目主仓库已经按功能拆分目录：

```text
Unity-Intelligent-Assistant/
├─ frontend/
│  └─ open-webui/
├─ backend/
├─ docs/
└─ README.md
```

其中：

- `frontend/open-webui`：当前前端项目与 Open WebUI 定制代码
- `backend`：预留给后续 Dify 适配层或项目自己的后端
- `docs`：部署说明、设计文档、接口说明

本文档默认面向 Windows 同学，从 0 开始部署。

## 1. 准备环境

### 1.1 安装 Git

下载地址：

https://git-scm.com/download/win

安装后执行：

```powershell
git --version
```

### 1.2 安装 Docker Desktop

下载地址：

https://www.docker.com/products/docker-desktop/

安装建议：

- 安装时按提示启用 `WSL 2`
- 安装完成后重启电脑
- 打开 Docker Desktop，左下角显示 `Engine running`

验证：

```powershell
docker --version
```

### 1.3 安装 Node.js

建议安装 Node.js `20.x` 或 `22.x` LTS。

下载地址：

https://nodejs.org/

验证：

```powershell
node -v
npm -v
```

### 1.4 Python

如果当前只做前端修改，不一定马上需要 Python。

如果后续要源码运行 Open WebUI 后端，建议安装：

- Python `3.11+`

下载地址：

https://www.python.org/downloads/

## 2. 获取项目源码

在你想存放项目的目录打开 PowerShell，执行：

```powershell
git clone https://github.com/hongliya0128/Unity-Intelligent-Assistant.git
cd Unity-Intelligent-Assistant
```

## 3. 进入前端项目目录

真正需要运行的前端项目在：

```powershell
cd frontend/open-webui
```

后面的安装依赖、启动前端，都在这个目录里执行。

## 4. 安装前端依赖

```powershell
npm install
```

## 5. 启动后端

当前推荐部署方式：

- 前端：源码开发模式
- 后端：Docker 容器模式

在任意 PowerShell 窗口执行：

```powershell
docker run -d -p 8080:8080 -v open-webui-dev:/app/backend/data --name open-webui-dev ghcr.io/open-webui/open-webui:main
```

说明：

- `8080:8080`：把容器中的后端映射到本机 `8080`
- `open-webui-dev:/app/backend/data`：保存数据库和配置
- `--name open-webui-dev`：给容器命名，方便管理

检查是否启动成功：

```powershell
docker ps
```

如果能看到 `open-webui-dev`，说明后端已启动。

## 6. 启动前端开发环境

在 `frontend/open-webui` 目录执行：

```powershell
npm run dev
```

启动成功后，终端会显示类似：

```text
Local:   http://localhost:5173/
```

浏览器打开：

```text
http://localhost:5173
```

## 7. 首次进入系统

第一次打开时注册管理员账号即可。

当前已完成的主要前端定制包括：

- 品牌名称替换为 `Unity开发智能适配助手`
- 欢迎区与项目图标替换
- Unity 场景化主页卡片
- 场景模式切换与自由提问模式
- 设置页中的 Dify / OpenAI 兼容接口配置说明

## 8. 接口配置说明

进入系统后：

`设置 -> 外部连接`

当前前端已经把这里整理成项目语境的配置入口。

主要配置项：

- `API Base URL`
- `API Key`

### 重要说明

Open WebUI 这里底层走的是：

```text
/chat/completions
```

所以：

- 如果你的后端提供的是 **OpenAI 兼容接口**，可以直接接入
- 如果你手里只有 **Dify 原生 API**
  - `/chat-messages`
  - `/completion-messages`

通常 **不能直接填进这里**

还需要一个：

- Dify -> OpenAI 兼容适配层

也就是说：

- 前端接口配置页已经准备好
- 但后端能否真正联通，取决于你们是否已经准备兼容接口

## 9. 常用命令

### 9.1 查看后端容器

```powershell
docker ps
```

### 9.2 停止后端容器

```powershell
docker stop open-webui-dev
```

### 9.3 启动已有后端容器

```powershell
docker start open-webui-dev
```

### 9.4 删除后端容器

```powershell
docker rm -f open-webui-dev
```

### 9.5 重新安装前端依赖

```powershell
cd frontend/open-webui
npm install
```

### 9.6 启动前端开发服务器

```powershell
cd frontend/open-webui
npm run dev
```

## 10. 常见问题

### 10.1 打开 `http://localhost:5173` 后白屏或一直加载

通常原因：

- 前端启动了
- 但后端 `8080` 没有启动

先检查：

```powershell
docker ps
```

### 10.2 PowerShell 提示 `docker` 不是内部或外部命令

说明 Docker Desktop 没安装好，或者装完后没有重启终端/电脑。

### 10.3 页面能打开，但顶部提示没有可用模型

说明界面已经启动，但还没有接入可用模型或兼容接口。

### 10.4 修改代码后页面没变化

先尝试：

- 浏览器强制刷新：`Ctrl + F5`

如果还不行，重新执行：

```powershell
cd frontend/open-webui
npm run dev
```

## 11. 推荐协作方式

- 普通演示同学：启动 Docker 后端并访问页面
- 前端同学：在 `frontend/open-webui` 下运行 `npm run dev`
- 后端同学：在 `backend/` 下处理 Dify 兼容适配层

## 12. 重点目录

前端主要目录：

```text
frontend/open-webui/src
```

静态资源目录：

```text
frontend/open-webui/static
```

如果要继续改主页、侧栏、欢迎区，优先关注：

- `frontend/open-webui/src/lib/components/chat/Placeholder.svelte`
- `frontend/open-webui/src/lib/components/layout/Sidebar.svelte`
- `frontend/open-webui/src/lib/components/chat/ModelSelector.svelte`
- `frontend/open-webui/src/lib/components/admin/Settings/Connections.svelte`

## 13. 部署完成后的访问地址

- 前端开发页面：
  - `http://localhost:5173`
- 后端服务：
  - `http://localhost:8080`
