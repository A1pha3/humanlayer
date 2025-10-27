# 部署与演示

本章给出本地/演示环境的推荐组合与注意事项。

## 本地演示最小组合
- 启动 hld（Go 后端）
- 启动 humanlayer-wui（前端可视化）或 hlyr（CLI）
- 可选：启动 apps/react 作为 Bun + 协作编辑示例

## 步骤
1) hld
```bash
cd hld
go run ./...
```

2A) WUI
```bash
cd humanlayer-wui
bun install
bun run dev
```
配置连接到本地 hld。

2B) CLI
```bash
cd hlyr
bun install
bun run dev
```

3) 可选示例（apps/react）
```bash
cd apps/react
bun install
bun dev
```

## 数据初始化与注意
- 若需要预置数据，可编写小脚本调用 hld API 创建会话/审批
- 端口冲突、CORS 与代理设置需提前确认

## 进一步
- 将前端打包后由静态服务托管；hld 作为独立服务运行
- 按需增加 Nginx/反向代理，保证 SSE 长连接不中断
