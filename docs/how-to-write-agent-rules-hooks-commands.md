# 如何写好 Agent Rules、Hooks 和 Commands

> 目标读者：长期维护 Cursor / Claude Code / Agent 工程配置的人。  
> 写作目标：把 `.cursor` 从提示词杂物间，变成可复制的生产级工程经验包。

## 一句话结论

**Rule 是默认行为，Hook 是强制边界，Command 是人工入口。三者的核心不是“多写提示词”，而是把经验放到最低成本、最高确定性的层级。**

## 1. 心智模型

```text
Agent 行为控制栈：

always-on context  →  Rule       →  让模型默认知道
on-demand context  →  Skill      →  让模型需要时会做
manual workflow    →  Command    →  让人显式启动流程
deterministic gate →  Hook       →  让危险动作必须被机器拦截
```

第一性原理：**越靠下越确定，越靠上越消耗上下文。**

| 层级 | 成本 | 确定性 | 典型用途 |
|---|---:|---:|---|
| Rule | 中，高频进入上下文 | 中 | 编码约束、架构边界 |
| Skill | 中，仅触发后进入 | 中 | 调试、写作、MCP 设计、成本优化 |
| Command | 低，手动触发 | 中 | review、retro、issue triage、release check |
| Hook | 低，事件触发 | 高 | 拦截危险命令、保护 secrets、格式化、审计 |

## 2. Rule：不是项目百科，而是默认约束

Rule 适合回答：**“以后在这个范围内，Agent 默认必须按什么边界做事？”**

### 适合写成 Rule

| 场景 | 例子 |
|---|---|
| 高频重复约束 | 不要 silent fallback；不要 bare `except` |
| 项目架构边界 | `backend/<core_service>/`、子服务 A、子服务 B 的职责 |
| 文件范围规范 | `frontend/**/*.tsx` 的 UI 状态语义 |
| Agent 输出契约 | RAG / 研究类 / 工具类回答必须有 evidence / citation |
| 反复犯错后的防线 | 不要把 `docs/private/` 当公开文档 |

### 不适合写成 Rule

| 反模式 | 应该放哪里 |
|---|---|
| 长篇教程 | `docs/` 或 skill `references/` |
| 多步骤调试流程 | Skill |
| 发布前 checklist | Command |
| 阻止危险 shell | Hook |
| 一次性项目计划 | 普通文档或 issue |

### Rule 甜点区

| 指标 | 建议 |
|---|---:|
| always-on rule 数量 | 1-3 个 |
| always-on rule 长度 | 20-50 行 |
| file-scoped rule 数量 | 5-10 个以内 |
| 单个 rule 关注点 | 1 个主问题 |
| description 长度 | 1 句，说明触发边界 |
| glob 范围 | 越窄越好，避免 `**/*` |

经验判断：**超过 100 行的 always-on rule 基本就是上下文税；超过 500 行的 rule 应拆成 skill 或 docs。**

### 好 Rule 长什么样

```text
frontmatter: 触发范围清楚
body: 只写默认约束
examples: 只放最短正反例
references: 引用文档，不复制文档
```

## 3. Hook：不是建议，是机器刹车

Hook 适合回答：**“这件事如果做错，会不会伤害代码、数据、密钥、git 历史或生产环境？”**

如果答案是会，就不要只写 rule。要写 hook。

### 适合写成 Hook

| 场景 | Hook 类型 |
|---|---|
| 禁止 `git push --force`、`git reset --hard` | `preToolUse` + `Shell`，或稳定后使用 `beforeShellExecution` |
| 禁止读写 `.env`、`.pem`、`.key` | `beforeReadFile` / `afterFileEdit` |
| 编辑后自动格式化 | `afterFileEdit` |
| 高风险 MCP 调用前审批 | `beforeMCPExecution` |
| 记录 Agent 行为审计 | `afterShellExecution` / `afterFileEdit` |

### Hook 甜点区

| 指标 | 建议 |
|---|---:|
| 项目级 hook 数量 | 2-5 个 |
| 单个 hook 脚本长度 | 50-150 行 |
| 安全 hook | `failClosed: true` |
| timeout | 3-10 秒 |
| matcher | 简单、宽松，复杂判断放脚本里 |
| 依赖 | 优先标准库，少依赖 `jq`、Node 包、项目环境 |

关键原则：**安全 hook 宁愿少，也要确定。一个会 fail-open 的安全 hook 比没有 hook 更危险，因为它制造了虚假的安全感。**

### 好 Hook 长什么样

```text
stdin JSON → 解析事件 → 确定性判断 → 输出最小 permission JSON
```

在 `failClosed: true` 下，允许路径也要输出有效 JSON，例如 `{"permission": "allow"}`；阻止路径输出 `permission: "deny"` 并给出 `user_message` / `agent_message`。当前 Cursor 社区仍有 `beforeShellExecution` deny/ask 兼容问题，因此高风险 shell 拦截可以优先放在 `preToolUse` + `Shell`。好 hook 不依赖模型理解，不靠自然语言猜测，不做复杂业务生成。

## 4. Command：手动启动的固定工作流

Command 是自定义 slash command。你在 Cursor 里输入 `/` 时，可以触发项目或个人 commands。

它适合回答：**“我经常需要手动让 Agent 按同一套步骤做这件事吗？”**

### Command 和 Rule 的区别

| 问题 | 用 Rule | 用 Command |
|---|---|---|
| 是否默认生效 | 是 | 否 |
| 是否需要用户显式触发 | 否 | 是 |
| 是否适合 checklist | 一般不适合 | 适合 |
| 是否适合安全强制 | 不适合 | 不适合 |
| 典型例子 | 架构边界 | `/review`、`/retro` |

### 适合写成 Command

| Command | 用途 |
|---|---|
| `/review` | 审查当前 diff 的安全、契约、质量问题 |
| `/retro` | 复盘对话，把经验沉淀到 rule / skill / hook / docs |
| `/issue-triage` | 把 bug 整理成 symptom、evidence、hypothesis、DoD |
| `/release-check` | 发布前人工检查 env、migration、health、rollback |
| `/skill-audit` | 检查 skill 数量、触发描述、职责重叠、references 分层 |

### Command 甜点区

| 指标 | 建议 |
|---|---:|
| 项目级 commands 数量 | 3-7 个 |
| 单个 command 长度 | 20-80 行 |
| 输出格式 | 必须固定 |
| 职责 | 一个 command 只做一个工作流 |
| 触发方式 | 人显式触发，不承担默认约束 |

Command 的价值不是“更短的 prompt”，而是**把高频人工流程标准化**。

## 5. 决策树

```text
这条经验应该放哪里？

会造成不可逆损害？
  ├─ 是 → Hook
  └─ 否
      └─ 是否默认长期适用？
          ├─ 是 → Rule
          └─ 否
              └─ 是否需要复杂资料/示例/脚本？
                  ├─ 是 → Skill
                  └─ 否
                      └─ 是否是手动流程？
                          ├─ 是 → Command
                          └─ 否 → 普通 docs / issue，不要塞进 .cursor
```

## 6. `.cursor` 经验包的推荐配比

| 组件 | 推荐数量 | 备注 |
|---|---:|---|
| always-on rules | 1-3 | 保持 `core-principles` 短小 |
| file-scoped rules | 3-8 | 架构、runtime、`.cursor` 边界 |
| project skills | 5-8 | 文档类 + Agent 工程类，后续可迁个人级 |
| commands | 3-7 | review、retro 优先，triage/release 后续补 |
| hooks | 2-5 | 先做危险 shell，后续补 secret 保护 |

关键不是数量越多越专业，而是每一层只做自己确定擅长的事。

## 7. 反模式

| 反模式 | 后果 | 改法 |
|---|---|---|
| 把所有东西塞进 always-on rule | 上下文污染，Agent 更不稳定 | 拆到 scoped rule / skill / docs |
| 用 rule 保护危险 git 操作 | Agent 可能忘 | 用 hook 拦截 |
| 用 command 做安全策略 | 用户不触发就失效 | 用 hook |
| 为每个小偏好建 skill | description 常驻税变高 | 合并或放 references |
| hook 依赖复杂外部工具 | 换机器就失效 | 用标准库 |
| command 没有固定输出格式 | 每次结果漂移 | 写死 checklist / schema |
| 复制社区配置不裁剪 | 与项目不匹配 | 只迁移不变量 |

## 8. 落地清单

- [ ] 是否只有 1-3 个 always-on rules？
- [ ] 每个 rule 是否只有一个主边界？
- [ ] 是否把长教程放到了 docs 或 skill references？
- [ ] 安全相关内容是否由 hook 强制执行？
- [ ] hook 是否使用 `failClosed: true`？
- [ ] hook 是否能在新机器用标准库运行？
- [ ] commands 是否都是手动高频流程？
- [ ] 每个 command 是否有固定输出格式？
- [ ] 是否避免把社区 pack 整包常驻？
- [ ] 新增 `.cursor` 内容前是否先检查已有层级能否承接？

## 9. 记忆版

```text
Rule    = 默认约束，少而硬
Skill   = 专业能力，按需读
Command = 手动流程，可复用
Hook    = 自动刹车，必须稳
```

真正成熟的 Agent 工程配置，不是 prompt 写得多，而是把每条经验放在正确的执行层。
