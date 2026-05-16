# Agent Engineering Kit

> A reusable `.cursor/` engineering kit.  
> Goal: give every new project a reliable, observable, and maintainable Agent collaboration baseline on day one.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Cursor](https://img.shields.io/badge/Cursor-compatible-1f6feb)](https://cursor.com/docs/rules)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-spec-0aa)](https://agentskills.io)

**English** | [中文](./README.md)

---

## Project Positioning

This repository provides a `.cursor/` package that can be copied into a project: rules, skills, commands, and hooks are maintained as separate layers.

```text
Rule    = default constraints (small, stable)
Skill   = specialized workflows (loaded on demand)
Command = manual workflows (review, retro)
Hook    = automatic safety gates (deterministic interception)
```

Design rule: keep each constraint in the layer that can enforce it best; if a hook can block it automatically, do not rely on model memory.

---

## When to Use

- Starting a new Agent/LLM project and needing an engineering baseline fast
- Cleaning up an existing `.cursor/` setup that has grown chaotic
- Turning cross-project lessons into reusable assets instead of rewriting rules from scratch

---

## What’s Included

```text
agent-engineering-kit/
├── cursor/
│   └── .cursor/
│       ├── rules/
│       │   ├── core-principles.mdc              # Core constraints: fail fast, no silent fallback, no hardcoded secrets
│       │   ├── cursor-package-boundaries.mdc    # Layer boundaries: clear responsibilities for rules/skills/commands/hooks
│       │   ├── configuration-management.mdc     # Config governance: centralized config, typed validation, no magic numbers
│       │   └── agent-runtime-contracts.mdc      # Runtime contracts: LLM/RAG/tool/async states and failure semantics
│       ├── skills/
│       │   ├── bootstrap-cursor-package/        # Initialization workflow: stage-based .cursor setup
│       │   ├── agent-debugging/                 # Debugging workflow: systematic Agent failure triage
│       │   ├── llm-observability-and-evals/     # Observability/evals: tracing, eval sets, launch gates
│       │   ├── tool-and-mcp-design/             # Tool design: stable Tool/MCP interfaces and error semantics
│       │   ├── llm-cost-optimizer/              # Cost optimization: budget guardrails and downgrade strategy
│       │   ├── writing-skill/                   # Meta skill: how to write SKILL.md with triggers and boundaries
│       │   ├── writing-tech-article/            # Technical writing: high-density reusable engineering articles
│       │   ├── writing-readme/                  # README governance: 10-second scanability and bilingual sync
│       │   ├── writing-architecture-docs/       # Architecture docs: current state + first principles + contracts
│       │   ├── writing-issue-backlog/           # Issue backlog: evidence-first problem tracking
│       │   ├── writing-pitfall-archive/         # Pitfall archive: recurring failure patterns and invariants
│       │   └── writing-engineering-playbook/    # Engineering playbook: transferable patterns and anti-patterns
│       ├── commands/
│       │   ├── review.md                        # Review workflow: risk/regression/test-gap focused checks
│       │   └── retro.md                         # Retrospective workflow: convert sessions into reusable improvements
│       ├── hooks.json                           # Hook orchestration: event binding, matchers, failClosed strategy
│       └── hooks/
│           └── block-dangerous-shell.py         # Safety gate: deterministic blocking for high-risk shell/git commands
├── docs/
│   ├── how-to-write-agent-skills.md             # Skill methodology: when to write skills, trigger design, context cost
│   └── how-to-write-agent-rules-hooks-commands.md # Layer methodology: boundaries and sweet spots
├── CHANGELOG.md
├── LICENSE
└── README.md
```

---

## Quick Use

### 1) Copy the template

```bash
cp -r path/to/agent-engineering-kit/cursor/.cursor ./.cursor
```

### 2) Trigger the bootstrap skill in Cursor

You can use prompts like:

- initialize `.cursor`
- apply the engineering kit
- copy cursor template

Mapped skill: `bootstrap-cursor-package`

### 3) Verify the safety hook

```bash
echo '{"tool_name":"Shell","tool_input":{"command":"git status"}}' | \
  python cursor/.cursor/hooks/block-dangerous-shell.py
# {"permission":"allow"}
```

---

## Non-negotiable Constraints

1. **Centralized configuration**: no magic numbers, no scattered env reads, no hardcoded provider/model names  
2. **Fail fast**: no silent fallback; failures must remain visible, traceable, and recoverable  
3. **Deterministic safety gate**: `failClosed: true` hook for destructive command protection

---

## Stage Strategy

| Stage | Do | Don’t |
|---|---|---|
| pre-MVP | keep generic rules/skills/hooks/commands | write project architecture rules too early |
| MVP forming | add architecture rule only for existing directories and responsibilities | define boundaries for services not implemented yet |
| stable iteration | add project-specific commands and checks | leak project-specific terms back into generic layers |

---

## References

- [How to write an Agent Skill that actually triggers](./docs/how-to-write-agent-skills.md)
- [How to write good Agent Rules, Hooks and Commands](./docs/how-to-write-agent-rules-hooks-commands.md)

---

## License

[MIT](./LICENSE)
