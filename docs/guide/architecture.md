# 架构与目录总览

本仓库是一个使用 Turbo + Bun 管理的 Monorepo，工作空间位于 `apps/*` 与 `packages/*`，同时包含 Go 与 TypeScript 子项目。

## 顶层信息
- 包管理：Bun（`packageManager: bun@1.2.23`）
- 根脚本：通过 `turbo` 统一执行 build/dev/lint 等
- 代码风格：Biome/Prettier，TS 严格模式

## 目录结构（要点）
- `hld/`：Go 后端守护与服务层（Session/Approval/Store/RPC/REST/SSE）
- `claudecode-go/`：Go SDK，用于程序化启动 Claude Code 会话等
- `hlyr/`：TS CLI + MCP 服务，与 hld 交互（JSON-RPC/MCP）
- `humanlayer-wui/`：React Web UI（Vite），会话与审批的可视化管理界面
- `apps/react/`：Bun + React + Tailwind 示例，展示 HTML 直引 TSX、协作编辑（Yjs/ElectricSQL）
- `packages/*`：共享包（例如 `packages/database`、`packages/contracts`）

## 高层数据/控制流
- 开发者通过 CLI（hlyr）或 WUI（humanlayer-wui）发起请求
- 请求到达本地 hld（Go），由其：
  - 管理会话生命周期与状态机
  - 触发审批流与权限校验（危险操作需确认）
  - 通过存储（SQLite）持久化事件/结果
  - 对外暴露 RPC/REST/SSE 以供前端/CLI 订阅与交互
- 前端（WUI）通过 HTTP/SSE 展示实时状态；CLI 通过 JSON-RPC/MCP 驱动任务执行

## 关键约定
- Go 侧：丰富的单元/集成测试、mockgen、SQLite 存储、事件驱动
- TS 侧：严格 TS、bundler moduleResolution、按子项目独立脚本
- 前端：Readable 组件拆分、Store 状态流清晰、热键与可用性设计

## 下一步阅读
- [hld 使用指南](./projects/hld.md)
- [hlyr 使用指南](./projects/hlyr.md)
- [WUI 使用指南](./projects/humanlayer-wui.md)
