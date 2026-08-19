---
name: research-investment
description: 用结构化市场数据、官方原始资料和 Web 证据完成专业、可追溯的个人投资研究。用户要求分析股票、ETF、基金、指数、行业或公司，核验代码与数据，比较标的，撰写投资报告，检查投资 Thesis，或询问某项市场变化意味着什么时使用。
---

# 研究投资问题

## 核心契约

让 Primary Codex 保持唯一的理解者和最终作者。工具提供事实，Custom Agent 提供有边界的专业意见；它们都不能代替 Primary Codex 做最终综合。

不得把搜索结果、Tushare 聚合数据或辅助 Agent 的结论直接当作最终答案。区分事实、计算、估计、解释和未知项。

## 工作流

1. 定义问题：写清标的、决策问题、时间范围、截至时间和对个人组合的相关性。代码或资产类型不明确时，先解析身份，不能猜。
2. 读取依据：材料性研究必须读取 [source-routing.md](references/source-routing.md) 和 [report-standard.md](references/report-standard.md)。涉及公司财务、基金结构或估值时，再读取 [financial-analysis.md](references/financial-analysis.md)。
3. 取得证据：按来源职责直接使用 Tushare MCP、Codex Web 和 Tavily。记录数据日期、口径、单位与原始 URL；权限不足或冲突必须显式保留。
4. 专业委派：简单事实由主 Codex 直接完成。涉及未来走势、估值、公司财务、投资 Thesis、标的比较、买卖/仓位判断或完整报告时，属于材料性研究，必须至少委派一个具名专业 Agent，不需要用户再次要求。多来源核验使用 `source_researcher`；财务、估值、历史统计或价格区间使用 `financial_analyst`；包含预测、推荐或高影响判断时，主 Codex 形成初步 Thesis 后必须使用 `thesis_critic` 寻找足以改变结论的反证。
5. 独立综合：主 Codex 比较冲突、补足关键缺口并亲自撰写报告。不得平均辅助意见、按票数表决，或让“报告写手 Agent”拼装最终答案。
6. 维护连续性：只有研究达到材料性门槛时，才更新 `theses/` 或 `library/`；在 `memory/now.md` 记录仍需观察的问题。不要把一次搜索过程当作长期记忆。
7. 若研究属于 active InvestmentProgram 的机会，使用 `$operate-investment-program` 保存证据并推进或淘汰 Opportunity。研究报告不能自己跳过阶段、入 DecisionQueue 或生成个人买卖指令。

若所需 Agent 调用失败或不可用，必须说明缺失的复核层并降低结论强度；不得静默退化为未经复核的完整报告。最终报告简要披露实际参与的专业 Agent、关键质疑以及主 Codex 对分歧的处理。

## 来源纪律

- 中国股票、基金、指数、财务与历史行情等结构化问题，先用 Tushare MCP。
- 公告、基金合同、指数编制规则和监管事实，以交易所、基金公司、监管机构等原始发布者为最高权威。
- Codex Web 用于普通发现和直接读取官方来源；Tavily 用于有日期或域名约束的系统发现，以及已知 URL 的正文提取。
- Tavily 和搜索摘要只是传输与发现层。最终引用底层原始网页，不写“根据 Tavily”。
- Tushare 数据不足时，说明具体权限或字段缺口后再走替代来源；禁止静默退化成搜索引擎猜测。
- 主动 Patrol 必须留下来源覆盖回执。没有达到最低检查来源数或原始来源数时，证据状态只能是 `insufficient_coverage`；“本地没有新增文件”不能推出“市场没有新信息”。

## 交付标准

第一行先给结论或当前最佳判断。随后只展开会影响判断的证据、解释、组合意义、反方证据与失效条件。

每个材料性数字都附带截至日期、单位与来源。每个推断都明确标记为分析而不是事实。资料不足时降低结论强度，不用篇幅掩盖未知。

投资研究不等于交易建议；当用户进一步询问买入、卖出、持有、仓位或资产配置时，切换到 `$decide-investment`。
