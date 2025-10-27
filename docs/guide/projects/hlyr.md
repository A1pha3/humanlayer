# hlyr 使用指南（TypeScript CLI + MCP）

hlyr 是与 hld 交互的命令行与 MCP 组件，适合脚本化/自动化场景：
- 通过 JSON-RPC/MCP 与 hld 通信
- 提供 `launch/claude/thoughts` 等命令

## 安装与运行
```bash
cd hlyr
bun install
bun run dev # 或查看 package.json/scripts 与 README
```

## 常见命令（示例）
- `hlyr launch`：发起一次会话
- `hlyr claude`：与 Claude Code 会话交互
- `hlyr thoughts`：思考/上下文管理相关命令

## 配置
- 默认连接到本地 hld。可通过环境变量/参数指定地址、密钥等
- 相关源码：`src/config.ts`、`src/daemonClient.ts`、`src/mcp.ts`

## 开发要点
- 使用严格 TS，`bundler` 模块解析
- 命令分布于 `src/commands/*`
- 与 hld 的数据类型对齐（参考 `packages/contracts`）

## 下一步
- 与 [hld 使用指南](./hld.md) 配合运行；或在 [WUI 使用指南](./humanlayer-wui.md) 中图形化查看会话与审批状态。
