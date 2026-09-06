<div align="center">
  <h2>
    nodejs-argo 隧道代理 · Unikraft Cloud 部署版
  </h2>
  基于 <a href="https://github.com/eooce/nodejs-argo">eooce/nodejs-argo</a> 的 Argo 隧道部署工具，
  专为 PaaS / 玩具平台设计，本仓库通过 GitHub Actions 一键构建并部署到
  <a href="https://unikraft.cloud">Unikraft Cloud</a>。
</div>

---

## 📌 项目说明

本项目源自开源项目 **nodejs-argo**（Argo 隧道代理），支持多种代理协议（VLESS、VMess、Trojan 等），
集成哪吒探针（v0/v1）与 Argo 隧道。原项目仓库：[https://github.com/eooce/nodejs-argo](https://github.com/eooce/nodejs-argo)。

**本仓库在 original 基础上增加了：**

- `Dockerfile` —— 打包 Node 应用（原项目 PaaS 部署用）
- `Kraftfile` —— Unikraft unikernel 构建配置（`runtime: base-compat:latest`，rootfs 取自 Dockerfile）
- `.github/workflows/deploy-unikraft.yml` —— GitHub Actions **部署到 Unikraft Cloud**
- `.github/workflows/delete-unikraft.yml` —— GitHub Actions **删除 Unikraft 实例**

---

## ☁️ 部署到 Unikraft Cloud

### 1. 前置：注册与获取 API Token（Unikraft Key）

1. 打开 [console.unikraft.cloud](https://console.unikraft.cloud) 注册 / 登录
2. 进入 **API Tokens**（或 Settings → Tokens）页面
3. **生成一个新的 API Token**（复制保存，只显示一次）
4. 记下你的**组织名（Organization）**—— 通常是你注册的用户名，形如 `sk684437`

### 2. 配置 GitHub Actions Secrets

在仓库 **Settings → Secrets and variables → Actions → New repository secret** 添加：

| Secret 名称      | 必填 | 说明                                                            |
| ---------------- | ---- | --------------------------------------------------------------- |
| `UNIKRAFT_TOKEN` | ✅   | 上一步生成的 Unikraft Cloud API Token，工作流用它登录并执行部署 |

> 没有 `UNIKRAFT_TOKEN`，部署工作流会在登录步骤失败。

### 3. 触发部署

进入 **Actions → Deploy to Unikraft Cloud → Run workflow**，填写输入项：

| 输入项          | 必填 | 默认值              | 说明                                                                |
| --------------- | ---- | ------------------- | ------------------------------------------------------------------- |
| `metro`         | ✅   | `fra`               | 部署区域，可选 `fra` / `sin` / `sfo` / `dal` / `was`                |
| `instance_name` | 否   | `nodejs`            | 实例名称（已存在则自动更新镜像/变量，不存在则新建）                 |
| `image_tag`     | 否   | `latest`            | 镜像 tag，发布为 `<org>/unikraft:<tag>`                             |
| `publish`       | 否   | `443:3000/http+tls` | 对外映射，格式 `<对外端口>:<容器端口>/<协议>`，应用监听 `PORT=3000` |
| `memory`        | 否   | `512MiB`            | 内存大小，如 `512MiB` / `1GiB`                                      |
| `vcpus`         | 否   | `1`                 | vCPU 数量                                                           |
| `env_vars`      | 否   | (空)                | 环境变量，**用分号 `;` 分隔**：`KEY1=VALUE1;KEY2=VALUE2`            |

> ⚠️ **env_vars 请用分号 `;` 分隔**（GitHub 输入框的换行不可靠，实测只有第一行生效）。


## 📋 环境变量

| 变量名       | 是否必须 | 默认值                               | 说明                                                    |
| ------------ | -------- | ------------------------------------ | ------------------------------------------------------- |
| UPLOAD_URL   | 否       | -                                    | 订阅上传地址                                            |
| PROJECT_URL  | 否       | https://www.google.com               | 项目分配的域名                                          |
| AUTO_ACCESS  | 否       | false                                | 是否开启自动访问保活                                    |
| PORT         | 否       | 3000                                 | HTTP 服务监听端口                                       |
| ARGO_PORT    | 否       | 8001                                 | Argo 隧道端口                                           |
| UUID         | 否       | 9afd1229-b893-40c1-84dd-51e7ce204913 | 节点 UUID                                               |
| NEZHA_SERVER | 否       | -                                    | 哪吒面板域名                                            |
| NEZHA_PORT   | 否       | -                                    | 哪吒端口（443/8443/2096/2087/2083/2053 时自动启用 TLS） |
| NEZHA_KEY    | 否       | -                                    | 哪吒密钥                                                |
| ARGO_DOMAIN  | 否       | -                                    | Argo 固定隧道域名，留空使用临时隧道                     |
| ARGO_AUTH    | 否       | -                                    | Argo 固定隧道密钥                                       |
| CFIP         | 否       | www.visa.com.tw                      | 节点优选域名或 IP                                       |
| CFPORT       | 否       | 443                                  | 节点端口                                                |
| NAME         | 否       | (空)                                 | 节点名称前缀                                            |
| FILE_PATH    | 否       | ./tmp                                | 运行目录                                                |
| SUB_PATH     | 否       | sub                                  | 订阅路径                                                |
| CHAT_ID      | 否       | (空)                                 | 推送节点的 chat id（需与 BOT_TOKEN 同时填写）           |
| BOT_TOKEN    | 否       | (空)                                 | 推送节点的 bot token                                    |
| SHOW_LOG     | 否       | true                                 | 是否显示日志，`no/false/disable` 屏蔽，`true/yes` 显示  |

---


**工作流做了什么：**

1. 安装官方 Unikraft CLI（`unikraft/setup-action@v1`）
2. 用 `UNIKRAFT_TOKEN` 登录：`unikraft login --token <token文件>`
3. 解析组织名 `<org>`
4. 构建并发布镜像：`unikraft build . --output <org>/unikraft:<tag>`（使用仓库内 `Kraftfile` + `Dockerfile`）
5. 部署实例：实例不存在则 `unikraft run` 创建，已存在则 `unikraft instance edit` 更新镜像与环境变量

### 4. 删除实例

进入 **Actions → Delete Unikraft Instance → Run workflow**：

| 输入项          | 必填 | 说明             |
| --------------- | ---- | ---------------- |
| `metro`         | ✅   | 实例所在区域     |
| `instance_name` | ✅   | 要删除的实例名称 |

流程：先列出当前区域实例确认，再执行 `unikraft instance delete <metro>/<instance_name>`。

> ⚠️ 删除不可恢复，请确认实例名后再运行。

## 🔗 参考

- 原项目：[https://github.com/eooce/nodejs-argo](https://github.com/eooce/nodejs-argo)
- Unikraft Cloud：[https://unikraft.cloud](https://unikraft.cloud)
- Unikraft CLI：[https://github.com/unikraft-cloud/cli](https://github.com/unikraft-cloud/cli)
- Kraftfile 规范：[https://unikraft.com/docs/kraftfile/v0.7](https://unikraft.com/docs/kraftfile/v0.7)
