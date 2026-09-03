# 公司精细分类规则

## 字段之间的逻辑

```text
company_scenario
  决定使用哪条业务分支
    ↓
primary_task
  决定公司当前最主要的文档/知识任务
    ↓
workflow_position
  决定该能力位于核心产品、嵌入功能还是内部基础设施
    ↓
target_team
  把业务判断转换成实际找人的组织边界
    ↓
company_size
  决定决策者层级以及是否允许 dual_role
```

`primary_document_type` 只补充邮件语境，不负责决定目标团队。没有可靠文档类型时使用 `unknown`，不得依据行业猜测。

## 第一步：公司主体标准化

至少确认：

- `company_name`：官方标准名称。
- `company_domain`：官网根域名，去掉 `www`、路径、查询参数和地区子域名。
- `linkedin_company_url`：对应当前实体的公司页。
- `entity_scope`：公司、品牌、产品线或子公司。
- `parent_company`：存在并购或母子关系时记录。

公司去重以根域名为主，以 LinkedIn Company URL 为辅助。只凭相似名称不得合并。同名公司、品牌页和被并购产品必须先解决实体边界。

## 第二步：快速资格判断

对每家公司回答四问，并为每问保存 `yes/no/unclear + 短证据`：

1. 公司主体是否真实且唯一？
2. 是否有具体产品、项目、客户案例或招聘职责涉及目标任务？
3. 它是否是潜在使用方或存在可触达团队？
4. 能否从证据推导目标任务和团队？

明确的相关信号包括 OCR、Document AI、IDP、文档解析/抽取/摄取、RAG、企业搜索、知识库、索引、合同/理赔/发票/KYC/档案处理、AI Agent 或流程自动化。

只有 `AI-powered`、`automation`、`data intelligence`、`digital transformation` 等泛词时保持 `pending_evidence`。

## 第三步：排竞

`company_scenario` 描述业务类型，`competitor_rule` 决定触达范围：

| competitor_rule | 含义 | 处理 |
|---|---|---|
| `clear` | 未发现直接竞争关系 | 可继续 |
| `product_team_only` | 集团内只有指定产品线构成竞品 | 排除该产品团队，其他明确团队可继续 |
| `company_wide` | 核心业务直接销售 OCR、IDP、文档解析等同类能力 | 公司级停止找人 |

不能因为公司属于 Document Workflow 就自动判定竞品，也不能因为是 Internal Enterprise 就自动判定安全。

## 第四步：五类 company_scenario

| 类型 | 判定 | 细分重点 | 找人起点 |
|---|---|---|---|
| `document_workflow` | 文档自动化、Document AI、数据提取或文档管理产品/服务 | 根据主要产品或招聘职责确定 `primary_task` | Document AI、Product Engineering、Automation |
| `search_knowledge` | 企业搜索、知识库、RAG、语义检索或知识平台 | 判断摄取、索引、检索链路中缺哪一环 | Search/Retrieval、Knowledge、Data Platform |
| `vertical_document` | 合同、法律、理赔、KYC、发票、专利、医疗等垂直文档软件 | 明确文档类型、业务结果和具体产品线 | 垂直产品的 Applied AI、Product、Solutions |
| `ai_workflow` | Agent、Copilot、工作流或自动化平台，文档/知识是输入 | 证明工作流实际处理文档或知识输入 | Applied AI、AI Platform、Integrations、Automation |
| `internal_enterprise` | 银行、制造、医药、高校等内部使用方 | 不设统一职位词；先确定知识库、摄取、解析、自动化或文控任务 | 对应内部平台或业务线，不默认集团 CTO |

如果一家公司同时符合多类，选择与本次证据和目标任务最直接的一类；其他类型记录为备注，不使用多选掩盖主路径。

## 第五步：primary_task

| primary_task | 判断问题 | 技术使用者先搜 | 决策者向上找 |
|---|---|---|---|
| `parse_extract` | 是否把 PDF、图片或文档转成字段、文本或结构化数据？ | Document AI、OCR、CV、NLP、ML、Applied Scientist | Document AI、Applied AI、ML Engineering 负责人 |
| `ingest_chunk` | 是否负责接收、清洗、切分、嵌入或管线化内容？ | Data Engineer、Ingestion、ETL、Pipeline、Platform Engineer | Data Platform、Data Engineering、AI Platform 负责人 |
| `retrieve_index` | 是否建立索引、检索、搜索、知识库或 RAG grounding？ | Search、Retrieval、Relevance、Knowledge、Vector Search Engineer | Search、Knowledge、Retrieval、Enterprise AI 负责人 |
| `classify_understand` | 是否分类、识别、总结、推理、风险识别或理解内容？ | Applied Scientist、NLP、ML、Knowledge Engineer | Applied Research、ML、AI Product 负责人 |
| `workflow_automation` | 是否让文档结果触发审批、路由、录入、Agent 或系统动作？ | Automation、RPA、Integration、AEM、Applied AI Engineer | Intelligent Automation、Enterprise AI、Product/Platform 负责人 |

只选择最主要任务。若一个案例横跨多步，以购买痛点或招聘职责的主要交付结果决定主任务，其余记录为 `secondary_tasks`。

## 第六步：workflow_position

| 值 | 判定 | 优先人员 | 不优先人员 |
|---|---|---|---|
| `core_product` | 文档、搜索或自动化能力本身就是公司核心产品 | 核心产品工程/产品负责人和直接实现者 | 无关集团 IT 或通用顾问 |
| `embedded_feature` | 能力嵌入某个更大的业务产品或解决方案 | 具体产品线的 Applied AI、ML、Search、Integration 负责人和实现者 | 母公司所有 AI 员工、无法证明产品归属的人 |
| `internal_infrastructure` | 能力服务公司内部业务、员工或运营系统 | Enterprise AI、Data Platform、Knowledge Management、Intelligent Automation、Document Control | 集团最高 CTO、纯业务高管、无目标职责的通用开发 |

单一产品小公司可从产品证据直接推导该字段；大型、多产品和 Internal Enterprise 公司必须明确记录。

## 第七步：target_team

| target_team | 常见任务 |
|---|---|
| `document_ai` | OCR、版面分析、字段抽取、文档理解 |
| `search_retrieval` | 企业搜索、索引、relevance、RAG 检索 |
| `data_platform` | 摄取、ETL、chunking、embedding、数据管线 |
| `applied_ai` | NLP、ML、CV、生成式 AI 在具体产品中落地 |
| `enterprise_ai` | 企业内部 AI 平台、共享模型或治理能力 |
| `knowledge_management` | 知识库、内容管理、数字资产、档案与发现 |
| `intelligent_automation` | RPA、Agent、审批、路由、系统集成 |
| `quality_document_control` | 制造/医药等内部文控、SOP、质量文件与合规流程 |

`target_team` 必须能被证据解释。找不到正式团队名时，可使用上述功能团队作为检索边界，但应标记 `team_name_unverified`。

## 第八步：company_size

| 规模 | 决策者优先级 |
|---|---|
| `0-20` | Founder/CTO/Founding Engineer 可为 dual_role |
| `20-99` | Head/Lead/Engineering Manager/Product Lead |
| `100-499` | Director/Head/VP，配同一产品或工程团队 IC |
| `500-1000` | 明确业务线或平台的 Director/VP，不找泛集团高管 |
| `1000+` | 先锁定产品、区域或内部平台，再沿团队找负责人 |

## 找人门槛

```text
people_search_ready =
  domain_confirmed
  AND qualification_status == qualified
  AND competitor_rule != company_wide
  AND company_scenario is not null
  AND workflow_position is not null
  AND primary_task is not null
  AND target_team is not null
  AND evidence_level in [A, B]
  AND (competitor_rule != product_team_only OR entity_scope is explicit)
```
