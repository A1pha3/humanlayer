# 深入理解 hld：Session/Approval 主线

本章从源码角度解读 hld 会话与审批主线设计，帮助你快速定位问题与扩展能力。

## 1. 状态机与关键类型
- 定义位置：`hld/session/types.go`
- 常见状态（示例）：
  - `running`：执行中
  - `waiting_input`：等待人工输入/审批（高风险工具触发）
  - `completed`：完成
  - `failed`：失败（含下游错误或拒绝审批）
  - `interrupting` / `interrupted`：中断流程（外部打断/重试）
- `SessionManager` 接口：抽象会话管理的对外能力（启动、继续、打断、查询）

## 2. 会话管理主线（manager.go）
文件：`hld/session/manager.go`
- 责任：
  - 启动/恢复会话，协调与下游（如 Claude Code）的交互
  - 处理流式事件（`processStreamEvent`），持久化与状态推进
  - 遇到需要审批的工具调用时，挂起为 `waiting_input` 并等待外部指令

时序（ASCII）：
```
[Start] -> create Session -> stream events -> [needs approval?] --yes--> create Approval -> set Status: waiting_input
                                                         |                                           |
                                                         no                                          v
                                                         |                                  wait human decision
                                                         v                                           |
                                        persist result & update status <----- approve/reject -------+
                                                         |
                                                      [End]
```

- 关键点：
  - 事件序列化存储，便于 Debug 与审计
  - 最终状态判定（`finalStatus` 推断）
  - 与存储、SSE 通知的耦合点在 API 层进行解耦

## 3. 审批与权限监控
- 审批逻辑：`hld/approval/*.go`
  - 创建、查询、批准与拒绝路径
  - 与会话状态联动（批准后继续、拒绝则失败/分支）
- 危险跳过权限（Dangerous Skip Permissions）：`hld/session/permission_monitor.go`
  - 定时扫描与过期处理，防止长期豁免导致风险
  - 关键日志："starting dangerous skip permissions expiry monitor"

## 4. API/Handlers 与 SSE
- 目录：`hld/api/handlers/*.go`
  - sessions：会话的创建、查询、继续/中断等
  - approvals：审批的列出、批准、拒绝
  - sse：服务端事件推送（前端订阅实时更新）
- 映射层：`hld/api/mapper/mapper.go`（Model ↔ API DTO）

## 5. 存储层（SQLite）
- 文件：`hld/store/sqlite.go`
  - 会话、事件、审批、MCP 配置等数据结构落盘
  - 事务边界与错误处理（参考各 `*_test.go`）
- 读写模式：先写后广播，保证可追溯性

## 6. 测试策略
- 强调端到端与集成测试：
  - `hld/daemon/*_integration_test.go`
  - `hld/session/*_test.go`
  - `hld/api/handlers/*_test.go`
  - `hld/store/*_test.go`
- 建议调试顺序：
  1) 复现失败的集成用例
  2) 打开 `manager.go` 与对应 handler，逐步比对日志与状态变更
  3) 查看 SQLite 中的状态/事件记录是否一致

## 7. 扩展建议
- 新增需要审批的工具路径：
  - 在业务处理处抛出“需要审批”的事件/标记
  - 写入审批记录并切换会话状态为 `waiting_input`
  - 在 approvals handler 中定义批准/拒绝后的分支行为
- 扩展事件模型：
  - 确认事件 JSON 的 schema 与 UI/CLI 的解码器对齐
  - 为新事件添加映射与前端展示

## 8. 性能与可靠性
- 流式事件处理与批次落盘的平衡
- 日志等级与字段统一，便于生产排障
- 对长时间挂起（等待审批）的会话进行健康监控与清理策略

## 9. 快速跳转
- `hld/session/manager.go`
- `hld/session/types.go`
- `hld/approval/*.go`
- `hld/api/handlers/*.go`
- `hld/store/sqlite.go`
