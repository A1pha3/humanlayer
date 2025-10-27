# humanlayer-wui 使用指南（React Web UI）

humanlayer-wui 提供可视化的会话与审批管理界面：
- 会话列表/详情
- 审批流管理（允许/拒绝）
- 设置与连接配置

## 开发与运行
```bash
cd humanlayer-wui
bun install
bun run dev
```

Vite 启动后在浏览器打开本地地址（端口见终端输出）。首次使用可到设置页面配置后端（hld）连接。

## 代码结构要点
- 入口与路由：`src/main.tsx`、`src/router.tsx`
- 全局 Store 与页面：`src/AppStore.ts`、`src/pages/*`
- 组件：`src/components/*`（含热键、调试面板等）

## 与后端的交互
- 通过 REST/SSE 订阅与渲染实时状态
- 与 hld 的数据结构保持一致（参考 `packages/contracts`）

## 调试建议
- 使用浏览器 DevTools 观察 SSE 流与网络请求
- 快速定位页面状态问题：从 AppStore 状态切入

## 下一步
- 结合 [hld 使用指南](./hld.md) 与 [hlyr 使用指南](./hlyr.md) 打通端到端闭环。
