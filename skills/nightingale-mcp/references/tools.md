# Nightingale MCP 工具参考（源码对齐）

## 工具集与工具

### alerts
- `list_active_alerts`
- `get_active_alert`
- `list_history_alerts`
- `get_history_alert`
- `list_alert_rules`
- `get_alert_rule`

### targets
- `list_targets`

### datasource
- `list_datasources`

### mutes
- `list_mutes`
- `get_mute`
- `create_mute`（写）
- `update_mute`（写）

### notify_rules
- `list_notify_rules`
- `get_notify_rule`

### alert_subscribes
- `list_alert_subscribes`
- `list_alert_subscribes_by_gids`
- `get_alert_subscribe`

### event_pipelines
- `list_event_pipelines`
- `get_event_pipeline`
- `list_event_pipeline_executions`
- `list_all_event_pipeline_executions`
- `get_event_pipeline_execution`

### users
- `list_users`
- `get_user`
- `list_user_groups`
- `get_user_group`

### busi_groups
- `list_busi_groups`

## 读写模式约束

- 当服务端以 `read-only` 启动时，仅注册 read tools。
- `create_mute`、`update_mute` 会被自动禁用。

## 服务端关键配置

- `N9E_TOKEN`：必填。
- `N9E_BASE_URL`：默认 `http://localhost:17000`。
- `N9E_TOOLSETS`：默认全部工具集，可按需缩减。
- `N9E_READ_ONLY`：`true` 时禁用写操作。
- `N9E_MCP_LOG_LEVEL`：支持 `debug|warn|error|info(默认)`。

## 参数校验规则

- `hours` 与 `stime/etime` 互斥。
- `stime < etime`。
- `limit >= 0`，`page >= 0`。
- `severity in {1,2,3}`（支持逗号分隔）。
- `cate in {prometheus,host,elasticsearch,loki,$all}`。
- `rule_prods in {host,metric,loki,anomaly}`（支持逗号分隔）。
- `is_recovered in {-1,0,1}`。
