# 投资经营契约

## 投资计划

计划内容为 `objective`、`success_criteria`、`benchmark`、`risk_budget`、`universe`、`horizons`、`operating_cadence`、`stop_conditions` 和 `account_ids`。Context 引用必须指向当前确认的 Investor、Mandate 和 Attention Revision。同一时刻只允许一份 active 计划；Context 或账户改变后失败关闭，直到用户确认修订版。修订与状态变更必须携带最近读取的 `expected_version`；确认必须携带对应用户批准消息的 `user_approval_ref`。

## 研究机会

候选 Manifest 产生后必须拥有一条 `candidate_triage` Research Work。分流必须精确覆盖冻结候选集，每项只能进入完整研究、明确淘汰或限期观察。完整研究任务只有在绑定同一 Program 的 Opportunity、active Thesis 和达到 `eligible_for_bounded_action` 或 `eligible_for_decision` 的正式 Thesis Validation 后才能记为 promoted。Research Work 是协调队列，不复制证据、Decision、Execution 或 Ledger 真相。

```text
observed → researching → qualified → actionable
       └──────── active 阶段可 rejected / expired / closed
```

阶段只能逐级前进。qualified 和 actionable 必须引用真实 Research Validation Calculation；actionable 还必须绑定当前有效 Decision 和通过的 Risk Gate。`eligible_for_bounded_action` 只能绑定 `conditional_action + bounded Risk Gate`；`eligible_for_decision` 才可绑定正式 `action`。阶段不是 LLM 置信概率，也不允许用自报证据等级绕过验证。

## 用户行动队列

只接受 active actionable 机会。状态是 ready、presented、snoozed、accepted、rejected、expired、closed。accepted 只记录用户选择；它不创建 Execution、不写 Ledger、不连接券商。snoozed 必须有未来且早于失效时间的 `snoozed_until`。

## 简报与记分卡

所有简报包含 `summary`、`what_changed`、`decision`、`risks`、`next_check_at` 和 `queue_item_ids`。Weekly 增加 `program_progress` 与 `research_pipeline`；Monthly 增加 `scorecard_id`、`lessons` 与 `proposed_changes`。`no_action` 不得引用行动队列，也不得与未完成 Research Work 并存；`action` 至少引用一个当前有效队列项。

记分卡每个 metric 只能包含 `name + calculation_id + output_path`；模型不能直接填写 value。计算器标记为 insufficient_evidence 的收益、基准、用户时间或成本不得被模型补写。

## 失败关闭

Context 漂移、Decision 过期、Calculation 校验失败、候选缺少 Research Work、研究任务逾期、研究资格不足、风险闸门阻断、行情陈旧、队列未真实送达或交付失败时，不得用自然语言绕过工具错误。应向用户解释具体缺口和安全下一步。
