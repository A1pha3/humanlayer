# 快速开始

目标：10 分钟内跑通一个最短闭环（hld 后端 + hlyr CLI 或 WUI 前端）。

## 0. 环境准备
- Node.js ≥ 20（本仓库要求）
- Bun（packageManager）：建议 v1.2.23
- Go（用于 hld）
- Git、bash/zsh

## 1. 安装依赖
在仓库根目录执行：

```bash
bun install
```

常用脚本（根目录 `package.json`）：
- `bun run dev` → `turbo run dev`
- `bun run build` → `turbo run build`
- `bun run lint` → `turbo run lint`

## 2. 启动 hld（Go 后端守护）
在 `hld/` 下：

```bash
# 常见做法（若提供 make 或脚本可优先使用）
# 直接 go run 示例：
cd hld
go run ./...
```

成功后应看到 HTTP/RPC 服务启动日志（默认本地端口以源码配置为准）。

## 3A. 路线 A：启动 hlyr（TS CLI + MCP）
在 `hlyr/` 目录：

```bash
cd hlyr
bun install
bun run dev # 或按 README 中的命令运行 CLI
```

典型命令：
- `hlyr launch` 启动一次会话
- `hlyr claude`/`hlyr thoughts` 等命令视具体实现

CLI 会通过 JSON-RPC/MCP 与本地 hld 通信。

## 3B. 路线 B：启动 Web UI（humanlayer-wui）
在 `humanlayer-wui/` 目录：

```bash
cd humanlayer-wui
bun install
bun run dev
```

打开浏览器访问本地地址（Vite 默认端口，具体见项目输出）。首次进入可在“设置/连接”中配置本地 hld 地址，查看会话与审批流。

## 4. 可选：启动示例应用（apps/react）
演示 Bun + React + Tailwind 与协作编辑（Yjs/ElectricSQL）：

```bash
cd apps/react
bun install
bun dev # 默认 PORT=4000
```

访问 http://localhost:4000，查看示例页面与编辑器。

## 5. 常见问题
- 端口占用：修改相应 `PORT` 或配置文件。
- Bun 版本：若构建异常，优先检查本地 Bun 版本并与仓库锁定版本靠近。
- 环境变量：本仓库多使用 Bun 自动加载 .env。前端项目可通过 `VITE_`/`BUN_PUBLIC_` 前缀暴露运行时变量。
