---
name: decide-investment
description: 把研究证据、人生目标、流动性、组合约束和机会成本综合成可解释的个人投资判断。用户询问买入、卖出、持有、减仓、仓位、资产配置、候选标的比较，或要求评估一项投资决定时使用。
---

# 做出投资判断

## 核心契约

Primary Codex 对最终判断负责。先保证事实足以支持决定，再比较行动方案；不让格式、投票或单一估值替代判断。

推荐、用户决定、实际成交是三个独立事实。永远不替用户下单，也不把建议记录成已执行交易。

## 工作流

1. 使用 `$operate-investment-program` 读取当前 InvestmentProgram；使用 `$manage-investment-lifecycle` 和 `context_current` 读取生效的 Investor、Mandate 与 Attention Policy，并用 `portfolio_state_as_of` 从确认流水重建组合。Markdown 仅作可读材料，不作为精确持仓事实源。
2. 明确决定：标的、方向、资金来源、期限、触发原因和用户真正要解决的问题。
3. 检查证据：如果身份、基本事实、估值或风险信息不足，先使用 `$research-investment`，不要在不完整研究上制造确定答案。
4. 建立基准：始终比较“不行动”、用户提出的行动，以及至少一个真实可行的替代方案；把现金和等待视为正式选择。
5. 检查组合：涉及仓位、现金和交易影响时必须调用 Financial Kernel 的确定性计算或模拟，引用 Calculation ID；不得让模型心算。
6. 形成建议：主 Codex 亲自给出结论、理由、适用条件、失效条件和下一观察点。读取 [decision-standard.md](references/decision-standard.md) 执行材料性决策。
7. 保持事实边界：Decision 发布时冻结 Context、Portfolio Calculation 和 Thesis Revision。若它关闭一个 qualified Opportunity 的全部重大未知项，才通过 `$operate-investment-program` 推进 actionable 并进入 DecisionQueue；不得直接创建成交。

## 输出

第一行直接回答“做什么”。随后依次说明：

- 为什么是现在；
- 哪些事实真正驱动判断；
- 对整个组合而非单一标的的影响；
- 最强反方、失效条件与下一观察点；
- 用户需要执行、确认或暂时不做的具体下一步。

结论强度必须与证据强度匹配。除非 Mandate、精确组合和交易摩擦都已知，否则仓位只给条件化范围，不给伪精确百分比。

用户接受行动卡仍不是成交。只有用户手工执行并明确确认的真实流水才能改变持仓。
