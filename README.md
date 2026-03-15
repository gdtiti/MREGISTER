# MREGISTER Web Console

- [English](#english)
- [中文](#中文)
- [快速跳转目录](#快速跳转目录)

## 快速跳转目录

### English

- [Project Layout](#project-layout)
- [Current Behavior](#current-behavior)
- [Authentication](#authentication)
- [Language Support](#language-support)
- [External API](#external-api)
- [Quick Start](#quick-start)
- [Docker Persistence](#docker-persistence)
- [Favicon](#favicon)
- [Security Notes](#security-notes)
- [Notes About the Workers](#notes-about-the-workers)
- [Suggested Next Improvements](#suggested-next-improvements)

### 中文

- [项目结构](#项目结构)
- [当前行为](#当前行为)
- [登录认证](#登录认证)
- [语言支持](#语言支持)
- [外部 API](#外部-api)
- [快速启动](#快速启动)
- [Docker 持久化](#docker-持久化)
- [Favicon](#favicon-1)
- [安全说明](#安全说明)
- [注册脚本说明](#注册脚本说明)
- [后续建议](#后续建议)

## English

`MREGISTER` is a lightweight web console that wraps the existing `openai-register` and `grok-register` Python workers into isolated, persistent tasks.

The web UI supports:

- Password-protected access
- Task creation, queueing, stop, download, and delete
- Independent task work directories and console logs
- Credential management for GPTMail and YesCaptcha
- Proxy management and default proxy selection
- Daily scheduled task creation
- API key management and external task API
- SQLite persistence across restarts
- Automatic UI language selection based on browser language
- Docker and `docker compose` deployment

## Project Layout

- `web_console/`
  - FastAPI web console
  - SQLite runtime database
  - static assets, templates, and Dockerfile
- `openai-register/`
  - OpenAI registration worker
- `grok-register/`
  - Grok registration worker

## Current Behavior

### Task model

Each created task is independent:

- Own runtime directory
- Own `console.log`
- Own result files
- Own downloadable zip archive

When a task finishes, you can:

- Open task detail
- Download the zip archive
- Delete the task record and files

### Real completion counting

Task quantity means real successful output count, not "attempt count".

The supervisor watches task output files and stops the worker only after the real completed count reaches the requested target quantity.

### Persistence

The web console stores persistent data in:

- Database: `web_console/runtime/app.db`
- Task files: `web_console/runtime/tasks/`

As long as `web_console/runtime/` is preserved, data will survive:

- service restarts
- container restarts
- machine reboots
- container rebuilds

## Authentication

The home page is protected by an admin password.

On first visit:

- the site shows a setup page
- you set the admin password
- the password is stored as a hash, not plaintext

After setup:

- every visit requires login
- the session uses an `HttpOnly` cookie

## Language Support

The UI supports:

- Simplified Chinese
- English

Language selection is automatic:

- if the browser prefers Chinese, the UI uses Chinese
- if the browser prefers English, the UI uses English

No manual language switcher is required right now.

## External API

The console exposes an external API for creating and querying tasks with an API key.

Current API flow:

1. Create an API key in the web UI
2. Call the external task creation endpoint
3. Query task status and completed count
4. Download the zip archive after the task finishes

API-created tasks support automatic cleanup:

- the task is deleted 24 hours after completion

## Quick Start

### Local Python

```bash
cd register-main
python -m pip install -r web_console/requirements.txt
uvicorn web_console.app:app --host 0.0.0.0 --port 8000
```

Open:

```text
http://localhost:8000
```

### Docker Compose

```bash
cd register-main
docker compose up -d --build
```

This setup builds the image locally from:

- build context: `.`
- Dockerfile: `web_console/Dockerfile`

## Docker Persistence

`docker-compose.yml` mounts the runtime directory to the host:

- host: `./web_console/runtime`
- container: `/app/web_console/runtime`

This is what keeps:

- SQLite data
- task outputs
- generated zip files

after container recreation.

## Favicon

To use your own site icon, place the file here:

```text
web_console/static/favicon.ico
```

The template already references:

```text
/static/favicon.ico
```

After replacing the icon, restart the service or rebuild the container if needed.

## Security Notes

Current security model is suitable for private deployment, but there are still limits:

- credentials are stored in SQLite
- there is no per-user permission system
- there is no rate limit on login attempts yet

If you plan to expose this publicly, you should at least add:

- HTTPS
- reverse proxy
- login rate limiting
- credential encryption

## Notes About the Workers

### `openai-register`

- Works well as a task-based worker
- Supports task-specific output directories
- Completion is controlled by the web supervisor

### `grok-register`

- Originally more interactive
- The web supervisor wraps it into the task model
- Completion is still based on real successful output count

## Suggested Next Improvements

- Change admin password in UI
- Backup and restore database
- Proxy pool rotation
- Login attempt throttling
- Optional manual language switcher

---

## 中文

`MREGISTER` 是一个轻量级 Web 控制台，用来把现有的 `openai-register` 和 `grok-register` Python 脚本封装成可持久化、可管理的独立任务。

当前 Web 控制台支持：

- 管理员密码保护
- 任务创建、排队、停止、下载、删除
- 每个任务独立工作目录和独立控制台日志
- GPTMail 与 YesCaptcha 凭据管理
- 代理管理与默认代理设置
- 每日定时任务
- API Key 管理与外部任务 API
- SQLite 持久化存储
- 按浏览器语言自动切换中英文界面
- Docker 与 `docker compose` 部署

## 项目结构

- `web_console/`
  - FastAPI Web 控制台
  - SQLite 运行时数据库
  - 静态资源、模板和 Dockerfile
- `openai-register/`
  - OpenAI 注册脚本
- `grok-register/`
  - Grok 注册脚本

## 当前行为

### 任务模型

每个任务都是独立的：

- 独立任务目录
- 独立 `console.log`
- 独立结果文件
- 独立下载压缩包

任务完成后，你可以：

- 打开任务详情
- 下载结果压缩包
- 删除任务记录和文件

### 真实完成数量

任务数量表示真实成功数量，不是“尝试次数”。

调度器会监控结果文件，只有当真实完成数量达到目标值时，才会结束任务。

### 持久化

Web 控制台把数据保存到：

- 数据库：`web_console/runtime/app.db`
- 任务文件：`web_console/runtime/tasks/`

只要 `web_console/runtime/` 目录没有被删除，下面这些情况都不会丢数据：

- 服务重启
- 容器重启
- 机器重启
- 容器重新构建

## 登录认证

首页受管理员密码保护。

首次打开时：

- 页面会先进入初始化密码页
- 你设置管理员密码
- 密码只会保存哈希，不保存明文

完成初始化后：

- 每次访问都需要登录
- 会话使用 `HttpOnly` Cookie

## 语言支持

当前界面支持：

- 简体中文
- English

语言选择是自动的：

- 浏览器偏好中文时，界面显示中文
- 浏览器偏好英文时，界面显示英文

目前还没有单独的手动语言切换器。

## 外部 API

控制台提供了外部 API，可以通过 API Key 创建和查询任务。

当前 API 流程：

1. 在 Web 页面创建 API Key
2. 调用外部任务创建接口
3. 查询任务状态和已完成数量
4. 任务完成后下载压缩包

API 创建的任务支持自动清理：

- 任务完成 24 小时后自动删除

## 快速启动

### 本地 Python 启动

```bash
cd register-main
python -m pip install -r web_console/requirements.txt
uvicorn web_console.app:app --host 0.0.0.0 --port 8000
```

打开：

```text
http://localhost:8000
```

### Docker Compose 启动

```bash
cd register-main
docker compose up -d --build
```

当前 Compose 会在本地构建镜像：

- build context：`.`
- Dockerfile：`web_console/Dockerfile`

## Docker 持久化

`docker-compose.yml` 已经把运行时目录挂载到宿主机：

- 宿主机：`./web_console/runtime`
- 容器内：`/app/web_console/runtime`

因此下面这些内容都会保留：

- SQLite 数据
- 任务输出文件
- 生成的压缩包

即使容器被重建也不会丢失。

## Favicon

如果你要替换站点图标，把文件放到这里：

```text
web_console/static/favicon.ico
```

模板已经引用了：

```text
/static/favicon.ico
```

替换后如果浏览器没有立即更新，重启服务或重新构建容器即可。

## 安全说明

当前安全模型适合私有部署，但仍然有边界：

- 凭据目前保存在 SQLite
- 还没有多用户权限系统
- 还没有登录限流

如果你准备公网部署，至少建议补这些：

- HTTPS
- 反向代理
- 登录限流
- 凭据加密

## 注册脚本说明

### `openai-register`

- 更适合任务化封装
- 支持任务专属输出目录
- 完成条件由 Web 调度器控制

### `grok-register`

- 原始脚本更偏交互式
- Web 调度器已经把它包装进任务模型
- 同样按真实成功数量判断完成

## 后续建议

- 在 UI 中支持修改管理员密码
- 增加数据库备份与恢复
- 增加代理池轮换
- 增加登录失败限流
- 增加手动语言切换器
