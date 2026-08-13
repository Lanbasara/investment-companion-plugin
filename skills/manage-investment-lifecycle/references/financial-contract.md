# Financial Kernel 契约

## Ledger 符号

- 买入：`quantity > 0`，现金 `amount < 0`。
- 卖出：`quantity < 0`，现金 `amount > 0`。
- 入金与收入：`amount > 0`；出金、费用和税：`amount < 0`。
- `fee` 表示额外现金流出，Portfolio 以 `amount - fee` 计算现金变化。
- 数值使用十进制字符串，禁止二进制浮点中间结果。

## 确认状态

`draft / needs_confirmation` 不影响持仓；`confirmed` 进入状态重建；错误的 confirmed 记录由等额反向流水冲销，原记录标为 `reversed` 但仍参与历史重放。

## Decision 冻结

Decision 发布至少引用：

- `investor_revision_id`
- `mandate_revision_id`
- `portfolio_calculation_id`
- `thesis_revision_ids`

所有材料性计算同时写入 `calculation_ids`。缺少任一项时不得发布完整 Decision。

## 数据质量

行情质量可能是 `healthy / stale / partial / conflicting / unauthorized / failed / unknown`。陈旧、冲突或缺失必须成为 Warning；不得解释成没有变化。
