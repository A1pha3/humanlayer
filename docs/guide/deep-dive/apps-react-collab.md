# 深入理解 apps/react：协作编辑与 Bun 构建

本章剖析示例应用的协作编辑链路（Yjs/ElectricSQL/TipTap）与 Bun 驱动的前端构建模式。

## 1. 运行方式（回顾）
```bash
cd apps/react
bun install
bun dev # 默认 PORT=4000
```

## 2. 入口与 Bun 构建
- `src/index.html`：HTML 直引 TSX（Bun 自动打包）
- `src/frontend.tsx`：React 入口（`createRoot` 渲染 `App`）
- `build.ts`：自定义 Bun 构建脚本（tailwind 插件、entrypoints 扫描、minify/target 配置）
- `bunfig.toml`：静态资源与插件配置

## 3. 协作编辑栈
- TipTap（`lib/tiptap.ts`、`components/Editor.tsx`）
  - `@tiptap/extension-collaboration` & `...-cursor`
  - Extensions 组合与 Editor 实例初始化
- Yjs（`y-electric/index.ts`）
  - `Y.Doc` 与 awareness 同步
  - `syncProtocol` 与 `awarenessProtocol` 的消息处理
  - `sendOperation`/`sendAwareness` 将本地更新编码并上行
- ElectricSQL 代理（`lib/electric-proxy.ts`）
  - 将请求参数透传到 Electric 后端，合并鉴权与表信息

## 4. 时序与消息流
1) 本地编辑触发 Yjs 更新（update）
2) 通过 `syncProtocol.writeUpdate` 编码后经 `sendOperation` 上行
3) 服务端（Electric）广播回其他客户端
4) 其他客户端 `readSyncMessage` 应用远端更新
5) awareness（光标/在线状态）同理经 `encodeAwarenessUpdate`

## 5. 关键调试点
- 本地判断 `origin !== this` 避免自发自收造成回环
- `lastSyncedStateVector` 与 `pendingUpdates` 协调首次/断线重连
- `index.css`/`editor.css` 的样式兼容（光标、代码块、blockquote）

## 6. 扩展示例
- 新增协作房间命名与权限校验（`roomName`/`table`）
- 在 Electric 侧做租户隔离（token/source_id 注入）
- 富文本扩展：自定义节点/marks、Slash Menu、快捷键

## 7. 性能与可靠性
- 批量发送/合并更新，减少网络压力
- 断线重连的状态恢复与重复消息去重
- 游标更新节流，避免大房间内频繁广播
