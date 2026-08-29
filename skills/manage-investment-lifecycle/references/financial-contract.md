# Financial Kernel 契约

## Ledger 符号

- 买入：`quantity > 0`，现金 `amount < 0`。
- 卖出：`quantity < 0`，现金 `amount > 0`。
- 入金与收入：`amount > 0`；出金、费用和税：`amount < 0`。
- `fee` 表示额外现金流出，Portfolio 以 `amount - fee` 计算现金变化。
- 数值使用十进制字符串，禁止二进制浮点中间结果。

## 确认状态

`draft / needs_confirmation` 不影响持仓；`confirmed` 进入状态重建。错误的 confirmed 记录使用 `investment_transaction_update(operation="reverse")` 产生不可变冲销流水，原记录仍保留在历史中。

## Decision 冻结

`investment_decision_publish` 至少冻结 Investor Revision、Mandate Revision、Portfolio Calculation、Thesis Revision、知识截止时间、不行动方案、替代方案与失效条件。行动型 Decision 还必须引用 Action Plan 与 Risk Gate 共用的当前 `preflight_ready` candidate Portfolio Qualification Calculation，以及通过的 Research Validation 和 Risk Gate Calculation；历史 Decision 保留当时资格 ID 与事实血缘，不用当前事实覆盖。

## 数据质量

行情质量可以是 `healthy / stale / partial / conflicting / unauthorized / failed / unknown`。陈旧、冲突、缺失和无权限必须变成 Warning 或 Risk Gate 阻断，不得解释成“没有变化”。

## 人工账本连续性

全量 matched 对账是锚点；用户通过 `continuity_confirm` 确认锚点之后没有漏报交易、现金流、收入/费用/税、公司行动或未结订单，并承诺之后及时报告。该确认允许系统继续使用 Ledger 数量和现金生成仓位方案，市场价格必须另行刷新。待确认流水、对账差异或用户报告遗漏会阻断或撤销精确数量；没有连续性确认时仍输出范围和条件化数量。最终下单前始终核对券商 App 的可用现金与可用持仓。
