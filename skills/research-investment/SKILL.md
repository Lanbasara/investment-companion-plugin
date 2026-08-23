---
name: research-investment
description: 用结构化市场数据、官方原始资料和 Web 证据完成专业、可追溯的个人投资研究。用户要求分析股票、ETF、基金、指数、行业或公司，核验代码与数据，比较标的，撰写投资报告，检查 Thesis，或询问某项市场变化意义时使用。
---

# 研究投资问题

## 核心契约

Primary Investment Codex 是唯一最终理解者和作者。数据工具提供事实，专业 Agent 提供有边界复核；它们都不代替 Primary 完成独立综合。

扫描榜单、预测候选、搜索摘要和 Agent 意见都是研究材料，不是 Decision、行动或成交。历史 Pipeline 名称不等于正式 StrategyVersion。

## 工作流

1. 定义问题：写清标的、决策问题、时间范围、知识截止时间和对用户组合的相关性。身份不明时先核验，不猜代码。
2. 恢复现状：使用 `research_context` 读取统一 ResearchRecord、正式 Validation 和待办；需要组合关联时使用 `portfolio_context`。
3. 读取规范：材料性研究必须读取 [source-routing.md](references/source-routing.md) 和 [report-standard.md](references/report-standard.md)；涉及财务、基金结构或估值时再读取 [financial-analysis.md](references/financial-analysis.md)。
4. 获取证据：使用 Tushare MCP、官方原始页面和受约束 Web 发现。记录事件时间、发布日、数据截止日、观察时间、口径、单位和原始 URL。
5. 冻结来源：对每个材料性来源调用 `investment_evidence_update(operation="publish_source")`。`source_group` 必须表示真实独立来源组，不得为了通过验证而伪造独立性。价格、汇率和其他决策时点观测使用 `operation="market_snapshot"`。
6. 专业复核：简单事实由 Primary 完成。涉及未来走势、估值、公司财务、Thesis、标的比较或买卖/仓位的材料性研究，至少委派一个具名专业 Agent。包含预测或推荐时，必须使用 `thesis_critic` 寻找可改变结论的反证。
7. 正式发布：使用 `investment_research_publish` 冻结 Thesis、Evidence Manifest ID、知识截止时间和可选 Validation Spec。没有通过 Validation 时仍然只是 `research_only`。
8. 维护闭环：属于当前投资计划的问题，使用 `$operate-investment-program` 登记、推进或淘汰机会。研究不能自己跳到用户行动或交易。

若所需 Agent 失败或不可用，明确说明缺失的复核层并降低结论强度；不静默退化为“已完成专业复核”。

## 交付标准

第一行先给截至绝对日期的当前最佳判断和证据状态。随后只展开会改变判断的关键事实、解释、组合意义、最强反方、未知与失效条件。每个材料性数字附日期、单位和来源；推断明确标记为分析。

投资研究不等于交易建议。当用户进一步询问买入、卖出、持有、仓位或资产配置时，切换到 `$decide-investment`。
