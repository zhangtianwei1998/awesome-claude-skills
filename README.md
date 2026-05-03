# Claude Code 插件与 Skill 全景图

> 整理时间：2026-05 | 覆盖范围：Claude Code 主流 Skill、插件与配置方案

本文档对当前较流行的 Claude Code skill、插件和配置方案进行系统整理，按功能分类介绍其原理与适用场景，并标注各方案之间的重叠与冲突关系。

---

## 目录

- [一、通用型](#一通用型)
  - [1.1 含完整工作流型](#11-含完整工作流型)
  - [1.2 规范辅助型](#12-规范辅助型)
  - [1.3 省 Token 型](#13-省-token-型)
  - [1.4 使用痛点解决型](#14-使用痛点解决型)
- [二、具体领域型](#二具体领域型)
  - [2.1 前端开发](#21-前端开发)
  - [2.2 UI 设计](#22-ui-设计)
- [三、附录：所有工具速查](#三附录所有工具速查)

---

## 一、通用型

### 1.1 含完整工作流型

提供完整的端到端开发方法论，将 Claude 的行为串联成有顺序、有检查点的开发生命周期。

#### 概览

| 工具 | 仓库 | Stars | 简介 |
|------|------|-------|------|
| superpowers | [obra/superpowers](https://github.com/obra/superpowers) | 177k | 14 个 skill 串联 TDD 驱动的七阶段开发方法论，子代理独立执行每个任务 |
| gstack | [garrytan/gstack](https://github.com/garrytan/gstack) | 88k | 23 个专家角色模拟虚拟工程团队，含 CEO/Designer/QA，唯一内置 Playwright 真实浏览器测试 |
| addyosmani/agent-skills | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | ~27k | 19 个 skill 覆盖 SDLC 六阶段，"Process, not prose"，含防跳步机制和证据前置要求 |

#### 功能重叠与冲突矩阵

| | superpowers | gstack | addyosmani/agent-skills |
|---|---|---|---|
| **superpowers** | — | ⚠️ 主要冲突 | ⚠️ 部分重叠 |
| **gstack** | ⚠️ 主要冲突 | — | ⚠️ 部分重叠 |
| **addyosmani/agent-skills** | ⚠️ 部分重叠 | ⚠️ 部分重叠 | — |

> **superpowers vs gstack**：两者都定义完整工作流，安装后均会影响 Claude 的整体开发行为。superpowers 以 TDD 为核心、子代理为执行单元；gstack 以角色扮演为核心、真实浏览器测试为差异点。**建议二选一**，不建议同时激活。
>
> **superpowers / gstack vs addyosmani/agent-skills**：均覆盖全 SDLC，存在阶段重叠。但 agent-skills 模块化程度更高，可按需取其中某个阶段的单个 skill 补强其他框架的缺失环节，**不必视为互斥**。

---

#### superpowers (`obra/superpowers`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/obra/superpowers |
| **安装** | `/plugin install superpowers@claude-plugins-official` |

**原理与作用**

将 Claude 从"被动写代码"变成"遵循结构化工程方法论的高级工程师"。提供 14 个技能（skill），串联成完整的软件开发生命周期。

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

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **andrej-karpathy-skills**（1.2）| ✅ 兼容 | 行为约束层不干扰工作流，建议叠加 |
| **RTK**（1.3）| ✅ 互补 | 建议同时安装，压缩命令输出 |
| **caveman**（1.3）| ⚠️ 部分冲突 | brainstorming/planning 阶段关闭 caveman |

---

#### gstack (`garrytan/gstack`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/garrytan/gstack |
| **安装** | `npx skills add https://github.com/garrytan/gstack` |
| **Star 数** | ~54,000 |

**原理与作用**

Y Combinator CEO Garry Tan 开源的工程师 toolkit，通过 23 个专家角色 + 8 个 Power Tool 将 Claude 变成"虚拟工程师团队"。

```
/plan（CEO：战略规划）→ /design（Designer：UI 设计）→ build
  → /review（Reviewer）→ /qa（QA：Playwright 真实浏览器测试）← 最大差异点
  → /ship → /reflect
```

**主要命令：**

| 命令 | 角色 | 特点 |
|------|------|------|
| `/ceo` / `/plan-ceo-review` | CEO | 挑战 feature scope 和排期 |
| `/design`, `/design-html` | Designer | 生成/审查 UI，专门对抗 AI 设计通病 |
| `/review` | Code Reviewer | 可调用 Codex 获取第二意见 |
| `/qa` | QA Lead | **Playwright 真实浏览器端到端测试** |

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **andrej-karpathy-skills**（1.2）| ✅ 兼容 | 建议叠加，行为约束不影响角色系统 |
| **vercel-labs/agent-skills**（2.1）| ✅ 互补 | 前端规则集 vs. AI 角色，可共存 |
| **awesome-design-md**（2.2）| ✅ 互补 | gstack 提供设计角色，awesome-design-md 提供品牌素材 |

---

#### addyosmani/agent-skills (`addyosmani/agent-skills`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/addyosmani/agent-skills |
| **安装** | `/plugin` → Discover 标签页，或 `cp -r agent-skills/skills/<name>/ ~/.claude/skills/` |
| **Star 数** | ~8,600 |

**原理与作用**

"Process, not prose"——不是告诉 Claude 知识，而是强制走流程（含检查点、退出标准、防跳步反驳条款）。

```
DEFINE → PLAN → BUILD → VERIFY → REVIEW → SHIP
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
- **证据前置**：完成任何步骤必须提供可验证的实物证据
- **渐进加载**：SKILL.md 为入口，支撑文档按需加载，token 消耗低

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **andrej-karpathy-skills**（1.2）| ✅ 兼容 | 理念相近（不跳步、不猜测），建议叠加 |
| **vercel-labs/agent-skills**（2.1）| ✅ 互补 | addyosmani 覆盖完整 SDLC，vercel-labs 专注 React/Next.js 规范 |

---

### 1.2 规范辅助型

不包含端到端完整工作流，核心价值在于纠正 AI 行为偏差或扩展特定通用能力。可叠加在任何工作流框架（1.1）之上使用，不与之冲突。

#### 概览

| 工具 | 仓库 | Stars | 简介 |
|------|------|-------|------|
| andrej-karpathy-skills | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | ~107,000 | 4 条原则纠正 LLM 编码常见失误（过度推测、动手前不思考、改多余的代码） |
| anthropics/skills | [anthropics/skills](https://github.com/anthropics/skills) | ~127,000 | Anthropic 官方 17 个独立能力单元，同时是 SKILL.md 格式规范来源 |

#### 功能重叠与冲突矩阵

| | andrej-karpathy-skills | anthropics/skills |
|---|---|---|
| **andrej-karpathy-skills** | — | ✅ 兼容 |
| **anthropics/skills** | ✅ 兼容 | — |

> 两者关注层面完全不同：andrej-karpathy-skills 约束 Claude 的行为方式，anthropics/skills 扩展 Claude 的具体能力。**无冲突，可同时安装**。

---

#### andrej-karpathy-skills (`forrestchang/andrej-karpathy-skills`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/forrestchang/andrej-karpathy-skills |
| **安装** | `/plugin install andrej-karpathy-skills@karpathy-skills` 或 `curl -o CLAUDE.md <url>` |
| **Star 数** | ~107,000 |

**原理与作用**

由 Andrej Karpathy 一条推文直接催生。不扩展能力，纠正 LLM 编码中最常见的失误模式。

```
1. Think Before Coding  → 显式声明假设，模糊时展示多种解读，不静默决策
2. Simplicity First     → 最少代码，不加推测性抽象，不加未请求的配置项
3. Surgical Changes     → 只动被要求修改的部分，不顺带"改进"周边代码
4. Goal-Driven Execution → 将指令转化为可验证目标，让 Claude 自主循环直到达成
```

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **superpowers / gstack / addyosmani/agent-skills**（1.1）| ✅ 全部兼容 | 建议与 1.1 任一工作流框架叠加安装 |

---

#### anthropics/skills

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/anthropics/skills |
| **安装** | `/plugin install example-skills@anthropic-agent-skills` |
| **Star 数** | ~127,000 |

**原理与作用**

Anthropic 官方 Skill 展示库，每个技能是**独立的能力单元**，没有规定整体开发流程，按需加载。同时是 `SKILL.md` 格式的规范来源（`spec/agent-skills-spec.md`）。

**完整 Skills 列表：**

| 类别 | Skills |
|------|--------|
| 创意与设计 | `algorithmic-art`, `canvas-design`, `frontend-design`, `theme-factory`, `slack-gif-creator` |
| 开发与技术 | `claude-api`, `mcp-builder`, `web-artifacts-builder`, `webapp-testing`, `skill-creator` |
| 企业与沟通 | `brand-guidelines`, `doc-coauthoring`, `internal-comms` |
| 文档（Source-available）| `docx`, `pdf`, `pptx`, `xlsx` |

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **vercel-labs/agent-skills**（2.1）| ⚠️ 部分重叠（`frontend-design`）| vercel-labs 更专注 React/Next.js；anthropics 更通用；可共存 |
| **awesome-design-md**（2.2）| ✅ 互补 | anthropics 偏生成创意内容，awesome-design-md 偏还原品牌风格 |

---

### 1.3 省 Token 型

#### 概览

| 工具 | 仓库 | Stars | 简介 |
|------|------|-------|------|
| RTK | [rtk-ai/rtk](https://github.com/rtk-ai/rtk) | — | Rust 编写的 CLI 代理，通过 PreToolUse hook 过滤 Bash 命令输出，平均节省 60-90% 输入 token |
| caveman | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | — | 改变 Claude 输出风格（穴居人模式），平均节省 65-75% 输出 token |

#### 功能重叠与冲突矩阵

| | RTK | caveman |
|---|---|---|
| **RTK** | — | ✅ 互补 |
| **caveman** | ✅ 互补 | — |

> RTK 作用于**输入侧**（压缩进入上下文的命令输出），caveman 作用于**输出侧**（压缩 Claude 的响应内容）。两者完全不重叠，**建议同时使用**以获得最大节省。

---

#### RTK - Rust Token Killer (`rtk-ai/rtk`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/rtk-ai/rtk |
| **安装** | `brew install rtk` 或 `cargo install --git https://github.com/rtk-ai/rtk` |

**原理与作用**

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

**⚠️ 重要限制：** `PreToolUse` hook 只拦截 Bash 工具调用，`Read`、`Grep`、`Glob` 等原生工具不经过 hook，需显式使用 `rtk read`、`rtk grep`、`rtk find`。

```bash
rtk gain              # 查看累计节省的 token 统计
rtk gain --history    # 查看历史命令的节省记录
rtk discover          # 找到未使用 rtk 的遗漏机会
rtk proxy <cmd>       # 不过滤，直接透传原始命令（调试用）
```

---

#### caveman (`JuliusBrussee/caveman`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/JuliusBrussee/caveman |
| **安装** | `npx skills add JuliusBrussee/caveman` |

**原理与作用**

通过改变 Claude 的输出风格来削减响应 token。去掉客套话、过渡句、重复摘要，只保留代码块和技术要点。

```
激活：/caveman 或 "caveman mode"    退出：stop caveman 或 "normal mode"
```

**三档压缩：**

| 档位 | 效果 |
|------|------|
| Lite | 去填充词，保留自然句子 |
| Full（默认）| 穴居人式简写 |
| Ultra | 电报式极限压缩 |

平均节省：output token **-65~75%**，input token **-45%**，极端 debug 场景最高 **-87%**

**子技能：** `caveman-commit`、`caveman-review`、`caveman-compress`

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **superpowers brainstorming/writing-plans**（1.1）| ⚠️ 部分冲突 | 规划阶段需要详细输出时请关闭 caveman |

---

### 1.4 使用痛点解决型

#### 概览

| 工具 | 仓库 | Stars | 简介 |
|------|------|-------|------|
| code-notify | [mylee04/code-notify](https://github.com/mylee04/code-notify) | ~208 | 任务完成时播放声音 + TTS + OS 原生通知；支持 Claude Code / Codex / Gemini CLI |
| claude-code-audio-hooks | [ChanMeng666/claude-code-audio-hooks](https://github.com/ChanMeng666/claude-code-audio-hooks) | ~53 | 26 种 hook 事件对应不同音效，2 套音效主题，支持 Cursor |
| ClaudeCodeStatusLine | [daniel3303/ClaudeCodeStatusLine](https://github.com/daniel3303/ClaudeCodeStatusLine) | ~459 | 终端底部常驻显示模型名 / token 用量 / 5h·7d rate limit / Git 分支 |
| claude-code-statusline | [rz1989s/claude-code-statusline](https://github.com/rz1989s/claude-code-statusline) | ~439 | 28 个可定制组件，含费用预测、缓存效率、MCP 监控，TOML 配置，Catppuccin 主题 |
| tmux-claude-status | [long-910/tmux-claude-status](https://github.com/long-910/tmux-claude-status) | ~4 | tmux 专用，在状态栏显示 5h/7d 用量百分比或费用 |

#### 功能重叠与冲突矩阵

本类型内两个子类别（音效 / 用量显示）之间无交集，仅在各子类别内部存在功能重叠：

**音效通知类：**

| | code-notify | claude-code-audio-hooks |
|---|---|---|
| **code-notify** | — | ⚠️ 功能重叠 |
| **claude-code-audio-hooks** | ⚠️ 功能重叠 | — |

> 两者功能基本相同，同时安装无实际意义。推荐 `code-notify`（更成熟，Homebrew 安装，覆盖更多 AI 工具）。

**用量显示类：**

| | ClaudeCodeStatusLine | claude-code-statusline | tmux-claude-status |
|---|---|---|---|
| **ClaudeCodeStatusLine** | — | ⚠️ 功能重叠 | ⚠️ 功能重叠 |
| **claude-code-statusline** | ⚠️ 功能重叠 | — | ⚠️ 功能重叠 |
| **tmux-claude-status** | ⚠️ 功能重叠 | ⚠️ 功能重叠 | — |

> 三者均基于 Claude Code `statusCommand` API，功能高度重叠，**选一即可**。推荐 `daniel3303/ClaudeCodeStatusLine`（简洁均衡），追求极致定制选 `rz1989s`，仅用 tmux 选 `long-910`。

---

#### 任务完成音效

**共同工作原理（基于 Claude Code Stop hook）：**

```
Claude 完成任务触发 Stop 事件
  → 执行 hook 命令
  → 播放声音 / 发送 OS 通知 / TTS 语音朗读
```

**DIY 最简方案（macOS，无需安装任何依赖）：**

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

**code-notify 安装：**

```bash
brew install mylee04/tools/code-notify
```

---

#### 用量常驻显示（Statusline）

**共同工作原理（基于 Claude Code `statusCommand` API）：**

```
Claude Code 定时调用 statusCommand 脚本
  → 脚本收到 JSON 载荷（cost / context 用量 / model / rate limit）
  → 脚本格式化输出到终端底部状态栏
```

---

## 二、具体领域型

### 2.1 前端开发

#### 概览

| 工具 | 仓库 | Stars | 简介 |
|------|------|-------|------|
| vercel-labs/agent-skills | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | — | Vercel 官方 React/Next.js 规范集，含 100+ 条 UI 规则、57 条性能规则，每周 133,000 次安装 |

> 本节目前仅一个库，无内部冲突矩阵。跨类型重叠见详细介绍。

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
| Web Design Guidelines | 100+ 条 UI 规则（可访问性、性能、UX）|
| React Best Practices | 57 条性能优化规则（React/Next.js）|
| React Native Guidelines | 16 条规则（性能、架构、平台特性）|
| Vercel Composition Patterns | 复合组件 / context provider 替代 boolean prop 泛滥 |
| Vercel Deploy Claimable | 一键部署到 Vercel |

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **addyosmani/agent-skills**（1.1）| ✅ 互补 | addyosmani 覆盖完整 SDLC，vercel-labs 专注 React/Next.js；前端项目建议同时安装 |
| **anthropics/skills frontend-design**（1.2）| ⚠️ 部分重叠 | vercel-labs 更专注 React/Next.js；anthropics 更通用；可共存 |
| **gstack /design**（1.1）| ✅ 互补 | 规则集 vs. AI 角色，可共存 |

---

### 2.2 UI 设计

#### 概览

| 工具 | 仓库 | Stars | 简介 |
|------|------|-------|------|
| awesome-design-md | [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | — | 68+ 个品牌设计系统的 DESIGN.md 文件集合，AI 读取后可生成像素级还原目标品牌风格的 UI |

> 本节目前仅一个库，无内部冲突矩阵。跨类型重叠见详细介绍。

---

#### awesome-design-md (`VoltAgent/awesome-design-md`)

| 字段 | 内容 |
|------|------|
| **仓库** | https://github.com/VoltAgent/awesome-design-md |
| **安装** | `npx skills add VoltAgent/awesome-design-md` |

**原理与作用**

`DESIGN.md` 是 Google Stitch 于 2026 年 4 月引入的设计系统规范格式，将颜色 token、排版、间距、组件状态等固化为 AI 可读的 Markdown 文件。

```
DESIGN.md（颜色 token + 排版 + 间距 + 组件状态 + 响应式规则）
  → Claude 读取后理解品牌视觉语言
  → 支持 "build me a page like Stripe" 式自然语言命令
```

**跨类型重叠与冲突**

| 对象 | 关系 | 建议 |
|------|------|------|
| **anthropics/skills canvas-design / theme-factory**（1.2）| ✅ 互补 | anthropics 偏生成创意内容，awesome-design-md 偏还原品牌风格 |
| **vercel-labs Web Design Guidelines**（2.1）| ✅ 互补 | vercel-labs 是通用 UI 规则，awesome-design-md 是具体品牌系统 |
| **gstack /design**（1.1）| ✅ 互补 | gstack 提供设计角色，awesome-design-md 提供品牌素材 |

---

## 三、附录：所有工具速查

| 工具 | 仓库 | 类型 | 核心价值 |
|------|------|------|---------|
| superpowers | [obra/superpowers](https://github.com/obra/superpowers) | 完整工作流 | TDD + 子代理工程方法论（14 skills）|
| gstack | [garrytan/gstack](https://github.com/garrytan/gstack) | 完整工作流 | 虚拟工程团队 + Playwright 浏览器测试 |
| addyosmani/agent-skills | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | 完整工作流 | SDLC 六阶段模块化工作流（19 skills）|
| andrej-karpathy-skills | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | 规范辅助 | LLM 编码失误纠正（4 条原则）|
| anthropics/skills | [anthropics/skills](https://github.com/anthropics/skills) | 规范辅助 | 17 个独立能力单元（官方 SKILL.md 规范来源）|
| RTK | [rtk-ai/rtk](https://github.com/rtk-ai/rtk) | 省 Token | 命令输出压缩（60-99%，输入侧）|
| caveman | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | 省 Token | 响应输出压缩（65-87%，输出侧）|
| code-notify | [mylee04/code-notify](https://github.com/mylee04/code-notify) | 痛点解决 | 任务完成音效 + OS 通知 |
| ClaudeCodeStatusLine | [daniel3303/ClaudeCodeStatusLine](https://github.com/daniel3303/ClaudeCodeStatusLine) | 痛点解决 | 用量/费用常驻状态栏（均衡推荐）|
| vercel-labs/agent-skills | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | 前端开发 | React/Next.js 规范 |
| awesome-design-md | [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | UI 设计 | 68+ 品牌设计系统 |

---

*欢迎 PR 补充新工具*
