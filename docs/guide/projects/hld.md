# hld 使用指南（Go 后端守护）

hld 是本项目的核心后端与编排层，负责：
- 会话（Session）生命周期管理与状态机
- 危险操作审批（Approvals）与权限监控
- RPC/REST/SSE 接口，对接 CLI 与 Web UI
- SQLite 持久化与事件溯源

## 启动
```bash
cd hld
go run ./...
```

启动后将开放本地 HTTP/RPC 服务（端口以源码配置为准）。

## 重要目录与文件（示例）
- `session/manager.go`：会话管理核心逻辑（创建、事件处理、状态变迁）
- `api/handlers/*.go`：REST/RPC/SSE 端点与路由处理
- `store/sqlite.go`：SQLite 存储实现与迁移
- `approval/*.go`：审批与权限相关逻辑

可通过测试文件快速理解用法：
- `daemon/*_integration_test.go`、`api/handlers/*_test.go`、`session/*_test.go`、`store/*_test.go`

## 常见操作
- 查看会话：通过 REST 或 CLI（hlyr）查询
- 处理审批：在 WUI 中执行允许/拒绝，或使用 CLI/API
- 订阅事件：通过 SSE 获取实时会话/审批更新

## 调试建议
- 增加日志级别（参考源码中的 slog 配置）
- 结合 `*_integration_test.go` 运行端到端测试来复现问题
- 定位状态：留意 `Status*` 常量与状态转换点

## 下一步
- 配合 [hlyr 使用指南](./hlyr.md) 与 [WUI 使用指南](./humanlayer-wui.md) 完成端到端闭环。
