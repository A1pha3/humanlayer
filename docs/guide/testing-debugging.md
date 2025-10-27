# 测试与调试

本章覆盖 Go 与 TypeScript 两侧的测试策略、调试技巧与常见工具。

## Go 侧（hld 等）

### 测试分类
- 单元测试：针对 store、mapper、handlers 的局部逻辑
- 集成/端到端：`daemon/*_integration_test.go`、`session/*_test.go`、`api/handlers/*_test.go`、`store/*_test.go`

### 运行测试
```bash
cd hld
go test ./... -v
```

### 调试建议
- 打开详细日志（slog），在关键节点打印状态与事件 JSON
- 遇到审批卡住：检查是否进入 `waiting_input`，是否写入审批记录
- 对比 SQLite 落盘数据与 SSE 推送是否一致

## TypeScript 侧（hlyr / humanlayer-wui / apps）

### 测试工具
- hlyr：vitest（见 `hlyr/vitest.config.ts`）或项目现有测试栈
- humanlayer-wui：@testing-library/react + vitest（参考现有 `*.test.tsx`）
- apps/react：可选轻量验证（Bun 自带 `bun test` 也可）

### 运行示例
```bash
# CLI
cd hlyr
bun test

# WUI（若配置了 vitest）
cd humanlayer-wui
bun test
```

### 调试建议
- 浏览器 DevTools 观测网络与 SSE 事件流
- TS 严格类型下先修复类型告警再看运行时错误
- 前后端 DTO 对齐：以 `packages/contracts` 为准

## 端到端调试流程建议
1) 复现路径最短化：最小输入、最少步骤
2) 双端日志对齐：hld 日志 vs. 前端网络请求/SSE
3) 数据一致性检查：store 写入与 UI 展示是否一致
4) 回归测试补齐：为 bug 修复追加测试用例

## 常用命令速查
- 根目录：`bun run dev | build | lint | check-types`
- Go：`go test ./...`、`go run ./...`
- Bun：`bun dev`、`bun start`、`bun run build.ts`（apps/react）
