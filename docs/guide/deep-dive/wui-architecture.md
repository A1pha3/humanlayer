# 深入理解 humanlayer-wui：状态管理与组件设计

本章解构 WUI 的架构、状态流、关键页面与组件模式，便于二次开发与性能调优。

## 1. 入口与路由
- `src/main.tsx`：应用入口与 Provider 注入
- `src/router.tsx`：路由定义（Session 列表/详情、设置、Demo 等）

## 2. 全局状态与数据流
- `src/AppStore.ts`：核心 Store（会话/审批/设置等）
  - 负责从后端加载、订阅、合并更新
  - 对 SSE 的响应更新 UI 状态
- Hooks：`src/hooks/*.ts`
  - `useDaemonConnection`：连接管理
  - `useApprovals`：审批数据选择与操作封装
  - 其他通用 hooks（防抖、热键管理等）

## 3. 页面与组件
- 页面：`src/pages/*.tsx`
  - `SessionTablePage` / `SessionDetailPage` / `DraftSessionPage` / `WuiDemo`
- 组件：`src/components/*`
  - 结构化 UI（Cards、StackedCards、CommandPalette 等）
  - 交互：热键、FuzzySearch、对话与提示

## 4. 与后端通信
- 通过 REST 获取数据、SSE 订阅实时事件
- DTO 对齐 `packages/contracts`，避免漂移
- 错误与加载态：统一的错误边界与提示组件（`ErrorBoundary.tsx`）

## 5. 性能与可用性
- 列表/详情的最小渲染单元，减少重绘
- SSE 批量合并更新，避免抖动
- 热键作用域（`HotkeyScopeBoundary.tsx`）保证快捷键不冲突

## 6. 新增功能的路径
1) 定义 DTO/类型（如需）→ `packages/contracts`
2) 后端补 API/SSE → `hld/api/handlers`
3) Store 增字段/方法 → `AppStore.ts`
4) 页面/组件消费新状态，补测试用例

## 7. 测试
- 参考：`*.test.tsx`（组件与页面层）
- 使用 testing-library 驱动用户视角用例

## 8. 常见问题
- 看不到实时更新 → 检查 SSE 连接、CORS、代理
- 状态不同步 → 查看合并逻辑与键值映射
- 渲染卡顿 → 分析渲染树，拆分组件与使用 memo
