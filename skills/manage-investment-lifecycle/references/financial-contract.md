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

`investment_decision_publish` 至少冻结 Investor Revision、Mandate Revision、Portfolio Calculation、Thesis Revision、知识截止时间、不行动方案、替代方案与失效条件。行动型 Decision 还必须引用通过的 Research Validation 和 Risk Gate Calculation。

## 数据质量

行情质量可以是 `healthy / stale / partial / conflicting / unauthorized / failed / unknown`。陈旧、冲突、缺失和无权限必须变成 Warning 或 Risk Gate 阻断，不得解释成“没有变化”。
