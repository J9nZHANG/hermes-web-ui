# Hermes Web UI Docker 部署设计

**日期**: 2026-05-27
**状态**: 待实施

## 目标

在本地服务器部署 hermes-web-ui Docker 容器，通过 nginx 反向代理对外提供服务，访问地址为 `http://39.106.176.178/hermes/`。

## 背景

### 项目信息
- **版本**: v0.6.3
- **基础镜像**: `nousresearch/hermes-agent:latest`
- **构建工具**: docker-compose

### 现有配置
- nginx 已运行，配置代理到端口 8648
- docker-compose.yml 默认端口 6060（需调整）
- dist 目录已构建

## 设计方案

### 端口配置
- **容器端口**: 8648（匹配现有 nginx 配置）
- **外部访问**: `http://39.106.176.178/hermes/`

### 实施步骤

**步骤一：修改 docker-compose.yml**
- 端口映射从 `6060:6060` 改为 `8648:8648`
- 环境变量 `PORT` 默认值改为 8648

**步骤二：构建并启动容器**
- 使用 `docker-compose up --build -d` 一键构建启动
- 构建基于 `nousresearch/hermes-agent:latest`
- 安装 Node.js v23.11.0
- 执行 npm build

**步骤三：验证服务状态**
- 检查容器运行状态
- 测试本地端口 `curl http://127.0.0.1:8648/health`

**步骤四：验证外部访问**
- 访问 `http://39.106.176.178/hermes/`
- 确认 Web UI 可正常加载

## 数据持久化

- `./hermes_data` 目录挂载到 `/home/agent/.hermes`
- `./hermes_data/hermes-web-ui` 目录挂载到 `/home/agent/.hermes-web-ui`

## 验收标准

1. Docker 容器持续运行（restart: unless-stopped）
2. 本地端口 8648 可访问健康检查接口
3. 外部可通过 nginx 代理访问 Web UI
4. 数据目录正确持久化

## 风险与应对

| 风险 | 应对措施 |
|------|---------|
| 构建时间过长 | 使用 --build 参数，耐心等待 |
| 基础镜像拉取失败 | 检查网络，必要时使用代理 |
| 端口冲突 | 确认 8648 端口未被占用 |