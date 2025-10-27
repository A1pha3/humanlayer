# 术语表

- Session（会话）
  - 一次任务/交互的执行容器，具有状态机（running/waiting_input/completed 等）
- Approval（审批）
  - 对敏感/高风险操作进行人工确认的流程
- Dangerous Skip Permissions（危险跳过权限）
  - 在短时间内跳过审批的豁免机制，受定时监控与过期清理
- MCP（Model Context Protocol）
  - 一种与模型交互、提供工具/上下文的协议，hlyr 侧提供实现
- JSON-RPC / REST / SSE
  - 与后端通信与事件订阅的主要协议/风格
- Store / SQLite
  - 后端持久化层，记录会话、事件、审批等
- DTO / Contracts
  - 前后端/CLI 共享的数据结构定义，位于 `packages/contracts`
- WUI（Web UI）
  - React 前端界面（humanlayer-wui），用于可视化管理会话与审批
- TipTap / Yjs / ElectricSQL（示例）
  - 富文本与协作编辑相关库，见 `apps/react`
