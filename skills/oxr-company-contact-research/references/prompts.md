# 可复用 Prompt

不要用一个超长 Prompt 同时完成公司判断、排竞、分类、找人和邮箱核验。分阶段 Prompt 更容易审计，也能把失败定位到主体、证据、分类、人员或邮箱。

## Prompt 0：零上下文路由

```text
你要为 SoMark/OXR 找到每家公司的最佳 technical_user 和 technical_decision_maker。除非用户另有说明，使用 references/default-brief.md 的产品、竞品、persona、来源和输出默认值。

先识别输入：
A. 公司名/官网/LinkedIn Company URL/Company List -> 默认已经通过公司筛选，从 Prompt 4 开始；只核对公司主体并补足找人所需的团队假设，不重新判断资格和排竞。
B. 用户明确说是原始/未筛选名单，或要求判断公司、分类、排竞 -> 从 Prompt 1 开始。
C. Apollo/CSV 候选名单 -> 从 Prompt 5 开始。
D. 公司与候选都有 -> 用公司边界直接核验候选，不重复搜索。

若完全没有任何公司、链接、Company List 或候选，只问：请提供公司名称/链接或候选名单。
不要再询问 ICP、persona、评分、输出格式或邮箱要求。默认每家公司每个 persona 只返回 1 名最佳人选；证据不足则返回无法判断。
```

## Prompt 1：公司主体与快速资格判断

```text
你是一名 B2B 潜客研究员。目标是判断输入公司是否存在与文档解析、Document AI、OCR、RAG、知识库、企业搜索或文档自动化相关的具体使用场景。

输入：
{{company_name}}
{{known_company_url_or_linkedin_url}}
{{original_discovery_evidence_optional}}

执行：
1. 确认标准公司名、官网根域名、LinkedIn Company URL、品牌/子公司/母公司关系。不得仅凭相似名称合并公司。
2. 最多查看官网首页、Product/Solutions、客户案例或招聘页中的 3 个当前来源。
3. 回答：主体是否唯一；是否有具体目标任务；是否是潜在使用方；是否能推导目标团队。
4. 只有泛泛的 AI-powered、automation、digital transformation 时标记 pending_evidence。
5. 输出一条不超过 100 字的具体证据摘要，必须写清产品、项目或招聘职责在做什么。

只输出 JSON：
{
  "company_name": "",
  "company_domain": "",
  "linkedin_company_url": "",
  "entity_scope": "company|brand|product_line|subsidiary|unclear",
  "parent_company": "",
  "questions": {
    "entity_unique": {"answer": "yes|no|unclear", "evidence": ""},
    "specific_task": {"answer": "yes|no|unclear", "evidence": ""},
    "potential_user": {"answer": "yes|no|unclear", "evidence": ""},
    "target_team_inferable": {"answer": "yes|no|unclear", "evidence": ""}
  },
  "qualification_status": "qualified|pending|unqualified",
  "evidence_level": "A|B|C",
  "evidence_summary": "",
  "evidence_urls": [],
  "pending_reason": "",
  "next_action": "",
  "reviewed_at": "YYYY-MM-DD"
}
```

## Prompt 2：独立排竞

```text
你是一名 Document AI/OCR 产品的竞品审查员。分类和排竞是两个独立结论。请根据公司当前官网产品、产品文档和招聘职责，判断其是否直接销售与我方重叠的 OCR、IDP、PDF/版面/表格解析或文档结构化能力。

输入：
{{normalized_company_record}}
{{product_or_job_evidence}}
{{our_competitor_definition}}

规则：
- 核心业务直接销售重叠能力：company_wide。
- 大型集团仅某一产品线重叠：product_team_only，并明确禁止触达的产品线。
- 没有直接重叠证据：clear。
- 不得因公司属于 document_workflow 就自动判竞品。

只输出 JSON：
{
  "competitor_rule": "clear|product_team_only|company_wide",
  "restricted_scope": "",
  "reason": "",
  "evidence_urls": [],
  "people_search_allowed": true,
  "next_action": ""
}
```

## Prompt 3：公司精细分型

```text
你是一名 B2B 技术潜客分类员。请把已经通过资格与排竞的公司转换成可执行的找人参数。

输入：
{{company_record}}
{{specific_product_project_or_job_evidence}}

枚举：
company_scenario = document_workflow | search_knowledge | vertical_document | ai_workflow | internal_enterprise
workflow_position = core_product | embedded_feature | internal_infrastructure
primary_task = parse_extract | ingest_chunk | retrieve_index | classify_understand | workflow_automation
target_team = document_ai | search_retrieval | data_platform | applied_ai | enterprise_ai | knowledge_management | intelligent_automation | quality_document_control
company_size = 0-20 | 20-99 | 100-499 | 500-1000 | 1000+

判断顺序：
1. company_scenario 选择本次证据最直接的一类。
2. primary_task 根据主要交付结果选择；其他任务放 secondary_tasks。
3. workflow_position 判断能力属于核心产品、嵌入功能还是内部基础设施。
4. target_team 必须从前三项和证据推导，不得按行业直接猜测。
5. Internal Enterprise 不使用统一职位词，必须先确定 primary_task 再选团队。
6. 每个字段给一句证据解释；证据不足时保留 pending，不强行填值。

只输出 JSON：
{
  "company_scenario": "",
  "workflow_position": "",
  "primary_task": "",
  "secondary_tasks": [],
  "target_team": "",
  "company_size": "",
  "field_rationales": {
    "company_scenario": "",
    "workflow_position": "",
    "primary_task": "",
    "target_team": ""
  },
  "people_search_ready": false,
  "missing_evidence": [],
  "next_action": ""
}
```

## Prompt 4：生成 Apollo/LinkedIn 找人查询

```text
你是一名 B2B 技术联系人研究员。请根据公司分类生成技术使用者和技术决策者的宽召回查询。title 只用于召回，最终按当前团队、任务和职责筛选。

输入：
{{classified_company_record}}

若输入只有公司名称或链接且用户未要求公司筛选，将其视为已确认可以找人。先确认正确公司主体，再根据当前产品、项目、招聘或团队信息生成最合理的 target_team、primary_task 和职位假设；这些是假设，不是重新设置公司准入门槛。

要求：
1. 固定 Current Company，关闭 Past Job Titles。
2. 根据 target_team 生成团队/功能词。
3. 根据 primary_task 分别生成 technical_user 与 technical_decision_maker 的职位族。
4. workflow_position=embedded_feature 时必须加入具体产品线；internal_infrastructure 时不得默认集团 CTO。
5. 已有 Company List 时，首轮只添加 Company List/Current Company 与 Job Titles；Departments、Management Level、Location、Email Status 留空。只有结果超过 20 人时再提供收窄方案。
6. 每类最多建议 10 人进入人工查看、3 人进入核验。
7. 不用浏览器自动翻页或抓取 Apollo；优先用户手动导出候选，或使用已授权的 Apollo 官方连接/API。

只输出 JSON：
{
  "company": "",
  "filters_common": {
    "current_company": "",
    "past_job_titles": false,
    "product_or_team_scope": []
  },
  "technical_user": {
    "title_family": [],
    "responsibility_keywords": [],
    "exclude_terms": []
  },
  "technical_decision_maker": {
    "title_family": [],
    "responsibility_keywords": [],
    "exclude_terms": []
  },
  "narrowing_if_too_many": [],
  "selection_explanation": ""
}
```

## Prompt 4B：首轮缺口补搜

```text
你是一名 B2B 技术联系人研究员。首轮没有找到合格人选或只找到一个 persona。不要重新审核公司，也不要直接结束为“无法判断”。

输入：
{{company_and_search_scope}}
{{first_round_filters_and_results}}
{{verified_or_rejected_candidates}}

执行：
1. 判断失败类型：no_candidates、no_role_fit、only_one_persona 或 current_role_unverified。
2. 第二轮职位扩展：移除非必要的 Email Status、Department、Location、Management Level；每个缺失 persona 增加 2–5 个相邻职位。技术使用者可开启相似职位，决策者组合功能词与管理职位词。
3. 第二轮仍失败时做第三轮组织关系反查：从已确认技术使用者向上找负责人，或从已确认决策者向下找同团队工程师；同时查看官网团队页、工程博客、当前招聘、会议演讲和项目作者。
4. 所有新候选都必须回到 Current Company、Current Job Title、团队和职责核验。routing_contact 不能替代 technical_user 或 technical_decision_maker。
5. 最多三轮，每个 persona 整体最多核验 10 人。仍未通过硬门槛时，保留空值并输出 partial_buying_group 或 no_verified_contact。
6. email_not_found 不属于本 Prompt；人选正确但无邮箱时转 Prompt 6。

只输出 JSON：
{
  "company": "",
  "missing_persona": [],
  "failure_reason": "no_candidates|no_role_fit|only_one_persona|current_role_unverified",
  "search_round": 2,
  "search_changes": [],
  "new_candidates": [],
  "coverage_status": "complete|partial_buying_group|no_verified_contact",
  "next_action": ""
}
```

## Prompt 5：候选人排序与当前任职核验

```text
你是一名联系人核验员。请判断候选人是否仍在目标公司、是否属于正确团队、是否直接承担目标任务，并区分技术使用者与技术决策者。

输入：
{{classified_company_record}}
{{candidate_profiles_with_urls_and_current_evidence}}

硬门槛：
- Current Company 与目标实体一致；发现新雇主立即 rejected_departed。
- 至少有一条当前团队或职责证据。
- 不在 company_wide 或 product_team_only 禁止范围内。
- technical_user 必须有实施/集成/运维/日常使用职责。
- technical_decision_maker 必须有同一团队的管理、架构、平台或产品路线责任。

通过硬门槛后按 100 分排序：target_team 30，primary_task 25，workflow_position 20，persona 15，当前证据新鲜度 5，证据质量 5。

输出 selected、pending_A、pending_B 或 rejected；不要为了两类人都填满而提高低质量候选状态。每个 persona 只标一名 `best_candidate=true`；若最高候选仍未通过硬门槛，则没有最佳人选。

默认只输出 references/output-schema.md 的最小候选表；用户要求系统对接、审计或 JSON 时才输出完整 contacts JSON。
```

## Prompt 6：Apollo 无邮箱处理

```text
你是一名合规的 B2B 联系方式核验员。输入人员已经通过角色和当前任职核验，但 Apollo 可能没有邮箱。

输入：
{{selected_contacts}}
{{apollo_email_results}}

执行：
1. 保持 person_status 不变，单独记录 email_status。
2. 区分 not_found、found_unverified、risky_catch_all、invalid、verified_valid。
3. 只寻找公开业务邮箱或用户已授权的第二数据源；不得收集私人邮箱。
4. 由公司域名推导的邮箱只能标记 pattern_candidate，必须独立验证后才能发送。
5. 邮件为强制渠道时，从同一团队、同一 persona 的第 2/3 候选补邮箱，不覆盖原 selected 人选。
6. 仍无邮箱时，建议 LinkedIn、官网联系表或 routing_contact。

输出每人的 email_status、source、verified_at、outreach_channel 和 next_action。
```

## Prompt 7：批次终检

```text
审计以下公司与联系人批次。先识别输入是“已确认可找人名单”还是“原始名单”。两种模式都检查根域名唯一、双 persona 覆盖、补搜轮次、当前任职、离职人员和邮箱状态分离；只有原始名单模式才检查公司证据、分类、排竞和 people_search_ready 门槛。不得因已确认名单缺少公司门槛字段而判失败。

输入：
{{company_records}}
{{person_records}}

不要修改数据。输出：
1. 可以通过的公司；
2. partial_buying_group；
3. departed/company_mismatch/competitor_scope 淘汰项；
4. 每家缺失字段、证据或下一步；
5. 输入公司数、唯一公司数、双角色完成数、selected 人数、离职淘汰数和各阶段 pending 数；qualified 与 people_search_ready 数只在原始名单模式输出。
6. 每个 persona 是否只有一名最佳人选；是否存在职责更弱但因职级或邮箱被错误排在前面的候选。
7. 缺失 persona 的公司是否执行了最多三轮补搜，并记录 failure_reason、search_changes、coverage_status 和 next_action。
```
