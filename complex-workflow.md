工作流程使用superpower，发现了一些问题：1.往往在brainstorm时提供的视角不够全面。可以利用gstack的多角色来确定功能实现
  2.superpower执行时间很久的话会导致上下文很长，回答质量较低 。可以使用gsd来进行任务拆分  3.superpower在执行时往往会询问一些问题，需要人工回答。可以利用gstack多角色自动给出全面思考后的回答。

三个框架各自的核心优势：
  - Superpowers — TDD（测试驱动开发）执行，测试先行、并行 Agent、自动验证
  - GStack — 多角色决策，由 CEO、工程经理、设计师、QA 等"角色"投票
  - GSD — 阶段分解，保证每个会话的上下文用量在 50% 以内

  组合方式：
  1. GStack 起草 spec，Superpowers 的头脑风暴技能来细化
  2. GSD 把 spec 拆分成每个阶段都能在 50% 上下文内完成的子任务
  3. 用 RalphLoop 将多个 Claude 无头会话串联起来，按阶段循环执行，可以整夜无人值守地跑
  4. 会话中途如有需要决策的问题，自动委托给 GStack 投票，不打断循环

  本质：把规格说明书（spec）驱动开发推到极致——由 AI 自动写完整个应用，人只需在开始时写好 spec。
   三个方案

  ---
  方案 A：「复杂度网关 Skill」（推荐）

  在每次会话开始时，由一个新的 complexity-gateway skill 评估任务复杂度（根据关键词、涉及文件数量、是否跨模块等）。低复杂度走原有 Superpowers 流程，高复杂度走增强流程：

  用户输入
    ↓
  complexity-gateway（自动判断）
    ├─ 低复杂度 → 原 Superpowers 流程
    └─ 高复杂度 →
         1. GStack 多角色生成视角 → 喂给 brainstorming
         2. writing-plans → 输出 GSD 兼容的 phases
         3. 执行中遇到问题 → GStack 自动投票回答
         4. RalphLoop 无人值守驱动全流程

  优点：入口单一，逻辑清晰，不破坏原有 Superpowers skill
  缺点：需要新建一个 skill，并修改 using-superpowers 的路由逻辑

  ---
  方案 B：「修改现有 Superpowers Skill」

  直接在 brainstorming、writing-plans、subagent-driven-development 三个 skill 内部插入 GStack/GSD 钩子。Brainstorming 阶段自动追加 GStack 多角色视角，writing-plans 输出 GSD phase
  结构，执行阶段遇问题时调用 GStack 投票。

  优点：无需新增 skill，改动集中
  缺点：改动官方 Superpowers 文件会被插件更新覆盖；复杂度判断分散在各 skill 里不统一

  ---
  方案 C：「外部 CLAUDE.md 编排器」

  不碰任何 skill，只在 ~/.claude/CLAUDE.md 里写一段编排规则，告诉 Claude 什么情况下主动调用 GStack/GSD/RalphLoop。完全靠 Claude 自己理解并执行。

  优点：最简单，零代码
  缺点：稳定性最低，依赖 Claude 的自我判断，容易被跳过；RalphLoop 无人值守逻辑无法靠 CLAUDE.md 实现

  ---
  我推荐方案 A，理由：
  - 复杂度判断有统一的入口，可以调优
  - 不修改官方 Superpowers（更新安全）
  - RalphLoop 可以从网关层直接配置，不需要侵入执行层

  ---

  2. Superpowers 已经会自动执行多任务，RalphLoop 存在的意义是什么

    核心区别在于上下文是否重置：

    - Superpowers 的多任务链：在同一个 session 内执行，上下文不断累积。执行 5-6 个任务后，context window 可能已经 60-80%，后续任务的质量下降——这正是你说的问题 2。
    - RalphLoop：每个阶段结束后退出当前 session，启动一个全新 session，新 session 通过读取 git history 和 GSD 的状态文件（ROADMAP.md、STATE.md 等）了解进度，相当于"每个阶段都是满血状态"。

    所以 RalphLoop 解决的正是"长任务上下文膨胀"问题，和你的诉求 2 直接对应。

    ---
    3. 方案 B 为什么没提 RalphLoop

    方案 B 的思路是"在现有 Superpowers skill 内部插入钩子"，本质上还是在同一 session 内运行。RalphLoop 是外部进程驱动多个 session，和"修改 skill 内部"的设计思路不兼容——你没法在一个 skill 里"启动一个新的
     Claude 进程"。所以方案 B 没有提 RalphLoop，它天然解决不了上下文膨胀问题，这也是方案 B 的主要缺陷之一。
