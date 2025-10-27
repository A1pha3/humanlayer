# 端到端审批工作流实操

目标：从发起会话→触发敏感操作→等待人工审批→批准/拒绝→会话继续/终止，全流程跑通，并学会排查常见问题。

## 前置条件
- 已启动 hld（见“hld 使用指南”）
- 二选一：
  - CLI 路径：已在 `hlyr/` 安装并可以运行基本命令
  - WUI 路径：已在 `humanlayer-wui/` 启动并配置连到本地 hld

## 概念速览
- 会话（Session）：任务的执行容器，具备状态机（running/waiting_input/completed 等）
- 审批（Approval）：对“高风险/敏感”操作进行人工确认，批准后继续、拒绝则终止或分支处理
- 事件（Events/SSE）：hld 通过 SSE 推送会话与审批的实时更新

相关源码（建议对照阅读）
- 状态与会话：`hld/session/types.go`、`hld/session/manager.go`
- 审批：`hld/approval/*.go`
- API/SSE：`hld/api/handlers/*.go`（sessions、approvals、sse）
- 存储：`hld/store/sqlite.go`

---

## 路径 A：使用 CLI（hlyr）

时序（ASCII）：
```
CLI launch -> hld create session -> task runs -> [approval?]
                                          | yes
                                          v
                                     create approval -> wait human -> approve/reject -> resume/abort
```

1) 启动 hld（已完成，不再赘述）

2) 启动/使用 hlyr
```bash
$ cd hlyr
$ bun install
# 输出示例（截断）：
#  Saved lockfile
#  + dependencies installed
#   143 packages installed in 2.14s
$ bun run dev
# 输出示例（截断）：
#  hlyr v0.x started
#  RPC target: http://localhost:8080
#  Ready for commands
```

3) 发起会话（示例）
- 运行 `hlyr launch`（或仓库中提供的等价命令）发起一次任务。
  - 输出示例（截断）：
  ```bash
  $ hlyr launch --query "Index project and start refactor"
  # session created: s_abc123
  # status: running
  # streaming logs...
  ```
- 该任务在执行过程中，一旦命中“需要审批”的操作，hld 会将会话状态置为 `waiting_input` 并创建待审批项。
  - 输出示例（WUI）：“待审批”卡片出现，CLI：
  ```bash
  # status: waiting_input
  # approval required: write_file to /critical/path
  # open WUI or run: hlyr approvals approve --id ap_456
  ```

4) 观察待审批
- 方式一：在 WUI 中打开“审批”/“会话详情”查看待处理项；
- 方式二：通过 CLI/HTTP 轮询或订阅 SSE（参见 `api/handlers/sse.go`）。

5) 执行审批
- 在 WUI 中点“允许”或“拒绝”；或使用 CLI/API 进行批准/拒绝操作。

6) 验证会话恢复
- 批准后，会话恢复执行并推进到下一阶段；
- 拒绝后，会话进入失败/中断或给出替代分支（视工具实现）。

7) 查看结果
- 在 WUI 的会话详情或 CLI 输出中查看最终结果（store/sqlite 中亦有持久化记录）。

---

## 路径 B：使用 WUI（humanlayer-wui）

1) 启动 WUI
```bash
cd humanlayer-wui
bun install
bun run dev
```

2) 连接后端
- 在“设置/连接”中填写本地 hld 地址。

3) 发起会话
- 通过 UI 的“启动/新会话”入口发起；或先使用 CLI 发起，再在 UI 查看。

4) 处理审批
- 会话触发敏感操作后，UI 会显示“待审批”卡片；
- 点击“允许/拒绝”，实时影响会话推进。

5) 观察状态流与日志
- 会话详情中可查看状态变化与事件时间线；
- 开发时建议同时关注浏览器网络面板与 hld 日志输出。

---

## 常见排错清单
- 看不到待审批
  - 确认任务确实触发了需要审批的工具路径；
  - 检查 hld 日志是否出现 `waiting_input`；
  - 确认 WUI 已连到正确的 hld 地址、SSE 正常（无 4xx/5xx）。
- 批准后无反应
  - 检查 `hld/session/manager.go` 事件处理日志；
  - 查看存储层是否成功写入结果（`store/sqlite.go`）。
- SSE 无推送
  - 浏览器网络/SSE 连接是否被代理或跨域阻断；
  - 服务端 `api/handlers/sse.go` 是否有连接与广播日志。
- 权限/豁免过期
  - 关注 `hld/session/permission_monitor.go` 的定时清理与日志；
  - 如果使用了“危险跳过权限”，确认过期策略与时间窗口。

---

## 进一步练习
- 将一次审批串联多个子步骤（多次等待输入）
- 在 CLI 增加一个命令，调用一个需要审批的工具分支
- 为审批流增加上下文说明与审计记录，并在 WUI 展示
