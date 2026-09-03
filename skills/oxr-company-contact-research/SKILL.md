---
name: oxr-company-contact-research
description: 从海外公司名单完成公司主体核验、资格判断、竞品排除、五类精细分型、Apollo/LinkedIn 双角色联系人检索与当前任职核验。适用于 OXR、Document AI、RAG、知识库和文档自动化的潜客研究；不用于直接发送邮件或未经授权回写业务系统。
---

# OXR 公司分类与目标联系人研究

## 目标

把只有公司名称或公司链接的名单，转成可解释、可复核的目标公司与联系人清单。最终为每家可找人公司提供至少一名 `technical_user` 和一名 `technical_decision_maker`，同时保留证据、置信度和缺口。

不要为了名单完整而使用离职人员、同名公司员工、只有泛 AI 背景的人或与目标工作流无关的高管。

## 按任务读取参考文件

- 需要判断公司分类、字段关系或生成搜索方向时，读取 [references/classification-rules.md](references/classification-rules.md)。
- 需要执行某一阶段或批量处理名单时，读取 [references/prompts.md](references/prompts.md)，选用对应 Prompt；不要默认把所有阶段合成一次推理。
- 需要交付 JSON、表格、评分或状态时，读取 [references/output-schema.md](references/output-schema.md)。

## 主流程

严格按下列门控顺序执行：

1. 标准化公司主体：确认标准名称、官网根域名、LinkedIn Company URL、品牌/子公司/母公司关系；以根域名去重。
2. 快速资格判断：寻找一条具体产品、项目、客户案例或招聘职责证据，确认公司确实存在目标文档、知识或自动化任务。
3. 排竞：分类描述公司业务形态，排竞决定能否触达；两者必须分开。
4. 精细分型：填写 `company_scenario`、`workflow_position`、`primary_task`、`target_team` 和 `company_size`。
5. 找人门槛：仅在公司主体、资格、排竞、任务、团队和证据均通过后生成联系人检索。
6. 宽召回：锁定 `Current Company`，分别搜索技术使用者和技术决策者；title 只用于扩大召回。
7. 职责排序：优先比较目标团队、当前任务和工作流位置，再比较职级。
8. 当前核验：核验 Current Company、Current Job Title、任期、团队及职责；发现离职立即淘汰。
9. 联系方式处理：人选正确性与邮箱可得性分开。Apollo 无邮箱不取消正确人选，继续补同 persona 的备选或使用其他合规渠道。

## 证据标准

优先使用当次可访问的当前来源：

- A 级：官网产品/案例、完整招聘职责、官方团队页或个人当前 Experience 直接说明产品、团队和任务。
- B 级：任务和技术栈明确，但产品线或团队名不完整；可以分类，找人前补团队证据。
- C 级：只有搜索摘要、第三方转述、`AI-powered`、`digital transformation` 等泛词；只能作为线索。

每项结论保存证据 URL 和采集日期。来源冲突时，以当前个人 Experience、官方团队页和完整职位信息优先。不得把搜索摘要当成最终当前任职证明。

## 公司状态与门控

- `qualified`：主体明确且有具体目标任务证据。
- `pending`：主体、任务或证据不足，下一步明确。
- `unqualified`：与目标需求无关。
- `excluded`：命中公司级竞品或其他明确排除规则。

`qualification_status` 与 `people_search_ready` 分开。公司可以已经 `qualified`，但因 `target_team` 不明确而暂时不能找人。

只有以下条件全部满足才设置 `people_search_ready=true`：

```text
domain_confirmed
AND qualification_status == qualified
AND competitor_rule != company_wide
AND company_scenario, workflow_position, primary_task, target_team 均明确
AND evidence_level in [A, B]
AND product_team_only 时产品线边界明确
```

门槛未通过时，输出 `pending_reason`、`missing_evidence` 和 `next_action`，不要扩大职位词强行找人。

## 双角色规则

- `technical_user`：直接构建、集成、部署、维护或日常使用目标工作流的人。
- `technical_decision_maker`：管理该团队，或负责该产品/平台的架构、工程或产品路线，能影响技术与供应商选择的人。
- 1–20 人公司允许同一人标记为 `dual_role`，但必须同时有实施和决策证据。
- 大公司从技术使用者所在团队向上找负责人，不默认集团 CTO、CIO 或最高级别高管。
- 招聘人员、销售人员和岗位发布人只能作为 `routing_contact`，不能替代双角色。

Apollo 首轮使用 `Current Company + 宽职位族`，关闭 Past Job Titles。结果过多时才添加管理层级、地区或产品线词。人工每类最多查看 10 人，保留最多 3 人进入核验。

## 候选状态

- `selected`：当前公司、当前职位和职责均能解释，与目标团队和任务匹配。
- `pending_A`：基本匹配，只缺一次团队或职责核验。
- `pending_B`：只有相邻团队、旧资料或技能背景，不能直接触达。
- `rejected`：离职、公司错配、竞品范围、职责无关或身份冲突。

硬门槛优先于评分。离职人员即使历史项目高度相关，也必须标记 `rejected_departed`。

## 邮箱处理

`person_status` 与 `email_status` 独立：

```text
person_status = selected
email_status = not_found | found_unverified | verified_valid | risky_catch_all | invalid
outreach_ready = person_status == selected AND email_status == verified_valid
```

Apollo 无邮箱时保留正确人选；查找公开业务邮箱或已授权的第二数据源，并独立验证。若邮件是强制渠道，再从同一团队、同一 persona 的第 2 或第 3 候选补邮箱。不得把未验证的格式推导邮箱直接用于发送，也不得为获得邮箱换成职责不相关的人。

## 权限边界

默认只输出建议与证据。除非用户明确授权，不写回飞书/Airtable，不解锁付费联系方式，不发送邮件或 LinkedIn 消息。任何回写前先读取当前字段定义，保护人工审核字段，写后回读数量、状态和重复域名。

## 完成标准

只有以下条件全部满足，批次才标记完成：

- 每家公司的根域名唯一，实体边界可解释。
- 每家 `qualified` 公司至少有一条具体任务证据。
- 分类、排竞、任务、工作流位置和目标团队均有合法值。
- 每家 `people_search_ready` 公司至少有一名已核验技术使用者和一名已核验技术决策者，或明确标记 `partial_buying_group`。
- 所有 `selected` 人员都有当前任职和职责证据，已知离职人员为零。
- 邮箱缺失不会改变人选状态；只有 `verified_valid` 计入邮件可触达。
- 输出能追溯每项结论的来源、日期和下一步。
