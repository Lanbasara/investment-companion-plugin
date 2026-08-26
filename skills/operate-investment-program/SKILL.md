---
name: operate-investment-program
description: 统一管理“今天做什么”、投资计划、机会漏斗、用户行动、日周月简报与结果记分。用户询问如何使用投资伴侣、今天是否行动、候选如何进入或淘汰、周度检查、月度成效，或希望系统持续改善投资过程时使用。
---

# 经营个人投资系统

## 核心契约

用户第一层只看到：当前计划、研究、行动、真实结果和下次检查。不向用户暴露 Pipeline 版本、Manifest、Gate、Job 或数据库对象，除非它们发生故障并直接影响结论。

投资计划、研究机会、行动队列和简报是协调层，不是第二套事实。持仓只来自 confirmed Ledger，个人约束只来自 confirmed Context，真实成交始终由用户手工执行并确认。

## 每次进入

1. 先调用 `investment_home`，不根据聊天历史猜测当前状态。
2. 读取返回的 `production_health`；只要关键运行版本、服务或研究流水线失败，结论必须是 `system_degraded`，不得写成 `no_action`。先报告故障、修复或触发恢复，再重新判断。
3. `setup_required`：读取 `portfolio_context` 和 `investment_program_context`，与用户确认目标、基准、风险、范围、节奏和停止条件。使用 `investment_program_update(operation="create")` 产生草稿；只有用户批准后才 `operation="confirm"`。
4. `action`：读取 `decision_context`，最多展示三个最重要行动，说明有效期、阻断条件、不行动和替代方案。
5. `no_action`：说明这是“全部关键链路成功、已检查但没有达到行动门槛”，不扩写成对市场的确定预测。

## 研究到行动的闭环

1. 新线索先用 `$research-investment` 冻结来源，再用 `investment_opportunity_update(operation="create")` 登记研究问题。这不是推荐。
2. 使用 `research_context` 读取当前 ResearchRecord 和 Validation；历史扫描、预测和信号只是研究输入，不自动成为 StrategyVersion。
3. 每个交易日必须处理最新的 `provisional_action` 和持续候选：选择“进入完整研究 / 明确淘汰 / 继续观察并给出期限”之一。不得仅抄录榜单后结束；至少把最有希望且资料可补齐的候选推进为 ResearchRecord 和正式 Validation。
4. Research Validation 决定允许的最大行动强度：`eligible_for_bounded_action` 只允许受限条件行动，`eligible_for_decision` 才允许正式行动。两者都可推进 qualified；推进 actionable 还必须绑定匹配等级的 Decision 和 Risk Gate。使用 `investment_opportunity_update(operation="transition")`，不用自报“证据等级”替代 Calculation。
5. 涉及买卖、仓位或资产配置时切换到 `$decide-investment`。正式行动和受限条件行动都必须通过 Research Validation 和 Risk Gate；受限行动还要求当前确认 Program 显式启用 bounded policy，缺少政策时不得用自然语言补一个仓位。
6. 只有 active actionable 机会才用 `investment_action_update(operation="enqueue")` 进入用户队列。队列不得直接创建 Execution 或 Ledger。

详细状态与失败关闭见 [operating-contract.md](references/operating-contract.md)。

## 用户行动

- 呈现、延后、接受、拒绝和关闭统一使用 `investment_action_update(operation="respond")`。用户的后续回复可以作为上一条消息已送达的证据；先用 `investment_delivery_update` 补记 Attention 送达，再记录队列响应。
- 接受不代表下单或成交。用户真实手工操作后，使用 `$manage-investment-lifecycle` 记录订单、待确认成交和确认账本。
- 有效期、账本、价格、Mandate 或 Decision 变化时，旧行动失效；生成新 Decision 和队列项，不修改旧记录。

## 日、周、月输出

- 使用 `investment_brief_update(operation="publish")` 冻结日/周/月简报。`no_action` 只有在 Production Doctor 全绿、当日研究产物新鲜且没有有效行动队列时成立；`action` 必须引用有效队列项。
- 月度先用 `operation="metrics_calculate"` 冻结过程指标，再用 `investment_performance_calculate` 得到真实投资结果，最后用 `operation="scorecard_publish"` 发布可追溯记分卡。模型不能手填绩效数值。
- 调度触发的用户结果使用 `investment_delivery_update(operation="prepare")`；摘要使用 `digest_send`。Run 成功不等于用户已收到结果。

## 输出给用户

第一行直接给“行动 / 不行动 / 需要补什么”。最多展示当前最重要的三项。不承诺高胜率、Alpha 或盈利；系统的价值用真实结果、风险、成本和用户时间衡量。
