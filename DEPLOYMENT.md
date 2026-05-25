# Unity开发智能适配助手部署说明

本项目基于 `open-webui` 二次开发，适用于本地演示、前端修改和课程项目验收。

本文档默认面向 Windows 同学，从 0 开始部署。

## 1. 准备环境

### 1.1 安装 Git

下载地址：

https://git-scm.com/download/win

安装完成后，在 PowerShell 中执行：

```powershell
git --version
```

如果能看到版本号，说明安装成功。

### 1.2 安装 Docker Desktop

下载地址：

https://www.docker.com/products/docker-desktop/

安装建议：

- 安装时按提示启用 `WSL 2`
- 安装完成后重启电脑
- 打开 Docker Desktop，左下角出现 `Engine running` 表示可用

验证：

```powershell
docker --version
```

### 1.3 安装 Node.js

建议安装 Node.js `20.x` 或 `22.x` LTS 版本。

下载地址：

https://nodejs.org/

验证：

```powershell
node -v
npm -v
```

### 1.4 Python

如果只做前端修改，Python 不是必须马上配置。

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

如果仓库目录名不是这个，请以实际下载后的目录名为准。

## 3. 安装前端依赖

进入项目根目录后执行：

```powershell
npm install
```

这一步会安装前端运行所需依赖，首次安装会稍慢。

## 4. 启动后端

本项目推荐：

- 前端：源码开发模式
- 后端：Docker 容器模式

这样最适合课程项目演示和界面修改。

在 PowerShell 中执行：

```powershell
docker run -d -p 8080:8080 -v open-webui-dev:/app/backend/data --name open-webui-dev ghcr.io/open-webui/open-webui:main
```

说明：

- `8080:8080`：把容器里的 Open WebUI 后端映射到本机 `8080`
- `open-webui-dev:/app/backend/data`：保存数据库和配置，避免数据丢失
- `--name open-webui-dev`：给容器命名，方便后续管理

检查容器是否成功启动：

```powershell
docker ps
```

如果能看到 `open-webui-dev`，说明后端已启动。

## 5. 启动前端开发环境

在项目根目录重新打开一个 PowerShell 窗口，执行：

```powershell
npm run dev
```

启动成功后，终端会显示类似：

```text
Local:   http://localhost:5173/
```

在浏览器打开：

```text
http://localhost:5173
```

## 6. 首次进入系统

第一次打开时需要注册管理员账号。

注册完成后即可进入系统主页。

当前项目已完成的主要前端定制包括：

- 品牌名称替换为 `Unity开发智能适配助手`
- 首页欢迎区与项目图标替换
- Unity 场景化主页卡片
- 场景模式切换与自由提问模式
- 设置页中的 Dify / OpenAI 兼容接口配置说明

## 7. 接口配置说明

### 7.1 当前前端配置入口在哪里

进入系统后：

`设置 -> 外部连接`

当前我们已经把该区域改造成更贴近项目的配置说明。

### 7.2 要填什么

主要配置项：

- `API Base URL`
- `API Key`

### 7.3 重要说明

Open WebUI 这套连接配置底层走的是：

```text
/chat/completions
```

因此：

- 如果你的后端提供的是 **OpenAI 兼容接口**，可以直接接
- 如果你手里只有 **Dify 原生 API**，例如：
  - `/chat-messages`
  - `/completion-messages`

那么通常 **不能直接填进这里**

还需要一个：

- Dify -> OpenAI 兼容适配层

也就是说：

- 当前前端配置页已经准备好
- 但后端是否能真正连通，取决于你们有没有准备兼容接口

## 8. 常用命令

### 8.1 查看后端容器

```powershell
docker ps
```

### 8.2 停止后端容器

```powershell
docker stop open-webui-dev
```

### 8.3 启动已存在的后端容器

```powershell
docker start open-webui-dev
```

### 8.4 删除后端容器

```powershell
docker rm -f open-webui-dev
```

### 8.5 重新安装前端依赖

```powershell
npm install
```

### 8.6 启动前端开发服务器

```powershell
npm run dev
```

## 9. 常见问题

### 9.1 打开 `http://localhost:5173` 后白屏或一直加载

通常原因：

- 前端启动了
- 但后端 `8080` 没有启动

先检查：

```powershell
docker ps
```

### 9.2 PowerShell 提示 `docker` 不是内部或外部命令

说明 Docker Desktop 没安装好，或者装完后没有重启终端/电脑。

### 9.3 页面能打开，但顶部提示没有可用模型

说明系统界面已启动，但还没有接入可用模型或兼容接口。

这不是前端报错，而是后端模型连接未完成。

### 9.4 修改前端代码后页面没变化

先尝试：

- 浏览器强制刷新：`Ctrl + F5`

如果仍无变化，重新执行：

```powershell
npm run dev
```

## 10. 推荐协作方式

建议组员按下面方式协作：

- 普通演示同学：只需要 Docker 启动后端 + 浏览器访问
- 前端同学：运行 `npm run dev`，直接改 `src/` 下的页面
- 后端对接同学：重点处理 Dify 接口兼容层

## 11. 当前目录结构建议关注

前端主要目录：

```text
src/
```

静态资源目录：

```text
static/
```

如果要继续改主页、侧栏、欢迎区，优先关注：

- `src/lib/components/chat/Placeholder.svelte`
- `src/lib/components/layout/Sidebar.svelte`
- `src/lib/components/chat/ModelSelector.svelte`
- `src/lib/components/admin/Settings/Connections.svelte`

## 12. 部署完成后的访问地址

- 前端开发页面：
  - `http://localhost:5173`
- 后端服务：
  - `http://localhost:8080`

如果只看到前端，不能正常问答，优先检查后端容器和接口配置。
