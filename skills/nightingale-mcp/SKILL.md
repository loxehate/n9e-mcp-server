---
name: nightingale-mcp
description: 使用 Nightingale MCP Server 的源码能力来执行告警、监控目标、通知、屏蔽、事件流水线与用户团队查询/管理。适用于用户要求“查告警/查目标/创建屏蔽/看流水线/按业务组排查问题”等 Nightingale 运维场景。
---

# nightingale-mcp

基于 `n9e-mcp-server` 源码整理的执行型 skill。用于让代理在 Nightingale 场景下稳定选择正确工具、参数与排障顺序。

## 何时使用

当用户需求涉及以下任一方向时触发：

- 告警排查（当前告警、历史告警、规则、订阅）
- 监控对象排查（targets）
- 值班/通知链路（通知规则、事件流水线执行）
- 临时静默（mutes 创建/更新）
- 人员与组织（用户、用户组、业务组）
- 数据源盘点（datasource）

## 运行前检查

1. 确认 MCP 服务端有 `N9E_TOKEN`。
2. 若用户要求“只读”，确认启用了 `N9E_READ_ONLY=true`（写工具会被自动禁用）。
3. 若用户要求最小暴露工具，设置 `N9E_TOOLSETS` 仅启用必要工具集。

> 默认工具集：`alerts,targets,datasource,mutes,busi_groups,notify_rules,alert_subscribes,event_pipelines,users`。

## 推荐执行流程

1. **澄清范围**：业务组（gid）、时间范围（hours 或 stime/etime）、严重级别（severity）。
2. **先查后改**：
   - 先用 list/get 工具收集上下文。
   - 仅在用户明确授权或请求下执行 create/update。
3. **最小化调用**：优先单条精确查询（get），避免无边界 list。
4. **结果结构化输出**：按“现状 → 影响面 → 建议动作”汇总。

## 参数约束（高频）

- 时间范围：`hours` 与 `stime/etime` 互斥。
- 分页：`limit >= 0`，`page >= 0`。
- 告警级别：`severity` 仅允许 `1,2,3`（可逗号分隔）。
- 类别：`cate` 仅允许 `prometheus|host|elasticsearch|loki|$all`。
- 恢复状态：`is_recovered` 仅允许 `-1|0|1`。
- 规则产品线：`rule_prods` 仅允许 `host|metric|loki|anomaly`。

详细工具说明见：`references/tools.md`。

## 常用任务模板

### 模板 A：告警速查

1. `list_active_alerts`（限定时间/级别/业务组）
2. 对重点事件调用 `get_active_alert`
3. 若需历史回溯，调用 `list_history_alerts` + `get_history_alert`
4. 输出 Top 风险对象与建议

### 模板 B：维护窗口静默

1. 先 `list_mutes` 避免重复策略
2. 根据用户给定标签与时长执行 `create_mute`
3. 回读 `get_mute` 确认生效范围

### 模板 C：通知未送达排查

1. `list_notify_rules` / `get_notify_rule` 核对路由
2. `list_event_pipeline_executions` 查看执行失败记录
3. 必要时扩展到 `list_all_event_pipeline_executions`

## 输出规范

- 明确写出筛选条件（时间、业务组、级别、关键标签）。
- 若数据不足，优先提出“下一条最小必要查询”。
- 涉及写操作时，先复述变更对象和持续时间再执行。
