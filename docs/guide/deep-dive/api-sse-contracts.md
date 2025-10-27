# 深入理解 API、SSE 与 Contracts

本章从接口与类型的角度，贯通 hld（后端）、hlyr（CLI）与 humanlayer-wui（前端）。

## 1. Contracts（共享类型）
- 位置：`packages/contracts`
- 作用：共享 DTO/类型，确保多端一致性
- 建议：任何接口变更先升级 contracts，再同步到后端与前端/CLI

## 2. REST / JSON-RPC
- hld 暴露的 HTTP/REST 与 JSON-RPC 接口由 `hld/api/handlers/*.go` 提供
- 会话：创建、查询、继续、打断等
- 审批：列出、批准、拒绝
- 配置/文件：按需扩展

## 3. SSE（服务端事件）
- 位置：`hld/api/handlers/sse.go`
- 作用：推送会话、审批、系统事件的实时更新
- 客户端：WUI 通过 EventSource 订阅；CLI 亦可消费

时序（ASCII）：
```
hld emit event -> store persist -> notify SSE -> WUI/CLI receive -> update UI/state
```

## 4. Mapper 层
- 位置：`hld/api/mapper/mapper.go`
- 作用：领域模型 ↔ API DTO 的双向映射
- 注意：新增字段需更新 mapper，避免遗漏

## 5. 版本与兼容性
- 为 REST 与 SSE 事件增加 `version`/`type` 字段有助于演进
- 向后兼容策略：新字段默认可选、客户端宽松解析

## 6. 开发建议
- 以 contracts 为单一事实源（SSOT）
- 端到端联调时校验：
  - 后端响应 JSON ↔ 前端/CLI 类型定义
  - SSE 事件载荷是否包含必需字段
- 回归测试：
  - handlers 层单测
  - SSE 订阅的端到端测试（WUI/CLI）
