# 深入理解 hlyr：CLI 与 MCP 设计

本章聚焦 hlyr 的命令实现、与 hld 的连接方式（JSON-RPC/MCP）、配置体系与测试策略。

## 1. 模块结构与入口
- 目录：`hlyr/src/*`
  - `index.ts`：程序入口与命令注册
  - `commands/*.ts`：各命令实现（如 `launch.ts`、`claude.ts`、`thoughts.ts`）
  - `daemonClient.ts`：与 hld 的通信客户端（RPC/HTTP 等）
  - `mcp.ts`：MCP 服务的适配与注册
  - `config.ts`：配置解析与默认值
  - `thoughtsConfig.ts`、`mcpLogger.ts`：扩展能力

建议先从 `index.ts` 看命令注册，再跳到对应的 `commands/*.ts`。

## 2. 与 hld 的通信：daemonClient
- 文件：`hlyr/src/daemonClient.ts`
- 职责：
  - 维护与 hld 的 JSON-RPC/HTTP 客户端
  - 封装对会话、审批、设置等 API 的请求
- 调试点：
  - 连接地址来源：`config.ts`
  - 失败重试与错误信息输出

## 3. MCP 集成
- 文件：`hlyr/src/mcp.ts`
- 作用：
  - 作为 MCP server 暴露工具给模型使用
  - 负责工具列表、上下文注入与调用日志（`mcpLogger.ts`）
- 关键点：
  - 工具定义需与 hld 的审批与权限模型对齐
  - 对需要审批的路径，保障人类在环

## 4. 配置体系
- 文件：`hlyr/src/config.ts`
- 来源优先级：命令行参数 > 环境变量 > 默认值
- 与 UI/后端对齐：建议通过 `packages/contracts` 共享 DTO/类型

## 5. 新增命令的步骤
1) 在 `src/commands/<name>.ts` 新建命令文件
2) 在 `index.ts` 注册命令（或自动扫描，视实现）
3) 在命令中从 `daemonClient` 调用后端能力
4) 为命令参数和响应使用 `packages/contracts` 中的类型
5) 增加测试（参数解析、调用、失败场景）

## 6. 测试与调试
- 测试：`hlyr/tests/*.test.ts`
  - e2e 示例：`hlyr/tests/claudeInit.e2e.test.ts`、`configShow.e2e.test.ts`
- 调试：
  - 打印 RPC 请求/响应（注意脱敏）
  - 遇到超时/连接失败时比对 hld 日志

## 7. 常见问题
- 与 hld 版本不匹配 → 更新 `packages/contracts` 并统一升级
- 命令解析冲突 → 调整别名与参数名
- 输出过长/不可读 → 提供 `--json` 或 `--verbose` 开关
