# yunyun-hub

一组可复用的 Codex Skills。每个 Skill 都把特定任务的判断标准、执行流程和输出格式放进独立目录，安装后可以直接在 Codex 中调用。

## Skills

| Skill | 用途 | 目录 |
|---|---|---|
| `oxr-company-contact-research` | 从已确认可找人的公司名单出发，寻找并核验技术使用者与技术决策者；首轮缺人时自动扩大职位、反查组织关系，最多执行三轮搜索 | [`skills/oxr-company-contact-research`](skills/oxr-company-contact-research) |
| `somark-xiaohongshu-image-prompts` | 为 SoMark 小红书封面和轮播图生成构图思路、生图提示词及负面提示词，默认遵循黑橙视觉规范 | [`skills/somark-xiaohongshu-image-prompts`](skills/somark-xiaohongshu-image-prompts) |

## 安装

### 让 Codex 安装

把目标 Skill 的 GitHub 链接发给 Codex，例如：

```text
请使用 $skill-installer 安装这个 Skill：
https://github.com/yiruiyuan3-cmd/yunyun-hub/tree/main/skills/oxr-company-contact-research
```

### 手动安装

```bash
git clone https://github.com/yiruiyuan3-cmd/yunyun-hub.git
mkdir -p "$HOME/.agents/skills"
cp -R yunyun-hub/skills/oxr-company-contact-research "$HOME/.agents/skills/"
```

Codex 通常会自动发现新安装的 Skill；如果没有出现，重启 Codex。

## 使用示例

### 找目标联系人

```text
使用 $oxr-company-contact-research，帮我在以下公司中寻找最合适的
technical_user 和 technical_decision_maker：

- Example Company
- Another Company
```

这份 Skill 默认把用户提供的公司名单视为已经完成筛选和排竞，直接进入找人流程。只有用户明确说明名单未经筛选，或要求重新判断公司时，才执行公司资格判断。

每家公司默认交付两个角色：

- `technical_user`：直接构建、集成、部署、维护或使用目标工作流的人。
- `technical_decision_maker`：管理同一团队，或影响架构、技术选型、采购和交付的人。

若首轮没有合格人选，Skill 会依次扩大职位范围、从已有联系人反查上下级和同团队成员，并核验当前任职与职责。最多三轮后仍有缺口时，会保留公司和缺失角色，说明失败原因、已尝试方法与下一步，不会用低质量候选凑数。

邮箱状态与人选匹配度分开判断。没有邮箱不会让职责更弱的人排到更合适的人前面。

### 生成小红书配图提示词

```text
使用 $somark-xiaohongshu-image-prompts，根据这段文案设计一张 3:4 的小红书封面。
```

默认只输出构图思路、完整生图提示词和负面提示词。只有明确要求直接生成图片时，才调用生图工具。

## 仓库结构

```text
yunyun-hub/
├── README.md
└── skills/
    ├── oxr-company-contact-research/
    │   ├── SKILL.md
    │   ├── agents/
    │   └── references/
    └── somark-xiaohongshu-image-prompts/
        ├── SKILL.md
        ├── agents/
        └── references/
```

每个 Skill 的 `SKILL.md` 是入口文件，`references/` 保存按需加载的规则与模板，`agents/openai.yaml` 保存界面元数据。

更多格式与安装说明可参考 [OpenAI 官方 Skills 文档](https://developers.openai.com/codex/skills)。
