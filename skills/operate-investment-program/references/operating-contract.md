# V5 投资经营契约

## InvestmentProgram

内容必须且只能包含：`objective`、`success_criteria`、`benchmark`、`risk_budget`、`universe`、`horizons`、`operating_cadence`、`stop_conditions`、`account_ids`。

Context 引用必须且只能包含当前已确认的 `investor_revision_id`、`mandate_revision_id`、`attention_revision_id`。同一时刻只允许一份 active Program。Trial 必须有未来 `expires_at`；确认必须保留用户批准引用。Context 或账户状态变化后，经营动作必须 fail closed，直到用户确认 Program 修订版。

## Opportunity

```text
observed → researching → qualified → actionable
       └──────── 任一 active 阶段可 rejected / expired / closed
```

证据阶段只能逐级前进。qualification 必须包含：

- `evidence_state`: incomplete / corroborated / decision_grade；
- `independent_source_count`；
- `data_freshness`: current / partial / stale / unknown；
- `falsifiers`、`major_unknowns`、`counterevidence`；
- `decision_basis`。

qualified 至少为 corroborated、两个独立来源和一个 falsifier。actionable 必须 decision_grade/current、重大未知项为空，并绑定 current issued Decision。阶段不是 LLM 置信百分比。

## DecisionQueue

只接受 active actionable Opportunity。Queue 不得比 Decision/ManualAction 活得更久。状态是 ready、presented、snoozed、accepted、rejected、expired、closed。

accepted 只记录用户选择；它不创建 Execution、不写 Ledger、不连接券商，并在有效期内继续作为“等待手工执行或反馈”的行动显示。snoozed 必须提供未来且早于 Queue 失效时间的 `snoozed_until`，到时恢复 ready。真实成交必须由 Lifecycle 记录为待确认并经用户确认。

## Brief

所有 Brief 共有：`summary`、`what_changed`、`decision`、`risks`、`next_check_at`、`queue_item_ids`。

Weekly 另有 `program_progress`、`research_pipeline`。Monthly 另有 `scorecard_id`、`lessons`、`proposed_changes`。

`next_check_at` 必须晚于 Brief 的 `as_of`。`no_action` 不得引用 Queue，且系统中不能存在 ready/presented/snoozed/accepted Queue；它只在当前 Program Revision 且尚未到 `next_check_at` 时作为今日结论。`action` 至少引用一个有效 Queue。

## Scorecard

每个 metric 只能包含 `name`、`calculation_id`、`output_path`；路径必须从 `outputs.` 开始并解析为标量。Comparison 只能用 `label`、`left_metric`、`right_metric`、`interpretation`，左右项必须引用已解析 metric 名。

先用 `v5_program_metrics_calculate` 冻结本期过程流量。它标为 insufficient_evidence 的收益、基准、用户时间或成本不得由模型补写，也不得用未归因的系统全局计数替代。

## 今日状态

- setup_required：缺 active Program；
- action：存在有效 Queue；
- no_action：最新日 Brief 明确无行动；
- review_required：没有行动，但本期尚未形成有证据的结论。

## 失败关闭

Context 变化、Decision/ManualAction 过期、Calculation hash 失败、证据引用不存在、Queue 未实际送达、Feature/Gate 未开放时，不得用自然语言绕过工具错误。向用户解释具体缺口和安全下一步。
