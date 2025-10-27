# 故障排查与 FAQ

汇总安装/运行/联调中的常见问题与排查路径。

## 安装与构建
- Bun 版本不匹配
  - 参考根 `package.json` 的 `packageManager: bun@1.2.23`
  - 解决：升级/切换版本；删除 `bun.lock` 后重装仅在确认必要时进行
- 依赖安装失败
  - 检查网络与私有源配置；必要时国内镜像
  - 清理缓存后重试：`bun install --force`

## 运行与端口
- 端口占用
  - 修改 `PORT` 或相关配置（WUI/Vite、apps/react、hld）
- 环境变量未生效
  - Bun 会自动加载 `.env`；前端暴露需 `VITE_`/`BUN_PUBLIC_` 前缀

## 前后端联调
- WUI 看不到数据
  - 确认已连接正确的 hld 地址
  - 浏览器网络面板检查接口与 SSE 是否 2xx、是否被 CORS/代理拦截
- CLI 与 hld 通信失败
  - 查看 `hlyr/src/daemonClient.ts` 配置与实际地址是否一致
  - hld 是否已启动、日志是否有连接记录

## 审批相关
- 任务应当触发审批但没有
  - 检查业务分支是否触发了审批事件（`manager.go` 与 `approval/*.go`）
- 批准/拒绝无效
  - 查看 approvals handler 的分支实现，确认会话状态是否推进

## SSE 与实时性
- 无事件推送
  - `api/handlers/sse.go` 是否有广播日志；浏览器是否断流
  - 反向代理/网关是否启用超时或断开策略

## 数据一致性
- UI 显示与存储不一致
  - 先看 store（SQLite）记录是否正确，再看 mapper 与 DTO 是否同步

## 其他
- 测试频繁波动
  - 避免时间/随机性未隔离；为异步流程增加等待/重试策略
- 性能问题
  - 关注批量写入/事件频率、SSE 连接数，优化日志等级与采样
