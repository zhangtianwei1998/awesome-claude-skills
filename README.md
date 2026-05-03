# Claude Code 插件与 Skill 全景图

> 整理时间：2026-05 | 覆盖范围：Claude Code 主流 Skill、插件与配置方案

本文档对当前较流行的 Claude Code skill、插件和配置方案进行系统整理，按功能分类介绍其原理与适用场景，并标注各方案之间的重叠与冲突关系。

---

## 目录

- [一、通用型](#一通用型)
  - [1.1 全流程工作流型](#11-全流程工作流型)
  - [1.2 省 Token 型](#12-省-token-型)
  - [1.3 使用痛点解决型](#13-使用痛点解决型)
- [二、具体领域型](#二具体领域型)
  - [2.1 前端开发](#21-前端开发)
  - [2.2 UI 设计](#22-ui-设计)
  - [2.3 官方技能库](#23-官方技能库)
- [三、重叠与冲突分析](#三重叠与冲突分析)

---

## 一、通用型

### 1.1 全流程工作流型

这类工具的核心价值在于为 Claude Code 提供完整的开发方法论，而非单一功能点。

---

#### superpowers (`obra/superpowers`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/obra/superpowers |
| **安装** | `/plugin install superpowers@claude-plugins-official` |

**原理与作用**

将 Claude 从"被动写代码"变成"遵循结构化工程方法论的高级工程师"。提供 14 个技能（skill），串联成完整的软件开发生命周期。

核心工作流（7 步）：

```
brainstorming（澄清需求）
  → using-git-worktrees（隔离工作区）
  → writing-plans（拆分原子任务）
  → subagent-driven-development（每任务独立子代理执行）
  → test-driven-development（强制 RED-GREEN-REFACTOR）
  → requesting-code-review（两阶段审查）
  → finishing-a-development-branch（合并决策）
```

**全部 14 个 Skills：**

| 类别 | Skill |
|------|-------|
| 测试 | `test-driven-development` |
| 调试 | `systematic-debugging`, `verification-before-completion` |
| 协作 | `brainstorming`, `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents` |
| 代码审查 | `requesting-code-review`, `receiving-code-review` |
| Git | `using-git-worktrees`, `finishing-a-development-branch` |
| 元技能 | `writing-skills`, `using-superpowers` |

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **gstack** | ⚠️ 冲突（都定义整体工作流） | 选其一作为主工作流；superpowers 更适合严格 TDD 项目，gstack 更适合需要真实浏览器测试和设计审查的全栈项目 |
| **andrej-karpathy-skills** | ✅ 兼容（行为约束层） | 建议同时安装；karpathy-skills 在 superpowers 框架内增加"谨慎性"约束 |
| **RTK** | ✅ 互补（命令输出压缩） | 建议同时安装 |
| **caveman** | ✅ 互补（响应风格压缩） | 视个人偏好，可同时安装 |

---

#### gstack (`garrytan/gstack`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/garrytan/gstack |
| **安装** | `npx skills add https://github.com/garrytan/gstack` |
| **Star 数** | ~54,000 |

**原理与作用**

Y Combinator CEO Garry Tan 开源的工程师 toolkit，通过 23 个专家角色 + 8 个 Power Tool 将 Claude 变成"虚拟工程师团队"。核心理念是角色切换（role-switching）而非单一 AI 助手。

核心 Sprint 循环：

```
/plan（CEO 角色：战略规划）
  → /design（Designer：UI 设计，对抗 AI slop）
  → build
  → /review（Reviewer：结构化 code review）
  → /qa（QA Lead：Playwright 真实浏览器测试）← 最大差异点
  → /ship
  → /reflect
```

**主要命令：**

| 命令 | 角色 | 特点 |
|------|------|------|
| `/ceo` / `/plan-ceo-review` | CEO | 挑战 feature scope 和排期 |
| `/design`, `/design-html` | Designer | 生成/审查 UI，专门对抗 AI 设计通病 |
| `/review` | Code Reviewer | 可调用 Codex 获取第二意见 |
| `/qa` | QA Lead | **Playwright 真实浏览器端到端测试** |
| `/browse` | Browser | 真实浏览器操作 |

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **superpowers** | ⚠️ 冲突（都定义整体工作流） | 选其一；superpowers 更工程化（强 TDD），gstack 更全栈化（有 UI 设计和真实浏览器测试） |
| **andrej-karpathy-skills** | ✅ 兼容（行为约束层） | 建议同时安装 |
| **vercel-labs/agent-skills** | ✅ 互补（前端 React 最佳实践） | 可同时安装 |

---

#### andrej-karpathy-skills (`forrestchang/andrej-karpathy-skills`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/forrestchang/andrej-karpathy-skills |
| **安装** | `/plugin install andrej-karpathy-skills@karpathy-skills` 或 `curl -o CLAUDE.md <url>` |
| **Star 数** | ~107,000 |

**原理与作用**

由 Andrej Karpathy 一条推文直接催生的行为约束层 skill。不扩展 Claude 的能力，而是纠正 LLM 编码中最常见的失误模式。

**四条原则：**

```
1. Think Before Coding  → 显式声明假设，模糊时展示多种解读，不静默决策
2. Simplicity First     → 最少代码，不加推测性抽象，不加未请求的配置项
3. Surgical Changes     → 只动被要求修改的部分，不顺带"改进"周边代码
4. Goal-Driven Execution → 将指令转化为可验证目标，让 Claude 自主循环直到达成
```

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **superpowers** | ✅ 兼容（可叠加） | 建议同时安装；karpathy-skills 在 superpowers 方法论上加"谨慎性"约束 |
| **gstack** | ✅ 兼容（行为约束，不影响角色系统） | 可同时安装 |
| **addyosmani/agent-skills** | ✅ 兼容（行为约束层） | 建议同时安装，agent-skills 的"防跳步"原则与 karpathy-skills 理念相近但覆盖更广 |

---

#### addyosmani/agent-skills (`addyosmani/agent-skills`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/addyosmani/agent-skills |
| **安装** | `/plugin` → Discover 标签页安装，或 `cp -r agent-skills/skills/skill-name/ ~/.claude/skills/` |
| **Star 数** | ~8,600 |

**原理与作用**

Chrome DevRel 负责人 Addy Osmani 发布的生产级 AI 编码代理工作流集合。与普通 skill pack 的最大区别：**"Process, not prose"**——不是告诉 Claude 知识，而是强制走流程（含检查点、退出标准、反合理化条款）。

核心工作流（SDLC 六阶段）：

```
DEFINE（idea-refine → spec-driven-development）
  → PLAN（planning-and-task-breakdown）
  → BUILD（incremental-implementation → TDD → frontend-ui-engineering → react-best-practices...）
  → VERIFY（browser-testing-with-devtools → systematic-debugging）
  → REVIEW（code-review-and-quality → simplify → security-and-hardening → performance-optimization）
  → SHIP（git-workflow → ci-cd → documentation → shipping-and-launch）
```

**全部 19 个 Skills（按阶段）：**

| 阶段 | Skills |
|------|--------|
| DEFINE | `idea-refine`, `spec-driven-development` |
| PLAN | `planning-and-task-breakdown` |
| BUILD | `incremental-implementation`, `test-driven-development`, `context-engineering`, `frontend-ui-engineering`, `api-and-interface-design`, `source-driven-development`, `react-best-practices` |
| VERIFY | `browser-testing-with-devtools`, `systematic-debugging` |
| REVIEW | `code-review-and-quality`, `simplify`, `security-and-hardening`, `performance-optimization` |
| SHIP | `git-workflow-and-versioning`, `ci-cd-and-automation`, `deprecation-and-migration`, `documentation-and-adrs`, `shipping-and-launch` |

**三大设计特点：**
- **防止跳步**：每个 skill 预埋"代理可能给出的跳过理由"并给出反驳
- **证据前置**：完成任何步骤必须提供可验证的实物证据（测试通过输出、构建结果等）
- **渐进加载**：SKILL.md 为入口，支撑文档按需加载，token 消耗低于整包加载

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **superpowers** | ⚠️ 部分重叠（都覆盖 SDLC 全流程） | agent-skills 模块化可按需取用，superpowers 是固定七阶段 TDD 流水线；可互补——用 agent-skills 的单个技能补强 superpowers 缺失环节 |
| **gstack** | ⚠️ 部分重叠（都覆盖全流程） | gstack 带产品/CEO 视角和真实浏览器测试，agent-skills 更偏工程规范；可共存但有角色重叠 |
| **andrej-karpathy-skills** | ✅ 兼容（理念相近，覆盖范围不同） | 两者均强调"不跳步、不猜测"，可叠加使用 |
| **vercel-labs/agent-skills** | ✅ 互补 | addyosmani 覆盖完整 SDLC，vercel-labs 专注 React/Next.js 代码规范；前端项目建议同时安装 |

---

### 1.2 省 Token 型

---

#### RTK - Rust Token Killer (`rtk-ai/rtk`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/rtk-ai/rtk |
| **安装** | `brew install rtk` 或 `cargo install --git https://github.com/rtk-ai/rtk` |

**原理与作用**

用 Rust 编写的 CLI 代理工具。通过 Claude Code 的 `PreToolUse` hook，在 Bash 命令输出进入上下文窗口前对其过滤、压缩。

工作原理：

```
Claude 发出 Bash 工具调用
  → PreToolUse hook 拦截
  → rtk 重写命令（git status → rtk git status）
  → 语义过滤（去噪音：通过的测试、进度条、重复日志）
  → 智能截断（大型 diff 只保留关键 hunk）
  → 压缩后的输出进入 Claude 上下文
```

**实际压缩效果：**

| 命令 | 原始 tokens | 压缩后 | 节省 |
|------|------------|--------|------|
| cargo test（262 个通过测试）| 4,823 | 11 | **99%** |
| 大型 git diff | 21,500 | 1,259 | **94%** |
| 平均 | — | — | **60-90%** |

**⚠️ 重要限制：** `PreToolUse` hook 只拦截 Bash 工具调用。`Read`、`Grep`、`Glob` 等原生工具不经过 hook，需显式使用 `rtk read`、`rtk grep`、`rtk find`。

**常用命令：**

```bash
rtk gain              # 查看累计节省的 token 统计
rtk gain --history    # 查看历史命令的节省记录
rtk discover          # 找到未使用 rtk 的遗漏机会
rtk proxy <cmd>       # 不过滤，直接透传原始命令（调试用）
```

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **caveman** | ✅ 互补（不同层级） | 建议同时使用；RTK 压缩命令输出（输入侧），caveman 压缩 Claude 的响应输出（输出侧） |

---

#### caveman (`JuliusBrussee/caveman`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/JuliusBrussee/caveman |
| **安装** | `npx skills add JuliusBrussee/caveman` |

**原理与作用**

一个 Skill，通过改变 Claude 的输出风格来削减响应 token。去掉客套话、过渡句、重复摘要，只保留代码块和技术要点。

```
激活：/caveman 或 "caveman mode"
退出：stop caveman 或 "normal mode"
```

**三档压缩：**

| 档位 | 效果 |
|------|------|
| Lite | 去填充词，保留自然句子 |
| Full（默认）| 穴居人式简写 |
| Ultra | 电报式极限压缩 |

平均节省：output token **-65~75%**，input token **-45%**，极端 debug 场景最高 **-87%**

**子技能：** `caveman-commit`、`caveman-review`、`caveman-compress`

**⚠️ 注意：** 开启后会显著影响可读性，与需要详细解释的工作流（如 brainstorming、writing-plans）不兼容。建议在机械性执行任务时开启，规划阶段关闭。

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **RTK** | ✅ 互补 | 建议同时使用；分别作用于不同方向（输入 vs 输出） |
| **superpowers brainstorming/writing-plans** | ⚠️ 部分冲突 | 需要详细输出时请关闭 caveman |

---

### 1.3 使用痛点解决型

---

#### 任务完成音效

**代表方案：**

| 仓库 | Stars | 特点 |
|------|-------|------|
| [mylee04/code-notify](https://github.com/mylee04/code-notify) | ~208 | 最成熟；声音 + TTS + OS 原生通知；Homebrew 安装；支持 Claude Code / Codex / Gemini CLI |
| [ChanMeng666/claude-code-audio-hooks](https://github.com/ChanMeng666/claude-code-audio-hooks) | ~53 | 26 种 hook 事件，2 套音效主题，支持 Cursor |

**DIY 最简方案（macOS）：**

在 `~/.claude/settings.json` 中添加：

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "/usr/bin/afplay --volume 0.3 /System/Library/Sounds/Glass.aiff"
      }]
    }]
  }
}
```

Linux 替换为 `aplay`，Windows 替换为 PowerShell `[System.Media.SystemSounds]::Asterisk.Play()`。

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| code-notify vs audio-hooks | ⚠️ 功能重叠 | 选一个；推荐 `mylee04/code-notify`（更成熟，Homebrew 安装更简便） |

---

#### 用量常驻显示（Statusline）

**代表方案：**

| 仓库 | Stars | 特点 |
|------|-------|------|
| [daniel3303/ClaudeCodeStatusLine](https://github.com/daniel3303/ClaudeCodeStatusLine) | ~459 | 最均衡；模型名 / token 用量 / 5h·7d rate limit / Git 分支 |
| [rz1989s/claude-code-statusline](https://github.com/rz1989s/claude-code-statusline) | ~439 | 功能最全；28 组件，费用预测，缓存效率，MCP 监控，TOML 配置，Catppuccin 主题 |
| [long-910/tmux-claude-status](https://github.com/long-910/tmux-claude-status) | ~4 | tmux 专用，显示 5h/7d 用量百分比或费用 |

**工作原理（基于 Claude Code `statusCommand` API）：**

```
Claude Code 定时调用 statusCommand 脚本
  → 脚本收到 JSON 载荷（cost / context 用量 / model / rate limit）
  → 脚本格式化输出到终端底部状态栏
```

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| daniel3303 vs rz1989s | ⚠️ 功能重叠 | 推荐 `daniel3303`（简洁易用），追求极致定制选 `rz1989s` |

---

## 二、具体领域型

### 2.1 前端开发

---

#### vercel-labs/agent-skills

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/vercel-labs/agent-skills |
| **安装** | `npx skills add vercel-labs/agent-skills` |

**原理与作用**

Vercel 官方维护的 Skill 集合，聚焦 React / Next.js 生产质量规范。

**核心 Skills：**

| Skill | 重点 |
|-------|------|
| Web Design Guidelines | 100+ 条 UI 规则（可访问性、性能、UX），每周 133,000 次安装 |
| React Best Practices | 57 条性能优化规则（React/Next.js） |
| React Native Guidelines | 16 条规则（性能、架构、平台特性）|
| Vercel Composition Patterns | 复合组件 / context provider 替代 boolean prop 泛滥 |
| Vercel Deploy Claimable | 一键部署到 Vercel |

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **addyosmani/agent-skills** | ✅ 互补 | agent-skills 覆盖完整 SDLC，vercel-labs 专注 React/Next.js 规范；前端项目可同时安装 |
| **anthropics/skills frontend-design** | ⚠️ 部分重叠（UI 最佳实践） | vercel-labs 更专注 React/Next.js；anthropics 更通用；可共存，不冲突 |
| **gstack /design** | ✅ 互补（规则集 vs. AI 角色）| 可共存 |

---

### 2.2 UI 设计

---

#### awesome-design-md (`VoltAgent/awesome-design-md`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/VoltAgent/awesome-design-md |
| **安装** | `npx skills add VoltAgent/awesome-design-md` |

**原理与作用**

收录 68+ 个品牌设计系统的 `DESIGN.md` 文件集合。`DESIGN.md` 是 Google Stitch 于 2026 年 4 月引入的设计系统规范格式，AI 读取后可生成像素级还原目标品牌风格的 UI。

```
DESIGN.md 包含：颜色 token + 排版规范 + 间距系统 + 组件状态 + 响应式规则
  → Claude 读取后理解品牌视觉语言
  → 支持："build me a page like Stripe" 式自然语言命令
```

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **vercel-labs Web Design Guidelines** | ✅ 互补 | vercel-labs 是通用 UI 规则，awesome-design-md 是具体品牌系统；可共存 |
| **anthropics/skills canvas-design / theme-factory** | ✅ 互补 | anthropics 侧重生成创意内容，awesome-design-md 侧重还原品牌风格 |
| **gstack /design** | ✅ 互补 | gstack 提供设计角色，awesome-design-md 提供品牌素材 |

---

### 2.3 官方技能库

---

#### anthropics/skills

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/anthropics/skills |
| **安装** | `/plugin install example-skills@anthropic-agent-skills` |
| **Star 数** | ~127,000 |

**原理与作用**

Anthropic 官方 Skill 展示库，包含 17 个技能，同时是 `SKILL.md` 格式的规范来源（`spec/agent-skills-spec.md`）。

**完整 Skills 列表：**

| 类别 | Skills |
|------|--------|
| 创意与设计 | `algorithmic-art`, `canvas-design`, `frontend-design`, `theme-factory`, `slack-gif-creator` |
| 开发与技术 | `claude-api`, `mcp-builder`, `web-artifacts-builder`, `webapp-testing`, `skill-creator` |
| 企业与沟通 | `brand-guidelines`, `doc-coauthoring`, `internal-comms` |
| 文档（Source-available）| `docx`, `pdf`, `pptx`, `xlsx` |

**重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **vercel-labs/agent-skills** | ⚠️ 部分重叠（`frontend-design`）| `anthropics/skills frontend-design` 更通用；vercel-labs 更专注 React/Next.js；可共存 |
| **awesome-design-md** | ✅ 互补 | anthropics 偏生成创意内容，awesome-design-md 偏还原品牌风格 |

---

## 三、重叠与冲突分析

### 3.1 快速参考表

| 工具 A | 工具 B | 关系 | 建议 |
|--------|--------|------|------|
| superpowers | gstack | ⚠️ **主要冲突** | 选一个作为整体工作流主框架 |
| superpowers | addyosmani/agent-skills | ⚠️ 部分重叠 | 可互补——agent-skills 按需取单个技能补强 superpowers 缺失环节 |
| gstack | addyosmani/agent-skills | ⚠️ 部分重叠 | gstack 带产品视角，agent-skills 更纯工程；可共存但角色有重叠 |
| superpowers | andrej-karpathy-skills | ✅ 互补 | 建议叠加 |
| RTK | caveman | ✅ 互补 | 建议叠加（分别压缩输入/输出）|
| vercel-labs/agent-skills | addyosmani/agent-skills | ✅ 互补 | addyosmani 覆盖完整 SDLC，vercel-labs 专注 React/Next.js；前端项目建议同时安装 |
| daniel3303/ClaudeCodeStatusLine | rz1989s/claude-code-statusline | ⚠️ 功能重叠 | 选一个，推荐 daniel3303 |
| mylee04/code-notify | ChanMeng666/claude-code-audio-hooks | ⚠️ 功能重叠 | 选一个，推荐 mylee04/code-notify |
| anthropics/skills (frontend-design) | vercel-labs (Web Design Guidelines) | ✅ 可共存 | vercel-labs 更专，anthropics 更通用 |

### 3.2 推荐搭配

**后端 / 工程向项目：**
```
superpowers + andrej-karpathy-skills + RTK + caveman（按需）+ daniel3303/ClaudeCodeStatusLine
```

**全栈 / 前端向项目：**
```
addyosmani/agent-skills + vercel-labs/agent-skills + RTK + gstack（/qa 浏览器测试）
```

**极简省 Token 配置：**
```
RTK + caveman（Ultra 档）
```

**UI 密集型项目：**
```
awesome-design-md + gstack（/design 命令）+ anthropics/skills（frontend-design）
```

---

## 附录：所有工具速查

| 工具 | 仓库 | 类型 | 核心价值 |
|------|------|------|---------|
| superpowers | [obra/superpowers](https://github.com/obra/superpowers) | 全流程 | TDD + 子代理工程方法论 |
| gstack | [garrytan/gstack](https://github.com/garrytan/gstack) | 全流程 | 虚拟工程团队 + 浏览器测试 |
| agent-skills | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | 全流程 | SDLC 六阶段模块化工作流（19 个 skills）|
| andrej-karpathy-skills | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | 行为约束 | LLM 编码失误纠正（4 条原则）|
| RTK | [rtk-ai/rtk](https://github.com/rtk-ai/rtk) | 省 Token | 命令输出压缩（60-99%）|
| caveman | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | 省 Token | 响应输出压缩（65-87%）|
| code-notify | [mylee04/code-notify](https://github.com/mylee04/code-notify) | 痛点解决 | 任务完成音效 + OS 通知 |
| ClaudeCodeStatusLine | [daniel3303/ClaudeCodeStatusLine](https://github.com/daniel3303/ClaudeCodeStatusLine) | 痛点解决 | 用量/费用常驻状态栏 |
| vercel-labs/agent-skills | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | 前端开发 | React/Next.js 规范 |
| awesome-design-md | [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | UI 设计 | 68+ 品牌设计系统 |
| anthropics/skills | [anthropics/skills](https://github.com/anthropics/skills) | 官方库 | 17 个官方 Skill 展示 |

---

*欢迎 PR 补充新工具*
