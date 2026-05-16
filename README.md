# Agent Engineering Kit

> A reusable, production-grade `.cursor/` engineering package for Cursor / Claude Code / Agent IDEs.  
> Stop writing prompts. Start shipping engineering invariants.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Cursor](https://img.shields.io/badge/Cursor-compatible-1f6feb)](https://cursor.com/docs/rules)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-8a2be2)](https://code.claude.com)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-spec-0aa)](https://agentskills.io)

[English](./README.md) · 中文（往下看）

---

## TL;DR

When you start a new Agent / LLM project, your `.cursor/` folder usually devolves into a prompt junk drawer. Rules drift, skills never trigger, hooks fail open, and the next project copies the mess again.

**This kit is the opposite.** It is a 4-layer governance package — `rules` / `skills` / `commands` / `hooks` — extracted from a real production Agent project and stripped of all project-specific context. Drop it into a new repo, run the bootstrap skill, and you get a serious engineering baseline on day one.

```text
Rule    = durable defaults     (small, hard, always-on)
Skill   = reusable workflows   (loaded on demand)
Command = manual slash flows   (review / retro / triage)
Hook    = deterministic guards (failClosed, no model trust)
```

## Why this exists

Most public "Cursor rules collections" are large, opinionated prompt dumps. They:

- pollute your context window
- mix project-specific architecture into supposedly reusable rules
- treat safety as a soft suggestion, not a deterministic gate
- never explain why a rule exists, so it dies on the next refactor

This kit takes the opposite stance, derived from one principle:

> **Every constraint should live at the lowest-cost, highest-determinism layer that can enforce it.**

That principle drives every file in this repo.

## Features

- 🧠 **4-layer governance**: clear responsibility boundaries between rules, skills, commands, and hooks
- 📦 **Production-extracted**: every artifact survived a real Agent project, not invented from scratch
- 🔒 **Deterministic safety**: a stdlib-only Python hook with `failClosed: true` blocks dangerous git/shell commands
- 🚦 **Bootstrap workflow**: a dedicated skill walks you through initializing `.cursor/` correctly for any project stage (pre-MVP / MVP / stable)
- 🧹 **Project-agnostic by construction**: zero leaked product names, env prefixes, or vendor-specific identifiers
- 📚 **Methodology, not just files**: two long-form articles explain *how* to write skills and rules/hooks/commands so you can extend the kit yourself
- 🎯 **Skill discipline**: every SKILL.md ≤ 130 lines, every description ≤ 300 chars, follows the [agentskills.io](https://agentskills.io) spec

## What's inside

```text
agent-engineering-kit/
├── cursor/
│   └── .cursor/
│       ├── rules/
│       │   ├── core-principles.mdc           # always-on, ≤30 lines
│       │   ├── cursor-package-boundaries.mdc # keep .cursor/ from sprawling
│       │   ├── configuration-management.mdc  # no magic numbers, centralized config
│       │   └── agent-runtime-contracts.mdc   # LLM/RAG/tool/async durability contracts
│       ├── skills/
│       │   ├── bootstrap-cursor-package/     # initialize .cursor/ for a new project
│       │   ├── agent-debugging/              # systematic Agent failure triage
│       │   ├── llm-observability-and-evals/  # trace + eval contract before launch
│       │   ├── tool-and-mcp-design/          # production tool & MCP server design
│       │   ├── llm-cost-optimizer/           # cost auditing & guardrails
│       │   ├── writing-skill/                # meta-skill for authoring SKILL.md
│       │   ├── writing-tech-article/         # dense engineering writeups
│       │   ├── writing-readme/               # standard-readme spec + 10s rule
│       │   ├── writing-architecture-docs/    # current-state + first-principles form
│       │   ├── writing-issue-backlog/        # evidence-before-solution tracking
│       │   ├── writing-pitfall-archive/      # SRE blameless multi-incident form
│       │   └── writing-engineering-playbook/ # cross-project engineering intuition
│       ├── commands/
│       │   ├── review.md                     # critical / important / style audit
│       │   └── retro.md                      # session retrospective → kit improvements
│       ├── hooks.json                        # preToolUse + Shell + failClosed
│       └── hooks/
│           └── block-dangerous-shell.py      # stdlib-only, cross-platform
├── docs/
│   ├── how-to-write-agent-skills.md
│   └── how-to-write-agent-rules-hooks-commands.md
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## Quick start

### Option A — drop into an existing repo

```bash
# from your project root
cp -r path/to/agent-engineering-kit/cursor/.cursor ./.cursor
```

Then open the project in Cursor. The `core-principles` rule loads always-on; everything else loads on demand.

### Option B — initialize a brand-new project properly

1. Copy `cursor/.cursor/` into your new repo as `.cursor/`.
2. Open the project in Cursor.
3. Ask the agent: `初始化 .cursor 工程包` (or `bootstrap the cursor package`). The `bootstrap-cursor-package` skill triggers and walks you through the [init checklist](./cursor/.cursor/skills/bootstrap-cursor-package/references/init-checklist.md).
4. **Do NOT add a project architecture rule yet.** Wait until you have ≥3 stable directories or services. Empty architecture rules are worse than no rules.

### Verify the safety hook

```bash
# allow case
echo '{"tool_name":"Shell","tool_input":{"command":"git status"}}' | \
  python cursor/.cursor/hooks/block-dangerous-shell.py
# -> {"permission":"allow"}

# deny case
echo '{"tool_name":"Shell","tool_input":{"command":"git reset --hard HEAD"}}' | \
  python cursor/.cursor/hooks/block-dangerous-shell.py
# -> {"permission":"deny", ...}
```

## Design principles

### 1. Lowest-cost, highest-determinism placement

| Layer | Cost | Determinism | Use for |
|---|---|---|---|
| Rule | medium (always or scope-loaded) | medium | durable defaults, code conventions |
| Skill | medium (loaded on trigger) | medium | complex multi-step workflows |
| Command | low (manual) | medium | review / retro / release / triage |
| Hook | low (event-triggered) | high | safety, secrets, formatting, audit |

If a constraint can be enforced by a hook, never write it as a rule. If a workflow is manual, never disguise it as always-on context.

### 2. Sweet spots, not hard limits

| Layer | Sweet spot | Hard warning |
|---|---:|---|
| Always-on rules | 1–3 files, 20–50 lines | >100 always-on lines becomes context tax |
| File-scoped rules | 5–10 focused files | broad globs with generic content cause accidental triggers |
| Active skills | 5–15 total | description quality matters more than count |
| Commands | 3–7 | commands are not safety mechanisms |
| Hooks | 2–5 | safety hooks must use `failClosed: true` |

### 3. Three invariants every project keeps

Regardless of project type, these never get cut:

1. **Configuration is centralized** — no magic numbers, no scattered `os.environ`, no hardcoded provider names ([`configuration-management.mdc`](./cursor/.cursor/rules/configuration-management.mdc))
2. **Fail fast, no silent fallbacks** — errors stay visible, observable, retryable ([`core-principles.mdc`](./cursor/.cursor/rules/core-principles.mdc))
3. **Deterministic destructive-command blocking** — `failClosed: true`, stdlib-only, cross-platform ([`block-dangerous-shell.py`](./cursor/.cursor/hooks/block-dangerous-shell.py))

## Methodology

The two articles in [`docs/`](./docs) explain the engineering reasoning behind every artifact:

- [How to write an Agent Skill that actually triggers](./docs/how-to-write-agent-skills.md) — progressive disclosure, description discipline, the 50% trigger problem, when *not* to write a skill
- [How to write good Agent Rules, Hooks, and Commands](./docs/how-to-write-agent-rules-hooks-commands.md) — the 4-layer model, sweet spots, anti-patterns, decision tree

If you only read one thing, read those two.

## Compatibility

| Surface | Status |
|---|---|
| Cursor (rules + skills + commands + hooks) | ✅ primary target |
| Claude Code (CLAUDE.md + skills + hooks) | ✅ skills follow [agentskills.io](https://agentskills.io) spec; rules portable as `CLAUDE.md` content |
| Codex / generic LLM IDEs | ⚠️ rules and methodology transfer; hook integration requires equivalent event hooks |

## Project status

This is **v0.x**, extracted from one production Agent project. It is intentionally **not tagged** until it has been validated in ≥1 additional independent project. See [`CHANGELOG.md`](./CHANGELOG.md) for what's in scope.

If you adopt this kit and find a missing invariant, a leaked project name, or a hook that misbehaves on your platform, please open an issue or PR — that's exactly the feedback this kit needs to mature.

## Contributing

Issues and PRs welcome. When proposing a new rule / skill / command / hook, please answer:

1. What repeated failure does this prevent?
2. Why is this the lowest-cost layer that can enforce it?
3. Does it leak any project-specific context?

If you cannot answer all three, the proposal probably belongs in `docs/` as a methodology note rather than as a new artifact.

## License

[MIT](./LICENSE) — use it, fork it, ship it.

---

## 中文版

### 一句话定位

> 一份从生产 Agent 项目里抽取、剥离所有项目专属信息的可复用 `.cursor/` 工程经验包。

新项目打开就有工程基线，不是又一份提示词大礼包。

### 核心理念

```text
Rule    = 默认约束（短小、强硬、常驻）
Skill   = 专业能力（按需加载）
Command = 手动流程（review / retro / triage）
Hook    = 自动刹车（failClosed，不依赖模型）
```

第一性原理：**每条约束都应放在能执行它的最低成本、最高确定性层级。**

### 它解决什么问题

| 常见问题 | 这个 kit 怎么处理 |
|---|---|
| Rules 越写越长，污染上下文 | always-on rule ≤ 3 个、每个 ≤ 50 行 |
| Skills 写了不触发 | description 公式化 + 触发词 + 负向边界 |
| Hooks 名义安全，实际 fail-open | `failClosed: true` + stdlib-only Python 实现 |
| 项目业务名词混进通用规则 | 全部抽象化，仅在反例段保留作为反模式标本 |
| 新项目初始化又从零开始 | `bootstrap-cursor-package` skill 提供初始化 checklist |
| Pre-MVP 就乱写 architecture rule | bootstrap 流程明确：**有 ≥3 个稳定目录再写** |

### 三条不可裁剪的工程红线

1. **配置集中治理**：禁止 magic numbers / 散落 env 读取 / 硬编码 provider 名
2. **fail fast，不要静默 fallback**：错误必须响亮、可观测、可重试
3. **危险命令 deterministic 拦截**：`failClosed: true`，仅依赖 stdlib

### 快速开始

```bash
# 复制 .cursor 到新项目
cp -r path/to/agent-engineering-kit/cursor/.cursor ./.cursor

# 打开 Cursor，让 Agent 触发 bootstrap-cursor-package skill
# 关键词：初始化 .cursor / 套用工程经验包 / 复制 cursor 模板
```

注意：**pre-MVP 阶段不要写项目 architecture rule**。等代码稳定再补。

### 方法论文章

- [`docs/how-to-write-agent-skills.md`](./docs/how-to-write-agent-skills.md)：写一份会触发的 skill
- [`docs/how-to-write-agent-rules-hooks-commands.md`](./docs/how-to-write-agent-rules-hooks-commands.md)：rule / hook / command 的边界

### 项目状态

v0.x，从一个生产 Agent 项目抽取而来。在被 ≥1 个独立项目验证之前不会打 tag。欢迎在新项目里使用并提 issue / PR。

### 灵感来源

- [Anthropic Agent Skills](https://www.anthropic.com/engineering/skills) — progressive disclosure 模型
- [agentskills.io](https://agentskills.io) — 跨平台 skill 标准
- [Cursor Hooks](https://cursor.com/docs/hooks) — deterministic 安全层
- 实际生产 Agent 项目踩过的工程坑

如果这个 kit 帮到你，欢迎 ⭐ Star 让更多人看到。
