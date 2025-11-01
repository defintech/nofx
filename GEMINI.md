# GEMINI.md

## 项目概述

本项目是一个名为 NOFX 的人工智能驱动的加密货币期货自动交易竞赛系统。它旨在支持在多个加密货币交易所（包括币安、Hyperliquid 和 Aster DEX）上进行自动交易。该系统可以利用不同的 AI 模型（如 DeepSeek 和 Qwen）来做出交易决策。

该项目由 Go 后端和 React 前端组成。后端负责核心交易逻辑、API 服务以及与 AI 模型的通信。前端提供一个基于 Web 的仪表板，用于监控交易活动、查看性能指标和管理系统。

该项目使用 Docker 进行容器化，推荐的运行方式是通过 Docker Compose。

## 构建和运行

推荐使用 Docker 来构建和运行本项目。

### Docker 一键部署

1.  **准备配置**：
    ```bash
    cp config.json.example config.json
    ```
    然后，编辑 `config.json` 文件，填入您的 API 密钥和其他配置。

2.  **启动系统**：
    ```bash
    ./start.sh start --build
    ```
    或者，您也可以直接使用 `docker-compose`：
    ```bash
    docker compose up -d --build
    ```

3.  **访问仪表板**：
    打开浏览器并访问 `http://localhost:3000`。

### 手动安装（适用于开发人员）

如果您想在没有 Docker 的情况下运行该项目，您需要安装 Go 1.21+ 和 Node.js 18+。

1.  **安装依赖**：
    *   **后端**：`go mod download`
    *   **前端**：`cd web && npm install`

2.  **运行系统**：
    *   **后端**：`./nofx`
    *   **前端**：`cd web && npm run dev`

## Kubernetes 部署

项目已集成到 `trading-infra` GitOps 仓库，使用 FluxCD 自动部署到 nofx-test 命名空间。

### 配置文件位置

-   **部署清单**：`trading-infra/apps/nofx/`
-   **Secret配置**：`trading-infra/secrets/nofx-backend-config.yaml`（SOPS加密）
-   **GHCR认证**：`trading-infra/secrets/nofx-ghcr-secret.yaml`（SOPS加密）

### 配置更新

修改 `apps/nofx/secret.sops.yaml` 后需重新加密：
```bash
cd trading-infra
sops --encrypt --in-place apps/nofx/secret.sops.yaml
git add apps/nofx/secret.sops.yaml
git commit -m "chore(nofx): update backend config"
git push
```

### 部署验证

```bash
# 查看部署状态
kubectl get all -n nofx-test

# 查看日志
kubectl logs -n nofx-test deployment/nofx-backend --tail=100
kubectl logs -n nofx-test deployment/nofx-frontend --tail=100
```

## 开发规范

*   **后端**：后端使用 Go 语言编写。它使用 Gin 框架作为 API 服务器，并与交易所和 AI 模型进行通信。
*   **前端**：前端是一个使用 TypeScript 编写的 React 应用程序。它使用 `swr` 进行数据获取，使用 `recharts` 进行图表绘制。
*   **样式**：前端使用 Tailwind CSS 进行样式设计，采用受币安启发的深色主题。
*   **容器化**：该项目使用 Docker 进行容器化，为后端和前端提供了单独的 Dockerfile。
*   **配置**：系统通过 `config.json` 文件进行配置（K8s部署中通过Secret挂载）。
*   **API**：后端为前端提供 RESTful API。
*   **日志**：系统为每个交易员生成决策日志，存储在 `decision_logs` 目录中。