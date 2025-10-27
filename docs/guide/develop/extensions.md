# 二次开发与扩展指南

本章面向需要在现有基础上扩展功能的同学，覆盖后端（hld）、CLI（hlyr）、前端（humanlayer-wui）与共享包（packages/*）的常见扩展路径与注意事项。

## 总体建议
- 自下而上梳理数据模型与 API：先扩展 `store`/`contracts`，再到 `handlers`/`CLI`/`UI`
- 保持端到端可运行：每一步都尽量加最小可用验证（小步快跑）
- 有测试就写：为核心路径补单测/集成测，减少回归

---

## 扩展 hld（Go 后端）

典型场景：新增一个需要审批的工具路径，或扩展会话的元数据/结果。

1) 扩展存储结构
- 位置：`hld/store/sqlite.go`
- 新增字段/表后，确保迁移流程完整（若有迁移逻辑/测试）

2) 扩展领域模型与类型
- 位置：`hld/session/types.go`（会话/状态/事件）、`hld/approval/*.go`（审批）
- 新增事件类型时，明确 JSON schema 并与前端/CLI 对齐

3) 扩展处理主线
- 位置：`hld/session/manager.go`
- 在合适的时机抛出“需要审批”的挂起状态（`waiting_input`），写入审批记录
- 审批通过/拒绝后的推进逻辑在 approvals handler 路径中定义

4) 暴露 API 与 SSE
- 位置：`hld/api/handlers/*.go`、`hld/api/mapper/mapper.go`
- 为新增/变更的数据提供 REST/RPC，必要时补 SSE 事件推送

5) 测试
- 参考：`hld/*/*_test.go`、`hld/*/*_integration_test.go`
- 覆盖：事件持久化、状态转换、审批通过/拒绝分支、SSE 推送

---

## 扩展 hlyr（TS CLI + MCP）

典型场景：新增一个 CLI 命令，驱动后端新增能力。

1) 新建命令
- 位置：`hlyr/src/commands/`
- 参考现有命令（如 `launch.ts`），对接 `daemonClient`

2) 配置与类型
- 位置：`hlyr/src/config.ts`、`packages/contracts`
- 确保与后端 DTO/类型一致，最好由共享包导出类型

3) 测试
- 位置：`hlyr/tests/*`
- 覆盖参数解析、RPC 调用、错误处理

---

## 扩展 humanlayer-wui（React Web UI）

典型场景：新增设置项/页面，或在会话详情中展示新的事件类型。

1) 路由与页面
- 位置：`humanlayer-wui/src/router.tsx`、`src/pages/*`
- 新增页面时更新路由表与导航

2) 状态与数据源
- 位置：`humanlayer-wui/src/AppStore.ts`、`src/hooks/*`
- 从 REST/SSE 取数，确保数据类型与 `packages/contracts` 对齐

3) 组件
- 位置：`humanlayer-wui/src/components/*`
- 复用已有 UI 模式（卡片、表格、热键动作）

4) 测试
- 位置：`humanlayer-wui/src/*.test.ts(x)`
- 覆盖渲染、交互、SSE 更新响应

---

## 扩展共享包（packages/*）
- `packages/contracts`：共享类型/DTO，后端与前端/CLI 一致性来源
- `packages/database`：若前端示例或工具需要共享数据库逻辑

变更流程：先改 contracts → 后端/CLI/前端同步升级，避免类型漂移。

---

## 提交与代码风格
- 格式化：Biome/Prettier（根脚本 `bun run format`/`bun run lint`）
- TS 严格模式、Go 统一日志（slog）、测试先行
- 提交信息简洁明了，PR 描述包含：动机、设计、影响面、测试情况
