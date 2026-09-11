---
name: oxr-company-contact-research
description: 从已确认可找人的公司名单或候选名单出发，推导目标团队、完成双角色找人、当前任职核验和缺口补搜；用户明确说明是原始名单或要求筛选时，才执行公司资格判断与排竞。适用于 Document AI、RAG、知识库和文档自动化潜客研究；不用于直接发送消息、抓取 Apollo 页面或未经授权回写业务系统。
---

# OXR 零上下文公司与目标联系人研究

## 目标

把已确认可找人的公司名称、公司链接、Apollo Company List 或候选导出，转成可解释、可复核的目标联系人清单。最终为每家公司选出最合适的 `technical_user` 和 `technical_decision_maker`；有必要时各补一名备选，同时保留证据、已尝试路径和下一步。

不要为了名单完整而使用离职人员、同名公司员工、只有泛 AI 背景的人或与目标工作流无关的高管。

## 零上下文启动

先识别输入，不要求用户复述完整策略：

- 只有公司名、官网、LinkedIn Company URL 或 Company List：默认这些公司已经完成资格判断与排竞，可以直接找人。只确认公司主体并补足推导目标团队所需的信息，不重新判断公司是否合格，也不因公开公司证据不足停止找人。
- 用户明确说这是原始/未筛选名单，或要求判断公司、分类、排竞：执行完整公司到联系人流程。
- 已有目标团队或五字段分类：直接复用，从宽召回开始；不重新研究已经给出的分类。
- 已有 Apollo/CSV 候选：直接做实体、现任、团队、职责与 persona 核验，不重新设计公司分组。
- 同时有公司和候选：以公司边界校验候选，不重新搜索已覆盖人员。

用户对名单“已经筛选、可以找人”的声明优先。公司主体核验只用于避免同名公司、母子公司或品牌错配；已知产品线限制只用于避开错误团队，不能借此重新否定整家公司。若主体存在无法消除的歧义，再询问一个最小澄清问题。

若用户没有提供任何公司、链接、Company List 或候选名单，只追问一个最小问题：`请提供公司名称/链接或候选名单。` 不追问 ICP、persona、评分、输出格式或邮箱要求；这些使用 [references/default-brief.md](references/default-brief.md) 的默认值。若用户明确给出不同产品、竞品或角色口径，以用户口径覆盖默认值。

默认交付为简表，每家公司、每个 persona 各选 1 名最佳人选；需要完整 JSON、批次统计、Apollo 查询配置或邮箱时再展开。若证据不足，明确写 `无法判断`，不把低质量候选凑成答案。

## 按任务读取参考文件

- 需要判断公司分类、字段关系或生成搜索方向时，读取 [references/classification-rules.md](references/classification-rules.md)。
- 零上下文启动、用户未说明产品/竞品/persona/输出时，读取 [references/default-brief.md](references/default-brief.md)。
- 需要执行某一阶段或批量处理名单时，读取 [references/prompts.md](references/prompts.md)，选用对应 Prompt；不要默认把所有阶段合成一次推理。
- 需要交付 JSON、表格、评分或状态时，读取 [references/output-schema.md](references/output-schema.md)。

## 默认找人流程

1. 确认公司主体：核对标准名称、根域名、LinkedIn Company URL 和品牌/母子公司关系；这一步不重新做资格判断。
2. 确定搜索边界：复用用户已有分类和目标团队；没有这些字段时，根据公司当前产品、项目、招聘或团队信息生成找人假设，但不因证据不足阻断搜索。
3. 宽召回：锁定 `Current Company`，分别搜索技术使用者和技术决策者；title 只用于扩大召回。
4. 职责排序与当前核验：核验 Current Company、Current Job Title、团队和职责；发现离职、实体错配或错误产品线立即淘汰。
5. 缺口补搜：首轮没有合格人选或只找到一个 persona 时，执行第二轮职位扩展和第三轮组织关系反查。
6. 最佳人选：先过硬门槛，再按职责直接性、团队精确度、persona 权限和证据质量排序；职位级别只作辅助。
7. 联系方式处理：人选正确性与邮箱可得性分开。Apollo 无邮箱不取消正确人选，继续补同 persona 的备选或使用其他合规渠道。

只有原始名单模式才在第 1 步后执行“快速资格判断 → 排竞 → 精细分型 → 找人门槛”。已有可靠结果就复用，不要求用户重新提供上下文。

## 证据标准

优先使用当次可访问的当前来源：

- A 级：官网产品/案例、完整招聘职责、官方团队页或个人当前 Experience 直接说明产品、团队和任务。
- B 级：任务和技术栈明确，但产品线或团队名不完整；可以分类，找人前补团队证据。
- C 级：只有搜索摘要、第三方转述、`AI-powered`、`digital transformation` 等泛词；只能作为线索。

每项结论保存证据 URL 和采集日期。来源冲突时，以当前个人 Experience、官方团队页和完整职位信息优先。不得把搜索摘要当成最终当前任职证明。

公司、产品、职位和人员任职属于易变事实。能联网时优先核验当次来源；不能访问当前来源时标记 `无法判断` 或相应 pending 状态，并写出缺失证据，不把历史资料表述为当前事实。

## 公司状态与门控（仅原始名单模式）

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

门槛未通过时，输出 `pending_reason`、`missing_evidence` 和 `next_action`，不要扩大职位词强行找人。本节不适用于用户已经确认可以找人的名单。

## 双角色规则

- `technical_user`：直接构建、集成、部署、维护或日常使用目标工作流的人。
- `technical_decision_maker`：管理该团队，或负责该产品/平台的架构、工程或产品路线，能影响技术与供应商选择的人。
- 1–20 人公司允许同一人标记为 `dual_role`，但必须同时有实施和决策证据。
- 大公司从技术使用者所在团队向上找负责人，不默认集团 CTO、CIO 或最高级别高管。
- 招聘人员、销售人员和岗位发布人只能作为 `routing_contact`，不能替代双角色。

若公司已经按 Company List 和目标团队分好，Apollo 首轮只使用 `Company List/Current Company + Job Titles`，关闭 Past Job Titles；Departments、Management Level、Location、Email Status 留空。技术使用者可开启相似职位，决策者默认关闭。结果过多时才逐项增加产品线、部门或管理层级。每类最多查看 10 人，保留最多 3 人进入核验。

Apollo 只负责召回与补充商业联系资料，不证明候选职责匹配。不要用浏览器自动翻页、批量读取或抓取 Apollo；优先用户手动导出，或使用 Apollo 明确提供且已授权的官方连接/API。任何可能消耗邮箱、电话、enrichment 或 export credits 的操作，未获授权时先停在预估或候选名单。

## 首轮未找到后的补搜

首轮没有结果时，先记录失败类型，不得直接把公司判为无人或只返回一句 `无法判断`：

- `no_candidates`：Apollo/公开搜索没有返回候选，通常是职位名称或数据覆盖问题。
- `no_role_fit`：有候选，但团队、任务或 persona 均不符合。
- `only_one_persona`：只找到技术使用者或技术决策者。
- `current_role_unverified`：人选看似相关，但无法确认当前任职、团队或职责。

按以下顺序补搜：

1. 第二轮职位扩展：移除 Email Status、Department、Location、Management Level 等非必要限制；每个 persona 增加 2–5 个相邻职位。技术使用者可开启相似职位；决策者使用功能词与 Manager、Director、Head、VP 等管理词组合，不直接跳到集团最高管理层。
2. 第三轮组织关系反查：已有技术使用者时向上找其团队负责人；已有决策者时向下找同团队工程师。再查官网团队页、工程博客、当前招聘、会议演讲或项目作者，并回到 LinkedIn/Apollo 核验现任。
3. 1–20 人公司可核验 Founder、CTO 或 Head of Engineering 是否满足 `dual_role`；必须同时具备实施与决策证据。
4. 招聘人员、岗位发布人或泛业务负责人只能标为 `routing_contact`，不能替代缺失 persona。

最多执行三轮搜索，整个流程每个 persona 最多核验 10 名候选。仍未找到时输出 `partial_buying_group` 或 `no_verified_contact`，并写明缺失 persona、失败原因、各轮改动和下一步。邮箱缺失不属于找人失败，继续按邮箱流程处理。

## 候选状态

- `selected`：当前公司、当前职位和职责均能解释，与目标团队和任务匹配。
- `pending_A`：基本匹配，只缺一次团队或职责核验。
- `pending_B`：只有相邻团队、旧资料或技能背景，不能直接触达。
- `rejected`：离职、公司错配、竞品范围、职责无关或身份冲突。

硬门槛优先于评分。离职人员即使历史项目高度相关，也必须标记 `rejected_departed`。

同一 persona 的排序按以下顺序决胜：

1. 当前职责直接承担目标任务。
2. 明确位于正确团队、产品线或内部平台。
3. persona 权限成立：使用者能实施/集成/维护/使用；决策者能管理团队或影响架构、选型、采购或交付。
4. 当前任职证据更新且来源质量更高。
5. 分数仍相同时，再比较职级与联系渠道可得性。

邮箱永远不能把职责较弱的人排到职责更直接的人前面。

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
- 原始名单模式完成资格、排竞、任务、工作流位置和目标团队判断；已确认可找人的名单不重复要求这些公司门槛。
- 每家公司至少有一名已核验技术使用者和一名已核验技术决策者，或在最多三轮补搜后明确标记 `partial_buying_group` / `no_verified_contact`。
- 所有 `selected` 人员都有当前任职和职责证据，已知离职人员为零。
- 每个 persona 只有一名 `best_candidate=true`；备选不得超过一名，除非用户要求扩大名单。
- 邮箱缺失不会改变人选状态；只有 `verified_valid` 计入邮件可触达。
- 输出能追溯每项结论的来源、日期、搜索轮次、失败原因和下一步。
