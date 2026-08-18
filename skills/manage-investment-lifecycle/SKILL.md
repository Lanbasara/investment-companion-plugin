---
name: manage-investment-lifecycle
description: 维护个人投资事实账本、账户、资产、现金、持仓、成交、对账、Investor/Mandate/Attention Policy 版本、Thesis、Decision、Execution、Review 和跨对话恢复。用户报告买卖成交、入出金、分红、费用、券商账单或人生约束，询问真实持仓、收益、组合暴露、调仓影响、历史决定、复盘、提醒偏好，或要求记录和恢复长期投资认知时使用。
---

# 管理投资生命周期

## 不可破坏的边界

- 把建议、Decision、Intent/Order 和 Execution 视为不同对象。只有用户确认的真实 Execution 或可信账单能产生已确认 Ledger Entry。
- 不让模型心算持仓、现金、收益、仓位或调仓影响；使用 Financial Kernel 工具并引用 Calculation ID。
- Ledger 只追加。错误使用 `ledger_reverse`，不覆盖或删除历史事实。
- 缺少账户、数量、币种、时间或成交口径时先创建待确认草稿或询问，不补猜。
- Simulation 永远不是成交。`trade_impact_simulate` 不改变真实持仓。
- V5 DecisionQueue 的 accepted 只表示用户选择，不表示下单或成交；不得据此创建 confirmed Ledger。

## 记录金融事实

1. 使用 `account_list` 与 `asset_list` 定位稳定身份；需要时创建 Account 或 Asset。
2. 把用户陈述转换为 `ledger_add` 草稿，逐项回显账户、类型、时间、数量、价格、金额、币种和费用。
3. 只有用户明确确认后调用 `ledger_confirm`。批量导入有差异时保持 `needs_confirmation`。
4. 用 `portfolio_state_as_of` 验证确认后的派生状态。账单使用 `portfolio_reconcile`；绝不自动补平差异。

详细符号、状态和工具纪律见 [financial-contract.md](references/financial-contract.md)。

## 形成投资判断

1. 使用 `context_current` 读取已确认 Investor、Mandate 和 Attention Policy；未确认时降低结论强度。
2. 使用 `portfolio_state_as_of` 获取真实组合。涉及交易规模时使用 `trade_impact_simulate`，不自行计算。
3. 读取相关 Thesis Revision 或使用 `recovery_package_create` 组装有界上下文。
4. 创建 Decision 后，用 `cognitive_revision_publish` 冻结 Investor Revision、Mandate Revision、Portfolio Calculation、Thesis Revision 和 Evidence Cutoff。
5. 用户决定行动时创建 Execution；只有确认流水后才能设置 `partially_filled` 或 `filled`。
6. 若存在 V5 Queue，成交确认后把 Review/Execution 结果交回 `$operate-investment-program`；Queue、Execution 和 Ledger 的 ID 必须保持可追溯但互不替代。

## 持续认知与复盘

- Thesis 发布完整不可变 Revision；新版本说明新证据、假设、估值、个人约束或措辞中的哪一项发生变化。
- 历史 Decision 永远引用具体 Revision，不能引用 `CURRENT.md`。
- Review 先判断当时信息下的过程质量，再判断结果；不得因一次好运升级 Principle。
- 跨会话先用 `recovery_package_create`，只读取返回的最少句柄，不扫描整个工作区。
- Investor、Mandate 或 Attention 发生新 current Revision 时，提醒 `$operate-investment-program` 重审 active Program；不得让旧 Program 静默沿用过期个人约束。

## 注意力治理

- 修改 Attention Policy 时先创建 Draft，回显静默时段、通知预算、冷却和场景覆盖；只有用户确认后启用。
- 负反馈使用 `attention_feedback`，它只能形成调整提案，不能自动改变 Policy。
- 主动消息必须先调用 `attention_decide`；遵守 `notify_now`、`queue_digest`、`file_only` 或 `suppress_duplicate`。
- 消息先说明“为什么现在联系你”，并提供继续观察、降低频率、暂不关注和误报反馈入口。

## 输出要求

明确区分：用户确认事实、外部市场事实、确定性计算、模型解释、假设和未知。涉及数字时给出 Calculation ID；涉及历史判断时给出精确 Revision；涉及主动消息时给出 Attention Decision ID。
