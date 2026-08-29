---
name: decide-investment
description: 把已验证研究、人生目标、流动性、确认组合、硬风险约束和机会成本综合成可解释的个人投资判断。用户询问买入、卖出、持有、减仓、仓位、资产配置、候选比较或要求评估一项投资决定时使用。
---

# 做出投资判断

## 核心契约

Primary Investment Codex 对最终判断负责。推荐、用户选择、券商订单和确认成交是四个独立事实。系统不替用户下单，不把建议或接受行动记录为已执行交易。

## Compatibility Gate

先读取 `investment_home.production_health`。`baseline.status` 不是 `compatible` 时停止形成或发布可信投资判断，并逐项报告 baseline incidents；`workflows.decide-investment.status` 不是 `compatible` 时只停用本工作流，不把失败扩大到已验证的其他工作流。optional enhancement 返回 `fallback` 时执行其声明的 `fallback`，说明降级原因，禁止静默成功。

## 工作流

1. 先调用 `investment_home`，再用 `investment_program_context`、`portfolio_context` 和 `research_context` 恢复当前计划、确认组合、Investor/Mandate 与研究验证。不从 Markdown 或聊天历史猜精确事实。
2. 明确决定：标的、方向、资金来源、期限、触发原因和用户真正要解决的问题。
3. 检查证据：身份、基本事实、估值、反证或风险信息不足时，先使用 `$research-investment`。`eligible_for_bounded_action` 最多支持条件行动；正式行动要求 `eligible_for_decision`。`research_only` 不得形成行动卡。
4. 建立基准：始终比较“不行动”、用户提出的行动，以及至少一个真实可行的替代方案。现金和等待是正式选择，不是分析失败。
5. 冻结市场事实：决策使用的价格、汇率和流动性观测使用 `investment_evidence_update(operation="market_snapshot")` 登记，不使用聊天中的暂存数字。
6. 构建行动方案：涉及仓位、现金和交易影响时调用 `investment_action_plan`。全量对账日期已久但没有差异时，不得自动退化为“不行动”：缺少连续性确认则给仓位范围和条件化数量；用户确认无漏报并承诺持续报告后，记录 `continuity_confirm` 并生成具体方案。市场行情单独刷新，最终下单前始终核对券商 App 的可用现金和可用持仓。正式行动使用 `action_tier="standard"`；证据仅支持受限行动时使用 `action_tier="bounded"`，并提供券商实际支持的 `validity_sessions`。后者的上限和允许执行类型必须来自当前确认 Program，工具返回阻断时不得自行缩小数字后假装通过。首版受限通道不使用网格；网格只走完整研究资格的 standard 通道。
7. 发布正式判断：使用 `investment_decision_publish` 冻结内容、知识截止、有效期、Thesis、来源、失效条件、不行动和替代方案。正式行动使用 `decision_kind="action"`；受限行动使用 `decision_kind="conditional_action"`。两者必须引用 Action Plan 返回的同一份当前 `preflight_ready` candidate Portfolio Qualification Calculation，以及匹配等级且通过的 Research Validation 和 Risk Gate Calculation；不得自行重算或替换资格 ID。watch 若引用较低资格，只能携带 `allowed_uses` 允许的数量精度，且不得进入 Action Card 队列。
8. 维护行动闭环：只有 Decision 关闭了 qualified 机会的重大未知项，才通过 `$operate-investment-program` 推进 actionable 并用 `investment_action_update(operation="enqueue")` 进入用户队列。不得直接创建成交。

材料性决策必须读取 [decision-standard.md](references/decision-standard.md)。

## 输出

第一行直接回答“做什么”，并标明行动等级：行动、条件行动、观察或不行动/退出。随后只说明：

- 为什么是现在；
- 哪些事实真正驱动判断；
- 对整个组合的影响；
- 最强反方、失效条件和下一观察点；
- 用户需要执行、确认或暂时不做的具体下一步。

结论强度必须与证据强度匹配。除非 Mandate、账本连续性、实时风险和交易摩擦都已冻结，否则仓位只给条件化范围，不给伪精确比例；但缺少精确数量资格不等于缺少投资判断，不能用它逃避方向、范围和替代方案。
