# 输出、状态与评分

## 默认最小输出

零上下文或用户只问“谁最合适”时，不先展示完整状态表。每家公司、每个 persona 最多一名最佳人选：

| 公司 | 最佳人选 | Persona | 当前职位 | 结论 | 为什么匹配 | 当前证据 | 缺口/下一步 |
|---|---|---|---|---|---|---|---|

`结论` 只使用：

- `符合`：通过当前公司、团队/产品线、职责和 persona 硬门槛。
- `不符合`：明确离职、实体/团队/职责不符或命中竞品限制。
- `无法判断`：来源打不开、只有旧资料或缺少当前团队/职责证据。

若输入是 Apollo/CSV 候选并需要回传文件，最小列为：

```text
company_name,candidate_name,persona,current_title,apollo_url,linkedin_url,match_result,reason,evidence_url
```

详细 JSON 和评分用于系统对接、审计或用户明确要求时。

没有找到某个 persona 时，仍在最小表中保留该公司和 persona，`最佳人选` 留空；`缺口/下一步` 必须写明已尝试的搜索轮次、失败原因和下一步，不能只写 `无法判断`。

## 找人覆盖与补搜状态

以下字段默认内部保留；仅在用户要求审计、补搜明细或系统对接时展开：

```json
{
  "company_domain": "example.com",
  "search_round": 3,
  "coverage_status": "complete|partial_buying_group|no_verified_contact",
  "missing_persona": ["technical_user"],
  "failure_reason": "no_candidates|no_role_fit|only_one_persona|current_role_unverified",
  "search_changes": [
    "第二轮增加相邻职位并移除非必要筛选",
    "第三轮从已确认决策者向下反查同团队工程师"
  ],
  "next_action": "核验目标团队当前工程人员"
}
```

- `complete`：两个 persona 均有通过硬门槛的最佳人选。
- `partial_buying_group`：只有一个 persona 通过硬门槛。
- `no_verified_contact`：三轮后仍没有人通过当前任职、团队、职责和 persona 硬门槛。
- `email_not_found` 只写入 `email_status`，不得作为 `failure_reason`。

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
  "best_candidate": true,
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

分数只在硬门槛通过后用于同一 persona 内排序。`best_candidate=true` 只能给得分最高且职责证据最直接的一人；邮箱状态不计入分数。备选默认最多一名，并标 `best_candidate=false`。

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
input_company_count
company_count_unique_domain
company_dual_persona_complete_count
company_partial_buying_group_count
person_selected_count
person_departed_rejected_count
pending_by_stage
duplicate_domain_count
search_round_counts
failure_reason_counts
```

只有原始名单模式额外输出 `company_qualified_count` 和 `company_people_search_ready_count`。已确认可找人的名单不要求这两个公司门槛统计。

只有两类 persona 都有当前任职和职责证据时，才计入 `company_dual_persona_complete_count`。`pending`、`routing_contact` 和离职人员均不计入。

## 面向人工审核的表格

| 公司 | Domain | 场景 | Primary task | Workflow position | Target team | 技术使用者 | 技术决策者 | 当前核验 | 邮箱状态 | 证据 | 下一步 |
|---|---|---|---|---|---|---|---|---|---|---|---|

不要把 `pending` 候选伪装成最终名单。联系人缺失时保留空值，写明搜索轮次、失败原因、已尝试改动和下一步。
