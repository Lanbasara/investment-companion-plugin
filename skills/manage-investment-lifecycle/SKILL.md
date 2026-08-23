---
name: manage-investment-lifecycle
description: 维护个人投资事实账本、账户、资产、现金、持仓、成交、对账、Investor/Mandate/Attention Policy、Decision、Execution、Performance、Review 和跨对话恢复。用户报告买卖成交、入出金、分红、费用、券商账单或人生约束，询问真实持仓、收益、暴露、历史决定或复盘时使用。
---

# 管理投资生命周期

## 不可破坏的边界

- 建议、Decision、下单意图、券商订单和确认成交是不同事实。只有用户确认的 Ledger Entry 改变现金与持仓。
- 不让模型心算金额、收益、仓位或调仓影响；使用 `portfolio_context`、`investment_action_plan` 和 `investment_performance_calculate`。
- 不覆盖或删除已确认历史。更正时使用 `investment_transaction_update(operation="reverse")`。
- 缺少账户、数量、币种、时间或成交口径时，只登记待确认事实或先询问，不补猜。
- 仿真、行动卡接受和用户说“准备买”都不是成交。

## 恢复真实状态

1. 新会话或需要完整投资上下文时，先调用 `investment_home`。
2. 涉及账户、现金、持仓、待确认成交或硬约束时，调用 `portfolio_context`。
3. 必须读取 `truth_freshness` 和 `precision_boundary`：正文明确说出最近对账时间；状态为 `stale` 时只能称“账本持仓”，不得称当前真实持仓，不得给出依赖精确数量或现金的仓位建议，并请求当前账单、持仓截图或成交记录完成对账。
4. 涉及过去决定、执行和绩效时，分别使用 `decision_context` 和 `evaluation_context`。不从聊天或 Markdown current 视图推断精确事实。

## 记录金融事实

1. 缺少账户或资产稳定身份时，使用 `investment_transaction_update` 的 `account_create` 或 `asset_register`。
2. 把用户陈述转换为 `operation="record"`；回显账户、类型、时间、数量、价格、金额、币种与费用。返回的流水仍是 `needs_confirmation`。
3. 只有用户明确确认后，才调用 `operation="confirm"`。随后重新读取 `portfolio_context` 验证派生状态。
4. 券商账单使用 `operation="reconcile"`；差异保留待核对，绝不自动补平。

详细符号和状态见 [financial-contract.md](references/financial-contract.md)。

## 个人约束与投资决定

- Investor、Mandate 或 Attention Policy 先用 `investment_context_update(operation="draft")` 生成草稿，回显后经用户确认，再用 `operation="confirm"` 生效。
- 人生事实或约束变化后，使用 `$operate-investment-program` 复核当前计划，不让旧计划静默沿用过期条件。
- 正式 Decision 使用 `investment_decision_publish`；必须冻结当前 Context、Portfolio Calculation、Thesis、替代方案和失效条件。
- 用户接受行动后，使用 `investment_execution_update` 分别记录 prepare、order、report_fill 和 confirm_fill。报告成交仍不改变持仓，confirm_fill 只能引用用户确认的 Ledger Entry。

## 绩效与复盘

- 客观期间结果使用 `investment_performance_calculate`，明确期初期末价格、基准、现金流和来源。
- 解释与修订提案使用 `investment_review_publish`。Review 只能提出新版本，不能改写历史或在线调参。
- 每次输出明确区分：用户确认事实、外部事实、确定性计算、模型解释、假设和未知。
