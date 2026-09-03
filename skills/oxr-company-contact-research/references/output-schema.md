# 输出、状态与评分

## 公司记录

```json
{
  "company_name": "Example Co",
  "company_domain": "example.com",
  "linkedin_company_url": "https://www.linkedin.com/company/example",
  "entity_scope": "company",
  "parent_company": "",
  "qualification_status": "qualified",
  "competitor_rule": "clear",
  "restricted_scope": "",
  "company_scenario": "internal_enterprise",
  "workflow_position": "internal_infrastructure",
  "primary_task": "retrieve_index",
  "secondary_tasks": ["ingest_chunk"],
  "target_team": "search_retrieval",
  "company_size": "1000+",
  "evidence_level": "A",
  "evidence_summary": "招聘职责明确建设企业搜索、RAG 与文档摄取管线。",
  "evidence_urls": ["https://example.com/evidence"],
  "people_search_ready": true,
  "pending_reason": "",
  "missing_evidence": [],
  "next_action": "",
  "last_reviewed_at": "2026-09-03"
}
```

## 人员记录

```json
{
  "company_domain": "example.com",
  "full_name": "Example Person",
  "linkedin_url": "https://www.linkedin.com/in/example",
  "persona": "technical_user",
  "current_company": "Example Co",
  "current_title": "Search Engineer",
  "current_team": "Enterprise Search",
  "matched_task": "retrieve_index",
  "matched_workflow_position": "internal_infrastructure",
  "matched_target_team": "search_retrieval",
  "score_total": 92,
  "score_breakdown": {
    "target_team": 30,
    "primary_task": 25,
    "workflow_position": 20,
    "persona": 12,
    "current_evidence_freshness": 3,
    "evidence_quality": 2
  },
  "verification_level": "V1",
  "person_status": "selected",
  "email_address": "",
  "email_status": "not_found",
  "outreach_channel": "linkedin",
  "evidence_urls": ["https://www.linkedin.com/in/example"],
  "verified_at": "2026-09-03",
  "rejection_reason": "",
  "next_action": "查公开业务邮箱或补同团队备选"
}
```

## 评分规则

硬门槛先于评分：

- 当前公司一致，否则 `rejected_company_mismatch`。
- 当前仍在职，否则 `rejected_departed`。
- 不在禁止竞品范围，否则 `rejected_competitor_scope`。
- 至少一条团队或职责证据，否则 `pending_responsibility`。
- persona 可以用职责解释，否则 `pending_persona`。

通过后按以下分数排序：

| 项目 | 分值 | 满分条件 |
|---|---:|---|
| `target_team` | 30 | 明确位于目标团队 |
| `primary_task` | 25 | 当前职责直接执行或负责目标任务 |
| `workflow_position` | 20 | 位于正确核心产品、指定产品线或内部平台 |
| `persona` | 15 | 决策或使用职责直接成立 |
| 当前证据新鲜度 | 5 | 两个当前来源一致 |
| 证据质量 | 5 | 官方团队页、当前 Experience 或完整 JD 直接支持 |

建议阈值：

- `80–100`：`selected`
- `60–79`：`pending_A`，补一次团队或职责核验
- `40–59`：`pending_B`
- `<40`：`rejected`

## verification_level

| 等级 | 条件 | 可用状态 |
|---|---|---|
| `V1` | 当前 Experience 与 Apollo Current Company/官方团队页一致，且职责直接匹配 | `selected` |
| `V2` | 一个当前任职来源明确，另有 JD/项目支持任务匹配 | `selected` 或 `pending_A` |
| `V3` | 只有摘要、旧文章、技能或相邻团队线索 | `pending_B` |
| `V0` | 新雇主、任期结束、同名人或实体冲突 | `rejected` |

## 批次汇总

至少输出：

```text
company_count_unique_domain
company_qualified_count
company_people_search_ready_count
company_dual_persona_complete_count
company_partial_buying_group_count
person_selected_count
person_departed_rejected_count
pending_by_stage
duplicate_domain_count
```

只有两类 persona 都有当前任职和职责证据时，才计入 `company_dual_persona_complete_count`。`pending`、`routing_contact` 和离职人员均不计入。

## 面向人工审核的表格

| 公司 | Domain | 场景 | Primary task | Workflow position | Target team | 技术使用者 | 技术决策者 | 当前核验 | 邮箱状态 | 证据 | 下一步 |
|---|---|---|---|---|---|---|---|---|---|---|---|

不要把 `pending` 候选伪装成最终名单。联系人缺失时保留空值并写明缺失阶段和下一步。
