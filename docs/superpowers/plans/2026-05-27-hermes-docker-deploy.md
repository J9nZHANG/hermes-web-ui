# Hermes Web UI Docker 部署实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在本地服务器部署 hermes-web-ui Docker 容器，通过 nginx 反向代理对外提供服务。

**Architecture:** Docker容器运行于端口8648，nginx反向代理将 `/hermes/` 路径映射到容器服务。

**Tech Stack:** Docker, docker-compose, nginx, Node.js v23.11.0

---

## 文件变更清单

| 文件 | 操作 | 说明 |
|------|------|------|
| `docker-compose.yml` | Modify | 端口配置从6060改为8648 |

---

### Task 1: 修改 docker-compose.yml 端口配置

**Files:**
- Modify: `docker-compose.yml`

- [ ] **Step 1: 读取当前 docker-compose.yml 内容**

确认当前端口配置为 6060。

- [ ] **Step 2: 修改端口映射**

将端口配置从 `6060` 改为 `8648`：

```yaml
services:
  hermes-webui:
    build:
      context: .
      dockerfile: Dockerfile
    image: ${WEBUI_IMAGE:-hermes-web-ui-local:latest}
    container_name: ${WEBUI_CONTAINER_NAME:-hermes-webui}
    ports:
      - "${PORT:-8648}:${PORT:-8648}"
      - "${XAI_OAUTH_PORT:-56121}:56121"
    volumes:
      - ${HERMES_DATA_DIR:-./hermes_data}:/home/agent/.hermes
      - ${HERMES_DATA_DIR:-./hermes_data}/hermes-web-ui:/home/agent/.hermes-web-ui
    environment:
      - PORT=${PORT:-8648}
      - HERMES_HOME=/home/agent/.hermes
      - HERMES_BIN=/opt/hermes/.venv/bin/hermes
      - HERMES_WEB_UI_MANAGED_GATEWAY=1
      - HERMES_WEB_UI_XAI_CALLBACK_BIND_HOST=0.0.0.0
      - PATH=/opt/hermes/.venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
      - HERMES_ALLOW_ROOT_GATEWAY=1
    restart: unless-stopped
    stdin_open: true
    tty: true
```

- [ ] **Step 3: 验证配置正确性**

检查修改后的文件内容。

---

### Task 2: 构建并启动 Docker 容器

**Files:**
- None (执行命令)

- [ ] **Step 1: 检查端口 8648 是否被占用**

```bash
netstat -tlnp | grep 8648 || echo "端口 8648 未被占用"
```

若端口被占用，需先停止占用进程。

- [ ] **Step 2: 构建并启动容器**

```bash
cd /root/projects/hermes-web-ui
docker-compose up --build -d
```

预期输出：容器构建并启动，最后显示 `Creating hermes-webui ... done`

- [ ] **Step 3: 确认构建成功**

检查容器运行状态：

```bash
docker ps | grep hermes-webui
```

预期输出：容器状态为 `Up`

---

### Task 3: 验证服务状态

**Files:**
- None (执行命令)

- [ ] **Step 1: 检查容器日志**

```bash
docker logs hermes-webui --tail 20
```

确认无启动错误。

- [ ] **Step 2: 测试本地端口连通性**

```bash
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8648/health
```

预期输出：HTTP 200

- [ ] **Step 3: 测试 API 接口**

```bash
curl -s http://127.0.0.1:8648/api/hermes/status | head -c 100
```

预期输出：返回 JSON 状态数据

---

### Task 4: 验证外部访问

**Files:**
- None (执行命令)

- [ ] **Step 1: 测试 nginx 代理**

```bash
curl -s -o /dev/null -w "%{http_code}" http://39.106.176.178/hermes/
```

预期输出：HTTP 200 或 301

- [ ] **Step 2: 测试完整访问路径**

```bash
curl -s http://39.106.176.178/hermes/ | grep -o '<title>.*</title>' | head -1
```

预期输出：返回页面标题，如 `<title>Hermes Web UI</title>`

- [ ] **Step 3: 确认 WebSocket 代理正常**

检查 socket.io 路径：

```bash
curl -s -o /dev/null -w "%{http_code}" http://39.106.176.178/socket.io/
```

预期输出：HTTP 200 或 400（socket.io 要求握手）

---

### Task 5: 验收确认

**Files:**
- None (执行命令)

- [ ] **Step 1: 确认容器持续运行**

```bash
docker ps -a | grep hermes-webui | grep -v Exited
```

预期输出：容器状态为 `Up`

- [ ] **Step 2: 确认数据目录持久化**

```bash
ls -la /root/projects/hermes-web-ui/hermes_data/
```

预期输出：存在 `hermes-web-ui` 子目录

- [ ] **Step 3: 完成部署**

部署完成，服务可通过 `http://39.106.176.178/hermes/` 访问。

---

## Self-Review 完成

- **Spec coverage**: 所有设计文档要求均已覆盖
- **Placeholder scan**: 无 TBD/TODO，所有步骤包含具体内容
- **Type consistency**: 无类型定义，纯部署任务