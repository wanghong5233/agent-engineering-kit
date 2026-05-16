# 如何写一份让 Agent 真正会用的 Skill

> 目标读者：长期做 Agent / LLM 应用工程的人。  
> 写作目标：复习可记、面试可讲、项目可复用。

## 一句话结论

**Skill 不是“更长的 prompt”，而是按需加载的工程流程包。写 skill 的核心不是多写规则，而是控制触发、体量、边界和数量。**

## 1. 心智模型

```text
Agent Skill = description 触发器 + SKILL.md 流程主干 + references/scripts/assets 按需资料

成本模型：
L1 metadata     永驻       name + description       常驻税
L2 SKILL.md     触发后驻留 流程主干 + 关键约束       触发税
L3 references   按需读取   长文/案例/脚本/模板        按需读
```

第一性原理：**每一行都应该放在它最低成本、最高命中的层级。**

## 2. 业界事实

| 判断 | 证据 |
|---|---|
| Agent Skills 是跨平台形态，不只属于 Cursor | Anthropic 2025-10-16 发布 Skills；2025-12-18 推出开放标准 [agentskills.io](https://agentskills.io) |
| description 是触发命门 | Cursor / Anthropic 都通过 description 判断何时加载 skill |
| spec 上限 ≠ 实践甜区 | `description` spec 硬阈值 1024 字符；实践甜区 <300 字符 |
| 未治理 skill 容易不触发 | 社区实测未经治理触发率约 50% |
| 未触发 skill 也有成本 | description 常驻可发现列表，形成 invisible context tax |
| inline 规则有时比 skill 更稳定 | 永久编码风格应进 `AGENTS.md` / `CLAUDE.md` / Cursor Rule，而不是 skill |

## 3. 最小结构

```text
skill-name/
├── SKILL.md              # 必填：frontmatter + 流程主干
├── references/           # 可选：长文、案例、设计依据
├── scripts/              # 可选：稳定脚本
└── assets/               # 可选：模板 / 数据
```

```yaml
---
name: skill-name
description: <what + when + trigger keywords + optional negative triggers>
---
```

| 字段 | 规则 |
|---|---|
| `name` | 1-64 字符，kebab-case，等于父目录名 |
| `description` | 1-1024 字符，单行 YAML，实践 <300 |
| body | <500 行；高质量 skill 通常 <150 行 |
| references | 大块案例、依据、checklist 下沉 |

## 4. 写 description

公式：

```text
<What it does>. Use when <specific triggers>. Do NOT use when <negative boundary>.
```

反例：

```yaml
description: Helps with README files.
```

正例：

```yaml
description: Write or refactor top-level README.md / README_EN.md. Use when user asks to 写/同步/修改/更新 README, or complains README has 口水话/啰嗦/不像专业 GitHub 项目. Do NOT use for sub-directory READMEs.
```

中英混写是否正常？

| 场景 | 判断 |
|---|---|
| description 触发词 | 正常，甚至必要。真实用户会中英混说：README、bug、架构、沉淀、playbook |
| 正文 instruction | 少混。除标准术语外尽量用一种语言，降低阅读成本 |
| 文件名 / 协议 / API | 保留英文，因为它们是稳定符号 |

## 5. 写 body

| 放 body | 下沉 references |
|---|---|
| 一句话准则 | 业界长文依据 |
| 主流程 | 多个完整案例 |
| 关键禁止项 | 历史讨论 |
| 1 个反例 + 1 个正例 | 详细 checklist |
| 输出格式 | 设计 rationale |

体量信号灯：

| 指标 | 优 | 警告 |
|---|---:|---:|
| body 行数 | <150 | ≥500 |
| 顶级 sections | 3-8 | ≥9 |
| 代码块 | ≤3 | ≥4 |
| `MUST/NEVER` | ≤5 | ≥6 |
| description | <300 | ≥300 |

## 6. 数量治理

Skill pack 不是越多越好。每个 description 都会进入可发现能力列表；太多会降低触发准确率、稀释注意力、增加 token 成本。

```text
open-source awesome list     用来检索，不常驻
        ↓
upstream benchmark           看结构、trigger、边界
        ↓
local rewrite                按自己的工程问题改写
        ↓
active skill pack            控制数量，可测触发
```

| 作用域 | 数量 | 原则 |
|---|---:|---|
| 项目级 `.cursor/skills/` | 5-8 | 只放项目强相关流程 |
| 个人级 `~/.cursor/skills/` / `~/.agents/skills/` | 8-12 | 放跨项目高频工程能力 |
| 总 active skills | ≤15 | 超过就复审合并 / 禁用 |
| 30+ | 不常驻 | 当 marketplace，而不是工作集 |

新增 skill 前问：能否并入已有 skill？能否只是 `references/`？是否只是开源 skill 的低差异复制？

## 7. 不要闭门造车

正确策略：**高 star 仓库做 benchmark，本地 skill 做改写。**

| 来源 | 用法 |
|---|---|
| [anthropics/skills](https://github.com/anthropics/skills) | 官方规范、`skill-creator`、`mcp-builder` |
| [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | 生产工程流程：debugging、API design、security、testing |
| [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) | LLM cost、prompt governance、SLO、feature flags |
| [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) | 索引和灵感，不整包安装 |

| 维度 | 采用 | 不采用 |
|---|---|---|
| 触发 | 有清晰 what/when/negative | “helps with X” |
| 内容 | 有流程、边界、输出格式 | 泛泛最佳实践 |
| 体量 | 主体短，细节下沉 | 几百行全塞 body |
| 差异 | 能解决你的真实高频问题 | 只是看起来专业 |
| 组合 | 不和现有 skill 冲突 | 与已有 skill 职责重叠 |

## 8. 生产级 Agent Skill Pack

| Skill | 解决的问题 |
|---|---|
| `agent-debugging` | 429、timeout、RAG 漂移、tool 失败、异步 job 失败的根因定位 |
| `llm-observability-and-evals` | trace、日志、指标、golden set、质量回归 |
| `tool-and-mcp-design` | tool schema、权限、错误语义、幂等、MCP contract |
| `llm-cost-optimizer` | token、模型路由、缓存、输出上限、成本归因 |
| `prompt-governance` | prompt 版本、registry、A/B、eval gate |
| `agent-security-and-sandboxing` | prompt injection、secret、文件/网络权限 |
| `feature-flags-and-rollout` | 灰度、kill switch、回滚 |
| `webapp-agent-testing` | Agent 产品端到端验证 |

当前实践：先落地前 4 个，避免一次性把 skill pack 做成 marketplace。

## 9. Walkthrough

```text
Step 1  选名字：writing-readme
Step 2  写 description：what + when + trigger + negative
Step 3  body 只写流程：定位、结构、反例、输出格式
Step 4  例子和依据下沉 references/
Step 5  用 3 个真实 prompt 测触发
```

触发测试：

```text
帮我写 README
这个 README 太口水了，像不专业 GitHub 项目
同步 README_EN 和中文 README
```

全触发才算合格；不触发先修 description，不要先加 body。

## 10. 自检清单

- [ ] 这个任务是否反复出现、且有完整流程？
- [ ] 是否不该写进 `AGENTS.md` / `CLAUDE.md` / Cursor Rule？
- [ ] 是否查过开源 benchmark，而不是闭门造车？
- [ ] description 是否单行、<300 字符、含中英触发词？
- [ ] body 是否 <150 行，且只保留执行主干？
- [ ] references 是否承载长案例、依据、checklist？
- [ ] 是否准备了 3 条真实 prompt 测触发？
- [ ] active skill 总量是否仍 ≤15？

## 11. 可讲给面试官的 5 句话

1. Skill 的本质是按需加载的流程包，不是更长的 prompt。
2. description 是触发器；body 是触发后成本；references 是按需资料。
3. 永久约束进 Rule / AGENTS.md，重复流程才进 skill。
4. skill pack 要控量，太多会产生 invisible context tax。
5. 高 star skill 不该整包复制，应作为 upstream benchmark 后本地改写。

## 12. 延伸阅读

- Anthropic: [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- Open standard: [agentskills.io](https://agentskills.io)
- Cursor Docs: [Agent Skills](https://cursor.com/docs/skills)
- Community critique: [Most Claude Code Skills Are Useless](https://codn.dev/blog/most-claude-code-skills-are-useless/)
- Trigger study: [Why Claude Code Skills Don't Trigger in 2026](https://dev.to/lizechengnet/why-claude-code-skills-dont-trigger-and-how-to-fix-them-in-2026-o7h)
- OpenAI Codex issue: [description >1024 chars loading failure](https://github.com/openai/codex/issues/13941)
