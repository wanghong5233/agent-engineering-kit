# Agent Engineering Kit

> 可复用的 `.cursor/` 工程经验包。  
> 目标：让新项目在第一天就有规范、可观测、可维护的 Agent 协作基线。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Cursor](https://img.shields.io/badge/Cursor-compatible-1f6feb)](https://cursor.com/docs/rules)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-spec-0aa)](https://agentskills.io)

[English](./README_EN.md) | **中文**

---

## 一句话定位

这不是提示词集合，而是一套分层治理的 `.cursor/` 模板：

```text
Rule    = 默认约束（短小、稳定）
Skill   = 专业流程（按需加载）
Command = 手动流程（复盘、审查）
Hook    = 自动防线（确定性拦截）
```

第一性原理：**每条约束都应该放在最低成本、最高确定性的执行层。**

---

## 适用场景

- 新建 Agent / LLM 项目，需要快速建立工程规范
- 现有项目的 `.cursor/` 结构混乱，需要重整
- 想把跨项目经验沉淀成可复用资产，而不是每次从零写规则

---

## 包含内容

```text
agent-engineering-kit/
├── cursor/
│   └── .cursor/
│       ├── rules/
│       │   ├── core-principles.mdc              # 核心工程约束：fail fast、禁止静默 fallback、禁止硬编码密钥
│       │   ├── cursor-package-boundaries.mdc    # 分层边界：规则/技能/命令/Hook 各司其职，避免 .cursor 膨胀
│       │   ├── configuration-management.mdc     # 配置治理：集中配置、类型校验、禁止魔法参数
│       │   └── agent-runtime-contracts.mdc      # 运行时契约：LLM/RAG/tool/异步任务状态与失败语义
│       ├── skills/
│       │   ├── bootstrap-cursor-package/        # 初始化流程：按项目阶段装配 .cursor，避免过早写架构规则
│       │   ├── agent-debugging/                 # 故障排查：系统化定位 Agent 问题与根因
│       │   ├── llm-observability-and-evals/     # 观测评测：建立 tracing、评测集与上线验收门槛
│       │   ├── tool-and-mcp-design/             # 工具设计：定义稳定的 Tool/MCP 接口与错误语义
│       │   ├── llm-cost-optimizer/              # 成本优化：识别高成本路径并建立预算与降级策略
│       │   ├── writing-skill/                   # 元技能：规范 SKILL.md 写法、触发词与边界
│       │   ├── writing-tech-article/            # 技术写作：沉淀高密度、可复用的工程文章
│       │   ├── writing-readme/                  # README 治理：10 秒可扫描、双语同步、信息分层
│       │   ├── writing-architecture-docs/       # 架构文档：现状陈述 + 第一性原理 + 契约化表达
│       │   ├── writing-issue-backlog/           # 问题台账：证据优先，区分现象/根因/下一步
│       │   ├── writing-pitfall-archive/         # 坑点归档：复盘可复发故障，沉淀不可破坏的不变量
│       │   └── writing-engineering-playbook/    # 工程手册：跨项目提炼可迁移的工程直觉与反模式
│       ├── commands/
│       │   ├── review.md                        # 审查流程：聚焦风险、回归与缺失测试
│       │   └── retro.md                         # 复盘流程：从会话与改动中沉淀可复用改进项
│       ├── hooks.json                           # Hook 编排：事件绑定、匹配器与 failClosed 策略
│       └── hooks/
│           └── block-dangerous-shell.py         # 安全闸门：确定性拦截高风险 shell/git 命令
├── docs/
│   ├── how-to-write-agent-skills.md             # 技能方法论：何时写 skill、如何触发、如何控上下文成本
│   └── how-to-write-agent-rules-hooks-commands.md # 分层方法论：rule/hook/command 的边界与甜点区
├── CHANGELOG.md
├── LICENSE
└── README.md
```

---


## 项目阶段策略

| 阶段 | 应该做 | 不应该做 |
|---|---|---|
| pre-MVP | 保留通用 rules / skills / hooks / commands | 预先写项目架构 rule |
| MVP 成形 | 补项目架构 rule（只写已存在目录和职责） | 写尚未实现的服务边界 |
| 稳定迭代 | 增加项目专属命令和检查项 | 把项目名词回流到通用层 |

---

## 参考文档

- [如何写一份让 Agent 真正会用的 Skill](./docs/how-to-write-agent-skills.md)
- [如何写好 Agent Rules、Hooks 和 Commands](./docs/how-to-write-agent-rules-hooks-commands.md)

---

## 开源协议

[MIT](./LICENSE)
