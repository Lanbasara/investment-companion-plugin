---
name: operate-investment-program
description: 以 V5 投资经营闭环统一管理“今天做什么”、InvestmentProgram、机会漏斗、行动卡、日/周/月简报与结果记分卡。用户询问如何使用投资伴侣、今天是否行动、投资体系如何推进、候选如何进入或淘汰、周度投资委员会、月度成效，或希望系统持续改善投资过程时使用。
---

# 经营个人投资系统

## 核心契约

把复杂后台组织成一个用户能长期运行的投资经营闭环。先问“当前计划和证据允许做什么”，再调用研究、决策和生命周期能力；不要从搜索直接跳到股票推荐。

InvestmentProgram、Opportunity、DecisionQueue、Brief 和 Scorecard 是协调层，不是第二套事实。持仓来自 confirmed Ledger，个人约束来自 confirmed Context，研究来自不可变证据，正式判断来自 Decision，真实成交始终由用户手工执行并确认。

## 每次进入

1. 新会话或用户问“今天做什么”时，先确认 MCP 是否提供 `v5_today`，或用 `system_status` 核对 Schema。生产仍低于 Schema 5 / 工具不存在时，不得假装 V5 已启用：明确说明尚未切换，退回现有 `$manage-investment-lifecycle` 与 `$manage-investment-companion` 的只读恢复入口，不创建 V5 对象。
2. V5 可用时调用 `v5_today`。不要扫描整个工作区，也不要根据聊天历史猜状态。
3. `setup_required`：使用 `$manage-investment-lifecycle` 读取当前 Context、账户和组合，与用户共同起草 InvestmentProgram；展示目标、基准、风险、范围、节奏和停止条件。只有用户明确批准后调用 `v5_program_confirm`。
4. `action`：逐项调用 `v5_action_card`，优先说明有效期、阻断条件、不行动方案和需要用户手工完成的下一步。
5. `no_action`：说明这是“本轮经过检查后没有达到行动门槛”，不要扩写成市场判断。
6. `review_required`：完成本期有边界的检查，资料不足时形成 insufficient-evidence 或待复核结论，不为了完成日报生成候选。

## 机会闭环

1. 新线索先保存不可变来源，再以 `v5_opportunity_create` 登记 observed；这不是推荐。持续量化扫描提供一个不可变来源、候选变化和持续性触发，不能把整张榜单机械拆成多个 Opportunity；满足 `research_shortlist` 或出现其他重大证据时可立即开始完整研究，不等待月末。
2. Primary 接受明确研究问题后才推进 researching，并使用 `$research-investment`。材料性研究继续遵守官方来源、结构化数据、反证和专业 Agent 复核。
3. 只有 active Thesis 或合格 Strategy、至少两个冻结来源、明确 falsifier/counterevidence 后才推进 qualified。
4. 只有当前数据、无重大未知项和 current issued Decision 才推进 actionable。涉及个人买卖、仓位或资产配置时必须使用 `$decide-investment`。
5. actionable 后用 `v5_decision_queue_enqueue` 入队。不得从 Opportunity 直接创建 Execution 或 Ledger。

详细字段、状态和禁止捷径见 [operating-contract.md](references/operating-contract.md)。

## 用户行动

- 呈现行动卡前调用 `attention_decide`，其 evidence 必须引用对应 Queue/Brief。只有实际飞书送达并 `attention_mark_delivered` 后，才把对象标记 presented。
- 用户可以接受、拒绝、稍后处理或要求补证据。稍后处理必须保存明确的 `snoozed_until`；记录真实选择，不劝用户为了闭环而接受。
- 接受 Queue 不代表成交。有 ManualActionSpec 时重新验证；用户手工下单并报告真实结果后，切换到 `$manage-investment-lifecycle` 创建待确认 Ledger/Execution 记录。
- accepted 在有效期内仍是待处理行动；完成成交记录、不再执行或过期后才关闭/失效，不能用 no-action 覆盖。
- 有效期、账本、价格、Mandate 或 Decision 变化时，旧行动卡失效；生成新 Decision/Spec/Queue，不修改旧卡。

## 日、周、月输出

- 日：读取当天最新量化扫描，将候选进入/退出、持续性、完整研究触发与持仓/Thesis 一起解释；只报告异常、有效行动或经过检查的 no-action。量化扫描不是行动，但不能因为前向样本尚少而从日报消失；没有运行证据时写 review_required。
- 周：汇总 Program 进展、机会推进/淘汰、研究管道、组合风险和下周一件最重要的事。
- 月：读取 `v5_continuous_quant_review` 并报告其确定性前向指标；再调用 `v5_program_metrics_calculate` 生成过程指标 Calculation，补充已有的合格投资结果 Calculation并生成 Scorecard；解释结果、过程质量、用户时间/Token 成本和需要修订/停止的部分。证据不足降低结论强度，不关闭每日功能。

Scorecard 指标只提交 `name + calculation_id + outputs path`。不得提交模型填写的 value；计算器标出的收益、基准、时间或成本覆盖缺口必须原样披露，没有合格 Calculation 时发布 insufficient-evidence。

## 输出给用户

第一行直接给“行动 / 不行动 / 需要补什么”。最多展示当前最重要的三项。内部 Gate、Job、Snapshot 和 Worker 只在故障影响结论时解释，并翻译成人能理解的影响。

不承诺高胜率、Alpha 或盈利。系统的进步以真实淘汰、可重放结果、风险控制和扣除时间/Token 后的净价值衡量。
