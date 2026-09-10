# Skills 机制拆解与映射

> 状态：审阅稿 v0.1
>
> 目的：解释 Matt Pocock Skills 主流程、本地新增 Skills 及其责任交接，为下一步提炼 AICoding 方法论提供依据。
>
> 本文刻意不提供最终 AICoding 流程，也不提供可直接背诵的通用 Prompt。

## 1. 分析范围与来源边界

### 1.1 本文区分三种表述

- **原文事实**：Skill 或参考文章明确写出的要求。
- **机制判断**：本文根据多个 Skill 的输入、输出和约束做出的归纳，会明确标成“本文判断”。
- **待确认项**：现有材料之间存在张力，或进入 AICoding 场景后必须由我们选择取舍的地方。

这样区分很重要：Skills 不是一篇完整的软件工程理论，其中既有可迁移的推理机制，也有服务于真实仓库、Issue Tracker、并行 Agent 和 GitHub 的运行设施。不能把所有细节都当成 AICoding 必须照搬的步骤。

### 1.2 来源分层

#### A. 上游安装内容

本机锁文件明确把本文涉及的 `ask-matt`、`setup-matt-pocock-skills`、`grill-with-docs`、`grilling`、`domain-modeling`、`codebase-design`、`to-spec`、`to-tickets`、`implement`、`tdd`、`code-review`、`diagnosing-bugs` 指向 GitHub 仓库 [`mattpocock/skills`](https://github.com/mattpocock/skills)。例如锁文件为每个 Skill 记录了 `source`、`sourceUrl`、`skillPath` 与文件夹哈希。[U0]

#### B. Codex 同名镜像

同名文件同时存在于：

- 上游安装目录：`C:\Users\22817\.agents\skills\<skill>`
- Codex 可调用目录：`C:\Users\22817\.codex\skills\<skill>`

本轮对 12 个 `SKILL.md` 以及 7 个直接引用文件进行了 SHA-256 比对，结果为 **19/19 逐字节一致**。因此，本文把 `.agents` 版本作为上游证据源，不把 `.codex` 同名副本误称为“本地改版”。这是一次本地核验结果，而不是上游文档中的声明。

#### C. 本地新增内容

以下三个 Skill 位于 `C:\Users\22817\.codex\skills\`，但不在 `.skill-lock.json` 的 `mattpocock/skills` 安装记录中：

- `plan-product-loop`
- `git-custody`
- `orchestrate-delivery-loop`

因此本文将它们称为**本地新增的组合与治理层**，而不是声称它们直接修改了上游同名 Skill。它们的确会覆盖上游流程的部分默认行为，最典型的是：上游 `implement` 默认提交代码，而本地编排明确规定，在 Git 由独立托管角色负责时，实现任务必须停在 commit 之前。[U8][L3]

#### D. AICoding 参考文章

用户提供的《[AICoding 笔试方法论：从需求拆解到提交复盘](https://github.com/Ceceliawai/Agent/blob/main/docs/notes/aicoding/index.md)》是本次场景参考。它提出“理解需求 → 设计方案 → 分模块实现 → 测试验证 → 整体联调 → 提交复盘”，并强调 P0/P1/P2、始终保留可运行版本，以及按“任务目标 + 已有约束 + 允许修改范围 + 验证方式”组织对 AI 的要求。[R1]

本文只分析它与 Skills 的机制关系，不把它当成 Matt Pocock Skills 的来源。

### 1.3 本轮未覆盖

- 没有分析 `spec-kit`，因为本轮指定的一手材料中没有它的本地实现。
- 没有分析完整的 `wayfinder`、`triage`、`prototype`、`handoff` 等旁路 Skill；只在 `ask-matt` 总图需要时说明它们的位置。
- 没有验证 Matt Pocock GitHub 仓库今天的最新版本是否与本地锁定版本相同。本文分析对象是**本机实际安装并被真实开发流程使用的锁定版本**。
- 没有执行任何 Skill 的动作，没有创建 Issue、提交、PR 或远程仓库。

## 2. 总体技能图谱

### 2.1 上游主流程

`ask-matt` 把大多数工程工作组织为一条从 idea 到 ship 的主线：

```text
一次性仓库配置
setup-matt-pocock-skills
          │
          ▼
需求/方案澄清
grill-with-docs
  ├─ grilling：决策树 + 分轮 frontier 提问
  └─ domain-modeling：术语表 + 少量 ADR
          │
          ├─ 必须运行才能回答的设计问题 → prototype 旁路
          │
          ▼
多会话项目？
  ├─ 否 → implement
  └─ 是 → to-spec → to-tickets → 每个 ticket 独立 implement
                                      │
                                      ├─ tdd：逐个 red → green 切片
                                      ├─ code-review：Standards / Spec 双轴
                                      └─ 上游默认 commit

故障入口：diagnosing-bugs → 可复现反馈回路 → 最小复现 → 假设 → 修复/回归
词汇底座：domain-modeling（业务语言）+ codebase-design（模块、接口、seam）
```

这不是简单的命令列表。`ask-matt` 还规定：从 grilling 到 tickets 应尽量保持一段连续上下文；每个 ticket 的实现则用新上下文，以自包含 ticket 作为新的工作记忆。[U1]

### 2.2 本地增强后的交付链

本地新增 Skills 没有替换 TDD、诊断、评审等实现技术，而是在它们外面增加了三层控制：

```text
产品意图
  │
  ▼
plan-product-loop
  产出：release sentence、capability map、Parent Spec、3–5 个收敛里程碑
  不变量：Build → Effect → Proof；最后一项把所有能力汇入真实产品路径
  │
  ▼（用户批准 Parent / milestones / dependency graph 后）
orchestrate-delivery-loop ＝ 控制平面
  ├─ Supervisor：产品决策、权限、门禁与用户沟通
  ├─ Implementation task：implement + TDD + validation + fixed-point review
  │                         但在独立 Git 托管时停在 commit 前
  └─ Parent Git custodian：git-custody，核对身份/范围/证据后发布
```

**本文判断：**上游链解决“怎样把想法变成经过测试和评审的代码”；本地增强主要解决三类真实交付风险：

1. **交付物被测试替代**：测试绿了，但说不清用户到底获得了什么。
2. **证据与代码树脱钩**：测试、review、部署结果没有绑定到 exact base/head。
3. **权限与角色串位**：实现 Agent 顺手 commit、push、改 Issue 或部署，超出本轮授权。

### 2.3 贯穿所有阶段的四条主线

| 主线 | 从哪里建立 | 向哪里传递 | 核心问题 |
|---|---|---|---|
| 语义线 | grilling / domain-modeling | Spec、ticket、测试名、review | 我们说的是否是同一个东西？ |
| 结构线 | codebase-design / to-spec | ticket、TDD、diagnosis | 公开接口在哪里，应该从哪条 seam 观察行为？ |
| 证据线 | TDD / diagnosis / review | acceptance、Git handoff、CI | 什么事实足以证明这份代码具备目标行为？ |
| 权限线 | 用户授权 / orchestrator | implementation / custody / deployment | 谁可以改代码、提交、发布、部署、清理？ |

## 3. 阶段机制卡

### 3.1 `setup-matt-pocock-skills`：先建立其他 Skill 依赖的仓库协议

| 字段 | 内容 |
|---|---|
| 触发 | 第一次在一个仓库使用整套工程 Skills。 |
| 输入 | Git remote、已有 `AGENTS.md`/`CLAUDE.md`、Issue Tracker、领域文档布局、是否为 monorepo。 |
| 核心动作 | 探索现状；向用户呈现建议；确认 Issue Tracker、triage label、单/多 context；再写仓库约定。 |
| 产物 | `docs/agents/issue-tracker.md`、可选 triage 映射、domain 文档规则，以及 Agent 指令中的入口。 |
| 硬约束 | 它是 prompt-driven，不是确定性脚本；先探索、展示、确认，再写；不能重复追加 Agent skills block；不能在已有 `CLAUDE.md` 时另建 `AGENTS.md`，反之亦然。 |
| 防止的失败 | 后续 Skill 不知道 Issue 应写到哪里、用什么状态词、从哪里读取领域语言；同一仓库形成多套相互冲突的约定。 |

证据见 [U2]。

**本文判断：**这是“流程运行环境初始化”，不是每道 AICoding 题都需要的推理步骤。其可迁移价值是：在开始前查明“权威需求在哪里、验收怎么跑、允许改哪里”；其具体文件、标签和 Tracker 配置属于可裁剪设施。

### 3.2 `grill-with-docs` + `grilling` + `domain-modeling`：把模糊意图变成显式决策

| 字段 | 内容 |
|---|---|
| 触发 | 有一个想法、方案或需求需要在动手前澄清；且当前有工作目录，可以留下持久记录。 |
| 输入 | 用户意图、现有代码、已有领域词汇和 ADR、环境中可自行查到的事实。 |
| 核心动作 | `grill-with-docs` 调用 `grilling` 和 `domain-modeling`；`grilling` 把问题组织成依赖决策树，每轮只问当前 frontier；事实由 Agent 调查，决策交给用户；`domain-modeling` 挑战含糊/冲突术语，用具体场景和代码交叉核验。 |
| 产物 | 共同理解；项目专属术语进入 `CONTEXT.md`；只有难逆转、缺上下文会令人意外、且确有权衡的决定才进入 ADR。 |
| 硬约束 | frontier 未清空前不能假装全部明确；用户确认共同理解前不能直接行动；`CONTEXT.md` 只能是术语表，不能变成 Spec 或实现笔记。 |
| 防止的失败 | Agent 靠猜补需求、把事实问题甩给用户、同一个词在需求和代码中含义不同、实现后才发现关键取舍未做。 |

证据见 [U3][U4]。

**关键机制不是“多问问题”，而是控制提问顺序。**如果 Q2 的答案依赖 Q1，Q2 不应与 Q1 同轮出现；这能减少模型基于未确认前提提前收敛。

### 3.3 `codebase-design`：把测试位置从“文件”提升为“公开 seam”

| 字段 | 内容 |
|---|---|
| 触发 | 需要确定模块形状、公开接口、测试位置，或发现现有代码难以从外部验证。 |
| 输入 | 目标行为、调用者必须知道的接口约束、会变化的依赖、现有模块与测试表面。 |
| 核心动作 | 使用 module、interface、depth、seam、adapter、leverage、locality 的固定词汇；把更多行为放在更小接口后；调用者与测试穿过同一 seam。 |
| 产物 | 对模块责任和接口的明确描述；必要时是 production adapter + test adapter；更集中的测试表面。 |
| 硬约束 | 不为只有一个 adapter 的情况凭空制造 seam；测试不能越过接口窥探内部；依赖应注入而非在内部创建。 |
| 防止的失败 | 为每个内部函数写脆弱测试、接口暴露实现复杂度、为了 mock 而制造抽象、一个变化散落在许多调用者中。 |

证据见 [U5]。

**本文判断：**`test seam` 是整套 Skills 的关键连接件。`to-spec` 先决定 seam，`tdd` 在 seam 上写行为测试，`diagnosing-bugs` 判断是否存在能覆盖真实故障的正确 seam，`code-review` 再检查实现是否守住了这些决策。

### 3.4 `to-spec`：把已有讨论压缩成可实现契约

| 字段 | 内容 |
|---|---|
| 触发 | 讨论已经完成，工作规模需要跨会话或交给后续 Agent。 |
| 输入 | 当前对话、代码库现状、领域词汇、ADR，以及已经确认的产品/技术决策。 |
| 核心动作 | 不再访谈，只做 synthesis；优先复用最高层现有 seam，必要时提出尽可能高的新 seam，并让用户确认；形成问题、方案、用户故事、实现决策、测试决策、非目标。 |
| 产物 | 发布到 Tracker 且可交给 Agent 的 Spec。 |
| 硬约束 | 不继续采访；不用具体文件路径和易过期代码片段代替设计；必须写 Out of Scope；测试只看外部行为。 |
| 防止的失败 | 上下文压缩后只剩“做某功能”的标题；后续 Agent 不知道为什么这样设计、在哪验证、什么明确不做。 |

证据见 [U6]。

**机制边界：**`grilling` 负责生成和确认决策，`to-spec` 负责把已经完成的决策压缩成可移交契约。让 `to-spec` 重新开始提问，会混淆“发现问题”和“固化答案”两个阶段。

### 3.5 `plan-product-loop`：把 Spec/计划强化为可收敛的产品闭环

| 字段 | 内容 |
|---|---|
| 触发 | 规划 MVP、迭代或发布，要求最后出现一个用户可见的端到端结果，而不是层级 backlog。 |
| 输入 | 最新用户决策、仓库说明、领域词汇、ADR、当前产品行为，以及已有 Spec/tickets。 |
| 核心动作 | 建立 Confirmed/Constraints/Deferred/Open 决策账本；画 capability map；写 release sentence 和最短真实演示路径；生成 Parent Spec 与通常 3–5 个纵向里程碑；做 obligation ledger 与 convergence audit。 |
| 产物 | Tracker-ready 的 Parent + milestones 草稿，及其依赖图、终局 system acceptance。 |
| 硬约束 | 每项能力必须明确 **Build → Effect → Proof**；测试、评审、报告只是证据，不能冒充产品结果；最终里程碑必须消费所有能力分支；默认只读，只有精确批准后才发布。 |
| 防止的失败 | 计划被拆成数据库/API/UI 的横向层；ticket 只写“测试通过”而未说明交付能力；多个里程碑各自绿但不能汇成一个真实用户路径；过度重复昂贵验证。 |

证据见 [L1]。

**本文判断：**它不是简单的 `to-spec + to-tickets` 改名。`to-spec` 偏“完整需求与技术/测试决策”，`to-tickets` 偏“可独立验证的 tracer bullet”；`plan-product-loop` 额外检查“这些切片是否共同构成一次产品变化”，并给每份证据指定拥有阶段。

### 3.6 `to-tickets`：用 tracer bullet，而不是文件/技术层拆工作

| 字段 | 内容 |
|---|---|
| 触发 | 已有 Spec、计划或足够清晰的讨论，需要拆成可由新上下文独立完成的工作单元。 |
| 输入 | 完整 Spec/对话、代码库现状、领域词汇、ADR、可选 prototype 结论。 |
| 核心动作 | 每个 ticket 切一条窄但完整的端到端路径；声明真实 blocking edges；先让用户审阅粒度和依赖，再按 blockers-first 发布。 |
| 产物 | 每个 ticket 一个 Issue/文件，包含 What to build、Acceptance、Blocked by，且默认 ready-for-agent。 |
| 硬约束 | 每个 slice 必须跨过完成行为所需的各层，能独立演示或验证，并适合一个新上下文；不能按 schema/API/UI 横向分层；不能修改 parent issue。 |
| 防止的失败 | 第一个 ticket 只做底层、长时间没有可运行结果；多 Agent 依赖隐含；每个层单独通过但组合失败；ticket 大到超出有效上下文。 |

证据见 [U7]。

#### Vertical slice 与 tracer bullet 的含义

- **Vertical slice（纵向切片）**：从真实入口切到可观察结果的完整窄路径，可能同时包含 schema、API、UI 和测试。
- **Tracer bullet（曳光弹）**：先打通一条真实、可运行、可验证的细路径，通过反馈校正后续实现；不是一次写完全部层。
- “纵向”描述穿过哪些责任层，“窄”控制一次交付的行为范围，“完整”要求该范围能独立成立。

上游也承认例外：机械性且爆炸半径很大的 wide refactor 不应硬切成纵向行为，而应用 expand–migrate–contract，保持中间状态尽可能绿色。[U7]

### 3.7 `implement`：一个很薄的编排入口

| 字段 | 内容 |
|---|---|
| 触发 | 用户已经给出明确 Spec 或 ticket。 |
| 输入 | 已确认需求、预先约定的 test seams、当前分支。 |
| 核心动作 | 尽量使用 TDD；经常运行单文件测试和类型检查；最后跑全量测试；完成后调用 code-review。 |
| 产物 | 实现、测试、review 结果；在上游默认契约中还包括当前分支上的 commit。 |
| 硬约束 | 必须沿预先约定 seam 实现和测试；不能只在结尾才验证。 |
| 防止的失败 | 把“实现”理解为一口气生成代码；遗漏持续反馈和结束前 review。 |

证据见 [U8]。

**本文判断：**`implement` 自身只有很少文字，它真正的行为来自被调用 Skill 和上游上下文。它更像一个“组合入口”，而不是足以独立约束弱模型的完整实现 Prompt。这也是未来做 AICoding 压缩时必须显式补入关键不变量的地方。

### 3.8 `tdd`：在一个 seam 上逐个 red → green

| 字段 | 内容 |
|---|---|
| 触发 | 构建具体行为、修复缺陷，或用户要求测试先行。 |
| 输入 | 预先确认的 public seam、一个待实现行为、来自 Spec/ worked example 的独立预期值。 |
| 核心动作 | 写一个能观察外部行为的失败测试；确认 red；只写使它 green 的最小实现；再进入下一条行为。 |
| 产物 | 一组可读作行为规范、能经受内部重构的测试，以及逐步增长的实现。 |
| 硬约束 | red before green；一次一个 seam、一个测试、一个最小实现；不能先写完所有测试再写实现；上游这个版本明确把 refactor 放到 review 阶段，而不放在 red-green 循环中。 |
| 防止的失败 | 测试只证明代码按自身逻辑运行；测试内部调用次数；大量 imagined tests 与真实实现脱节；提前加入未来可能用到的功能。 |

证据见 [U9]。

#### 测试质量约束

1. **行为而非实现**：从 public interface 观察调用者在意的结果。
2. **预期值独立**：不能用与实现相同的算法重新算 expected，否则是 tautological test（同义反复式测试）。
3. **mock 只放在系统边界**：外部 API、时间/随机、必要时数据库或文件系统；不要 mock 自己控制的内部模块。
4. **优先具体 SDK 风格接口**：每个外部操作一个明确函数，比需要在 mock 内部写条件分支的 generic fetcher 更容易验证。

证据见 [U9a][U9b]。

### 3.9 `diagnosing-bugs`：先造诊断反馈回路，再允许提出理论

| 字段 | 内容 |
|---|---|
| 触发 | 难复现 bug、性能回归、间歇性失败，或普通阅读没有快速找到根因。 |
| 输入 | 用户的精确症状、可访问环境、脱敏日志/trace、已知 good/bad 状态。 |
| 核心动作 | 构造并收紧一条能命中该症状的命令；复现并最小化；提出 3–5 个可证伪、带预测的排序假设；一次只改一个变量做 probe；在正确 seam 上先写回归测试，再修复并回跑原始场景。 |
| 产物 | red-capable 的单命令反馈回路、最小复现、假设与排除证据、回归测试、根因说明和清理结果。 |
| 硬约束 | 没有已经运行且能对精确症状变红的命令，就不能进入假设阶段；所有输出先脱敏；没有正确 seam 时必须把“无法可靠锁定回归”本身作为架构发现。 |
| 防止的失败 | 读到可疑代码就猜根因；修复附近的另一个失败；靠大段日志碰运气；只让最小样例绿而不回跑原始症状。 |

证据见 [U10]。

#### Diagnostic feedback loop 与普通“看报错修代码”的区别

普通修复常把一次失败输出当成背景信息；这里把反馈回路本身当作要构建的产品。合格回路必须：

- 对用户的**精确症状**有判别力；
- 快到可以反复运行；
- 确定，或把偶现故障提高到足以调试的复现率；
- 无需临场人工判断即可运行。

因此“测试失败”不自动等于已经有反馈回路；测试若只断言“没崩溃”，就无法证明目标 bug 被修复。

### 3.10 `code-review`：Standards 与 Spec 两条轴不能互相抵消

| 字段 | 内容 |
|---|---|
| 触发 | 需要审阅 branch、PR 或相对固定点的一组进行中修改。 |
| 输入 | 用户指定的 fixed point、`fixed-point...HEAD` diff、commit list、Spec 来源、仓库标准来源。 |
| 核心动作 | 先验证固定点和非空 diff；寻找 Spec 和 Standards；用相互隔离的两个 sub-agent 并行审阅；最后并排汇总，不跨轴重排。 |
| 产物 | `Standards` 与 `Spec` 两份独立报告，各自有 finding 数量和本轴最严重问题。 |
| 硬约束 | fixed point 必须能解析；Spec 缺失必须诚实注明；仓库明确标准覆盖通用 smell baseline；两轴发现不能合并成一个总分。 |
| 防止的失败 | 代码很整洁却实现错需求；功能看似正确却破坏仓库契约；一个维度的“亮点”掩盖另一个维度的阻断问题。 |

证据见 [U11]。

**双轴的本质：**

- `Standards` 问“是否按照这个仓库的方式写”。
- `Spec` 问“是否写了被要求的东西，而且没有越界”。

它们使用相同 diff，但依据不同，不能互相投票抵消。

### 3.11 `git-custody`：把代码实现与 Git 发布分权

| 字段 | 内容 |
|---|---|
| 触发 | 已有实现和验证证据，需要由独立任务负责 commit、push、PR、checks、merge。 |
| 输入 | 物理 worktree、branch、exact base/head 或明确 changed paths、staged/unstaged/untracked 状态、绑定到该树的验证/review 证据、当前授权。 |
| 核心动作 | 冻结 handoff；核对 worktree 所有权、身份、范围与 remote head；复用仍有效的实现证据；只暂存明确路径；发布并验证 base/head/diff/CI/merge 状态。 |
| 产物 | 可审计 custody record：base → commit → remote → PR → merge，以及本地/远程/Tracker/cleanup 状态。 |
| 硬约束 | 不修产品代码、不默认重跑仍有效的实现验证、不 force push；deploy、Issue closure、branch deletion、worktree cleanup 各自需要单独授权。 |
| 防止的失败 | 把无关用户修改带入提交；证据对应旧代码树；实现者顺手扩大 Git/GitHub 权限；发布任务发现代码问题后自行“修一下”。 |

证据见 [L2]。

#### 上游 `implement` commit 与本地 custody 的覆盖关系

这里不是含糊冲突，而是本地编排写出的**显式优先级**：

1. 单独运行上游 `implement` 时，其默认结束动作是 commit。[U8]
2. 当本地交付由独立 Git custodian 承担时，implementation task 必须停在 commit/push/PR/merge 前。
3. `orchestrate-delivery-loop` 明文规定：“用户的 Git-custody boundary 覆盖 `$implement` 的通用 commit 指令”。[L3]
4. 后续 Git 动作由 custodian 按当前授权执行；custodian 不获得修产品代码的权限。

这体现的是**调用方 overlay（覆盖层）可以收紧被调用 Skill 的默认权限，但不能静默扩权**。

### 3.12 `orchestrate-delivery-loop`：监督、实现、托管三角色

| 字段 | 内容 |
|---|---|
| 触发 | 已批准 Parent Spec，需要启动、监控、门禁、交接或推进多个里程碑。 |
| 输入 | Parent/Issue、依赖、non-goals、`origin/main` 与 intended base、任务/worktree 身份、path allowlist、外部动作授权、当前 gate。 |
| 核心动作 | 建立 control ledger；按 Prepare/Start/Monitor/Gate/Advance 操作；用 gate markers 汇报真实状态迁移；把 review finding 退回实现者；把 immutable Git handoff 交给 custodian。 |
| 产物 | 任务调度、证据化 gate 状态、实现 handoff、Git handoff，以及 milestone completion/advance 判断。 |
| 硬约束 | Supervisor 不写产品代码；implementation 不越过 custody 边界；custodian 不修产品代码；marker 只是证据指针而非证据；未完成 required review/acceptance/publication 时不能把 milestone 说成完成。 |
| 防止的失败 | 一个 Agent 同时决定产品、写代码、审批自己并发布；看到“任务运行中”就误报进度；本地、remote、deployed SHA 与 Issue 状态互相不一致。 |

证据见 [L3]。

#### 三角色责任边界

| 角色 | 拥有 | 不拥有 |
|---|---|---|
| Supervisor | 产品语义、用户沟通、scope/permission/gate、下一步调度 | 产品代码实现、擅自发布 |
| Implementation | Issue 范围内代码、测试、验证、fixed-point review、handoff 证据 | 独立 custody 场景下的 commit/push/PR/merge；未授权外部副作用 |
| Git custodian | 身份/范围核验、明确路径提交、push/PR/checks/merge 及状态记录 | 产品代码修复、擅自重做设计、未授权部署/清理 |

**本文判断：**这三个角色是“责任隔离”，不天然要求三个不同模型。在弱模型或笔试环境里，它们可以压缩成同一个 Agent 的三个显式阶段，但阶段之间仍应保留权限和证据边界。是否这样压缩属于下一阶段方法论设计，不在本文直接定案。

## 4. 上游主流程与本地增强对照

| 维度 | Matt Pocock 上游锁定版本 | 本地新增/覆盖 | 机制变化 |
|---|---|---|---|
| 计划起点 | `grill-with-docs` 通过访谈清空决策树 frontier | `plan-product-loop` 先锁 product frame、capability map 和 release sentence | 从“所有决策已明确”增加到“所有能力能收敛为一次产品结果” |
| Spec 形态 | 大量 user stories + implementation/testing decisions + out of scope | Parent 保持产品高度，明确 capability、constraint、system acceptance、delivery graph | 避免 Parent 被文件/类/测试库存淹没 |
| 拆分单位 | `to-tickets`：每个 ticket 是一个可独立验证的 tracer-bullet vertical slice | 通常 3–5 个 vertical milestones，每个写 Build、Effect、Proof，最后汇流 | 保留纵向切片，但强化里程碑的产品意义和终局收敛 |
| 证据语义 | TDD、typecheck、全量 test、review | 明确“test/review/report 是 proof，不是 build/effect”；证据按阶段拥有并在条件不变时复用 | 防止把“运行过检查”误报成完成能力 |
| Review | 相对 fixed point 的 Standards / Spec 双轴 | orchestration 要求 fixed-point review 成为 gate，finding 回实现任务 | 从一次工具调用升级为状态迁移门槛 |
| Git 默认 | `implement` 最后 commit | implementation 可被覆盖为停在 commit 前，交给 `git-custody` | 将实现权限与发布权限分离 |
| 发布前身份 | 上游 `implement` 本身没有完整 custody ledger | 核对 worktree、base/head、path set、remote/CI/merge | 让证据绑定到精确代码树 |
| 任务组织 | 每个 ticket 新上下文，主线靠 ticket 自包含 | Parent 级 supervisor + issue 级 implementation + Parent 级 custodian | 增加跨任务控制平面和长期发布账本 |
| 完成定义 | 实现、测试、review、commit | 产品 effect、所需 acceptance、授权发布、部署 SHA、Tracker 状态必须一致 | “代码完成”不自动等于“里程碑完成” |

### 4.1 没有改变的核心

本地增强没有推翻以下上游机制：

- 领域语言必须一致；
- 在最高、最少的公开 seam 上验证行为；
- ticket 要切纵向的可观察行为；
- 实现用逐个 red-green 切片收敛；
- 难 bug 先构造可复现反馈回路；
- review 分 Standards 与 Spec 两轴。

### 4.2 真正新增的东西

**本文判断：**三个本地 Skill 新增的不是更多“写代码技巧”，而是：

- 产品结果账本：能力是否全部被 build、产生 effect、拥有 proof；
- 证据生命周期：谁运行、绑定哪棵树、什么时候失效、什么时候不应重复；
- 权限与职责账本：谁能实现、谁能发布、谁能改 Tracker/部署/清理；
- 完成状态的一致性：本地、remote、PR、deployed SHA、Issue 不一致时不宣布完成。

## 5. 技能间的关键数据与责任交接

### 5.1 核心产物如何流动

| 上游产物 | 生产者 | 消费者 | 交接时必须保留的内容 |
|---|---|---|---|
| 已确认决策 | grilling | to-spec / plan-product-loop | 用户选择、约束、open/deferred、关键理由 |
| 领域词汇 | domain-modeling | Spec、tickets、tests、review | canonical term 与明确避免的同义词，不混入实现细节 |
| ADR | domain-modeling | planning / implementation / review | 难逆转选择、真实 trade-off、为什么 |
| test seams | codebase-design + to-spec | TDD / diagnosis / review | public interface、可观察行为、依赖 adapter、错误模式 |
| Spec | to-spec | to-tickets / implement / Spec review | problem、solution、decision、testing decision、out of scope |
| capability map / Parent | plan-product-loop | milestones / orchestrator | Build、Effect、约束、终局 integrated scenario |
| ticket / milestone | to-tickets / plan-product-loop | implementation | 独立可见行为、acceptance、blocked by、non-goals |
| red-green evidence | TDD | implementation handoff / review | 哪个测试先红、最小实现、当前相关检查结果 |
| diagnostic loop | diagnosing-bugs | regression test / fix / post-mortem | 一个已运行命令、精确症状、最小复现、正确假设证据 |
| fixed-point review | code-review | orchestrator / custody | fixed point、diff、Standards 与 Spec 分离结果 |
| immutable Git handoff | implementation/orchestrator | git-custody | repo/worktree、base/head、path set、三态、验证、授权 |
| custody record | git-custody | supervisor / user | base→commit→remote→PR→merge、CI、剩余动作和 owner |

### 5.2 最容易在压缩时丢失的交接字段

1. **为什么**：只给 ticket 标题，不给已经确认的约束和 non-goals。
2. **从哪里验证**：只说“补测试”，不保留 pre-agreed public seam。
3. **如何知道完成**：只说“测试通过”，不保留可观察 effect 与 acceptance。
4. **基于哪份代码**：保留测试结论，却丢掉 fixed point/base/head。
5. **谁有权做下一步**：实现完成后默认继续 commit、push 或部署。

### 5.3 内容交接与权限交接必须分开

**本文判断：**一份 handoff 至少同时含两种不同信息：

- **内容状态**：做了什么、没做什么、证据是什么、还有什么风险。
- **授权状态**：接收者这次可以执行哪些外部动作。

前者完整不代表后者自动扩大。比如 implementation handoff 给出了 branch 和全部测试结果，也不等于 custodian 已被授权 merge 或部署。

## 6. 与 AICoding 参考文章的机制映射

| 参考文章阶段 | 最接近的 Skill 机制 | 相似处 | 不应忽略的差异 |
|---|---|---|---|
| 先读 README、代码、运行与验收方式，列 P0/P1/P2 | setup 探索 + grilling 事实调查 + Spec/plan 的 scope lock | 都要求先读题、找约束和风险，不立刻改代码 | `grilling` 假设可向用户确认决策；笔试常只有 README 和测试，无法获得产品方实时答复 |
| 写简短设计文档 | to-spec + codebase-design | 都要求职责、接口、状态、异常和测试策略 | 上游 Spec 刻意避免具体文件路径；参考文章偏模块/文件设计。两者粒度并不完全一致 |
| 按模块逐个开发并开独立会话 | to-tickets + 每 ticket 新 implement 上下文 | 都控制上下文，一次处理有限范围 | 参考文章是“按模块”；`to-tickets` 明确反对按技术层横切，要求每个 ticket 独立打通端到端行为 |
| 同步写测试，检查格式/类型/测试 | implement + TDD | 都强调持续验证而非结尾一次运行 | TDD 更严格：pre-agreed seam、先红后绿、一次一个行为、预期值必须独立 |
| 依赖模块未完成时先用接口或 mock | codebase-design + TDD mocking | 都承认依赖需要可替换边界 | 上游 TDD 禁止 mock 自己控制的内部模块，只允许系统边界 mock；“为了并行先 mock 内部模块”存在直接张力 |
| 测试失败时读完整错误、最小修改、重跑回归 | diagnosing-bugs | 都反对根据测试名猜问题，要求复现与回归 | diagnosis 额外要求“没有 red-capable 单命令就不准提出假设”，并要求最小复现与 3–5 个可证伪假设 |
| 全部完成后整体 Review 和入口联调 | code-review + plan-product-loop terminal acceptance | 都验证组合后能否工作 | code-review 分 Standards/Spec；terminal acceptance 要求真实入口消费全部能力，而不是若干断开的组件检查 |
| 将判分反馈作为下一轮输入 | diagnosing-bugs feedback loop | 都把完整错误、输入输出、复现条件带回下一轮 | 隐藏判分环境通常不可控制，因此反馈回路可能只能基于公开测试和本地构造的边界样例近似 |
| 每阶段说明目标、约束、修改范围、验证方式 | 所有 Skill 的 trigger/input/constraints/output | 都把 Prompt 视为阶段契约而非一句魔法话术 | Skills 还显式记录产物交接、权限、固定点和停止条件；手输 Prompt 必须选择性压缩 |

### 6.1 一个关键分歧：按模块，还是按纵向行为？

参考文章的“按模块开发”有助于降低上下文负担，但如果把数据库、API、UI 分成互相等待的横向模块，早期就没有可演示主链。`to-tickets` 的规则恰好是针对这个失败模式设计的。

**本文机制判断：**“一次只处理一个有限范围”可以保留，但范围的一级单位更适合是**一个窄的可观察行为**；模块/文件是该行为内部的修改边界，不宜天然成为独立里程碑。这个判断是否纳入最终 AICoding 方法论，需要用户审阅确认。

## 7. 真实开发中可裁剪与不可裁剪的机制候选

> 本节只做机制分级，不给出最终笔试执行顺序。

### 7.1 候选分级

| 级别 | 机制 | 裁剪判断 |
|---|---|---|
| 核心不变量 | 先完整读取权威题面、现有代码、测试/验收入口 | 不可删；可以压成一次只读扫描 |
| 核心不变量 | 明确目标、约束、允许修改范围、non-goals | 不可删；没有 Tracker 也应在当前输出中短暂保留 |
| 核心不变量 | 区分环境事实与需要做出的设计决定 | 不可删；无产品方可问时，应标记假设而不是伪装成事实 |
| 核心不变量 | 选择最高可行 public test seam | 不可删；可以只写一句“从 CLI/API/公开函数验证” |
| 核心不变量 | 先打通一条窄而完整的可运行路径 | 不可删；小题可能整题就是一个 slice |
| 核心不变量 | red → minimal green → 下一行为 | 不可删；题目没有测试框架时也应先构造可失败的最小检查 |
| 核心不变量 | 失败必须绑定精确症状和可重复命令 | 不可删；简单失败可跳过完整 3–5 假设仪式，但不能跳过复现 |
| 核心不变量 | 最终同时对照题面与工程约束 | 不可删；双轴可以由同一 Agent 串行检查，但不能合成“总体感觉不错” |
| 条件机制 | 完整 grilling 决策树与多轮 frontier | 需求歧义高且能与用户互动时保留；固定题面可降级为“歧义/假设清单” |
| 条件机制 | `CONTEXT.md` 和 ADR 持久化 | 小题通常可省文件，但统一术语与记录关键不可逆选择的思想仍有价值 |
| 条件机制 | 3–5 个 Parent milestones 与终局收敛审计 | 中大型题保留；很小题不应为了形式硬造多个 ticket |
| 条件机制 | 完整 diagnosis 六阶段 | 顽固、偶现、性能问题保留；普通可直接复现错误可缩短，但仍先复现再修 |
| 工程设施 | Issue Tracker、triage labels、native blockers | 笔试通常可删，用内存/临时 checklist 表达依赖即可 |
| 工程设施 | 每 ticket 新 Agent、sub-agent 并行 review | 弱模型/单会话环境可删；用清晰阶段边界和小上下文替代 |
| 工程设施 | managed worktree、独立 Git custodian、PR/CI/merge ledger | 只在题目要求 Git 或多人协作时保留；普通笔试可省 |
| 工程设施 | exact SHA、remote/deployed/tracker 一致性 | 无远程交付时可省；但“验证结果必须对应当前文件状态”仍不可省 |

### 7.2 裁剪原则候选

**本文判断：**后续简化应遵循“删载体，不删不变量”：

- 可以删除 GitHub Issue，但不能删除任务边界和验收条件。
- 可以删除 ADR 文件，但不能静默改变关键设计决定。
- 可以删除三个独立 Agent，但不能让实现、评审、发布在逻辑上彼此自我授权。
- 可以删除复杂 gate marker，但不能把 partial、red 或未联调状态说成 complete。
- 可以减少测试数量，但不能让测试越过 public seam 或用实现自身重新计算 expected。

这仍只是候选原则；具体压缩到几段 Prompt、如何按难度展开，应在用户审阅本稿后另做。

## 8. 已识别的冲突与待用户确认项

### 8.1 已由本地 Skill 明确解决的冲突

1. **上游 `implement` 默认 commit vs 独立 Git 托管**

   本地 `orchestrate-delivery-loop` 已明确：用户的 custody boundary 覆盖通用 commit 指令。因此真实开发中以当前调用场景的更窄权限为准。[U8][L3]

2. **“测试/报告完成” vs “产品完成”**

   `plan-product-loop` 和 `orchestrate-delivery-loop` 都明确：测试、review、report、marker 是证据，不是交付能力本身；完成还需目标 effect 与必要 acceptance 成立。[L1][L3]

### 8.2 需要在 AICoding 方法论阶段选择的张力

1. **交互式澄清 vs 固定题面**

   `grilling` 把设计决策交给用户；AICoding 常没有可实时回答的产品方。后续需要决定：哪些歧义必须停下，哪些可按最小假设推进并显式记录。

2. **按模块开发 vs tracer-bullet vertical slice**

   参考文章明确建议按模块；`to-tickets` 明确要求不要按层横切。建议审阅重点：是否接受“行为切片为一级单位，模块为切片内部责任边界”。

3. **内部模块 mock vs 仅系统边界 mock**

   参考文章允许依赖模块未完成时先用 mock；TDD 指南明确“不 mock 自己的模块”。后续需统一为 fake/adapter、最小真实实现，还是在强时间压力下允许临时 mock，并规定清理条件。

4. **Spec 的长度与形态**

   上游 `to-spec` 要求极其完整的 user-story 列表；参考文章强调短设计文档；`plan-product-loop` 又主张 Parent 保持产品高度。笔试版需要确定最低字段，而不能直接照抄三者任意一个模板。

5. **TDD 是否包含 refactor**

   常见口号是 red-green-refactor，但本机上游 `tdd` 明文把 refactor 推迟到 review 阶段。后续 Prompt 应忠实采用本地锁定版本，或明确选择更传统循环，不能两种说法混用。

6. **review 是否需要两个独立上下文**

   上游依靠并行 sub-agent 隔离 Standards 与 Spec；弱模型笔试环境可能做不到。可以压缩执行载体，但需要保留两套依据、两次检查和不互相抵消的输出。

7. **中文要求与本地 GitHub 内容规则**

   用户希望本次技术文档和未来 AICoding Prompt 使用全中文。`git-custody` 当前对 GitHub 可见内容的默认是“中文主体 + 末尾英文说明”，而 `docs/` 内部文档默认英文。若后续上传本稿，需要用户决定是否以本项目的显式中文约定覆盖该默认规则。[L2]

8. **优先级分级与纵向切片的关系**

   参考文章用 P0/P1/P2 控制时间；Skills 用 capability、non-goal、blocking edge 和 vertical slice 控制范围。后续需决定 P0/P1/P2 是给“需求能力”分级，还是给“ticket”分级，避免把一个完整行为拆成只有 P0 后端、P1 前端的横向残片。

9. **隐藏测试下的诊断上限**

   `diagnosing-bugs` 理想上要求精确 red-capable loop；判分平台的隐藏测试可能无法本地重现。后续应规定可接受的替代证据，以及不能把猜测说成已诊断根因的边界。

### 8.3 建议用户本轮优先审阅的五个问题

1. 是否认可“删工程仪式，不删推理与验证不变量”作为后续简化方向？
2. 是否认可用“可观察行为的纵向切片”修正参考文章较容易被误解的“按模块开发”？
3. 是否认可把 `plan-product-loop`、`git-custody`、`orchestrate-delivery-loop` 定义为本地组合/治理增强，而非上游 Skill 的直接改写？
4. AICoding 的内部依赖是否坚持不 mock 自有模块，还是允许有清理条件的临时替身？
5. 后续公开仓库文档是否全中文，不附加英文说明？

## 9. 结论

这组 Skills 的价值不在于命令名称，而在于一组可组合的阶段契约：

1. **需求契约**：事实由 Agent 查，决策由人确认；术语和 non-goals 不可静默漂移。
2. **结构契约**：先确认 public seam，再让 Spec、实现、测试和诊断围绕同一观察点工作。
3. **增量契约**：ticket 与 TDD 都使用“窄而完整”的 tracer bullet，持续获得真实反馈。
4. **诊断契约**：先拥有能命中精确症状的反馈回路，再提出、证伪和修复假设。
5. **审查契约**：Standards 与 Spec 是两个不能互相抵消的正确性维度。
6. **交付契约**：Build、Effect、Proof 必须可追踪；证据必须绑定当前代码树。
7. **权限契约**：Supervisor、Implementation、Git custodian 各自拥有不同责任；更窄的用户授权覆盖 Skill 默认动作。

**本文判断：**未来 AICoding 方法论真正需要压缩的，是这些契约的表达成本，而不是删除它们。对简单题，可以把多个角色、文件和 Tracker 压进一次会话；但如果连“目标/范围、public seam、纵向主链、red-green、精确复现、双轴检查”也一起删掉，就不再是同一套方法的简化版，而只是普通的“让 AI 写代码”。

---

## 来源索引

### 参考文章

- **[R1]** 《[AICoding 笔试方法论：从需求拆解到提交复盘](https://github.com/Ceceliawai/Agent/blob/main/docs/notes/aicoding/index.md)》：重点参见“先读题”“按模块开发”“测试通过后再推进”“整体联调”“如何组织与 AI 的协作要求”“时间有限时的执行顺序”。

### Matt Pocock 上游安装证据与本地锁定文件

- **[U0] 来源锁**：`C:\Users\22817\.agents\.skill-lock.json`，L4–L153、L284–L293。各记录指向 `source: mattpocock/skills` 与具体 `skillPath`。
- **[U1] 总路由与阶段边界**：`C:\Users\22817\.agents\skills\ask-matt\SKILL.md`，L11–L69；`C:\Users\22817\.agents\skills\ask-matt\PHASE-BOUNDARIES.md`，L1–L55。
- **[U2] 仓库协议初始化**：`C:\Users\22817\.agents\skills\setup-matt-pocock-skills\SKILL.md`，L9–L116。
- **[U3] 访谈机制**：`C:\Users\22817\.agents\skills\grill-with-docs\SKILL.md`，L1–L7；`C:\Users\22817\.agents\skills\grilling\SKILL.md`，L6–L22。
- **[U4] 领域建模**：`C:\Users\22817\.agents\skills\domain-modeling\SKILL.md`，L8–L74；`C:\Users\22817\.agents\skills\domain-modeling\CONTEXT-FORMAT.md`，L1–L52；`C:\Users\22817\.agents\skills\domain-modeling\ADR-FORMAT.md`，L1–L39。
- **[U5] 深模块与 seam**：`C:\Users\22817\.agents\skills\codebase-design\SKILL.md`，L8–L114；`C:\Users\22817\.agents\skills\codebase-design\DEEPENING.md`，L1–L39。
- **[U6] Spec 合成**：`C:\Users\22817\.agents\skills\to-spec\SKILL.md`，L7–L75。
- **[U7] Tracer-bullet tickets**：`C:\Users\22817\.agents\skills\to-tickets\SKILL.md`，L9–L105。
- **[U8] 实现编排与默认 commit**：`C:\Users\22817\.agents\skills\implement\SKILL.md`，L7–L15。
- **[U9] TDD 主约束**：`C:\Users\22817\.agents\skills\tdd\SKILL.md`，L8–L38。
- **[U9a] 测试质量**：`C:\Users\22817\.agents\skills\tdd\tests.md`，L1–L77。
- **[U9b] Mock 规则**：`C:\Users\22817\.agents\skills\tdd\mocking.md`，L1–L59。
- **[U10] 诊断循环**：`C:\Users\22817\.agents\skills\diagnosing-bugs\SKILL.md`，L18–L140。
- **[U11] 双轴 Review**：`C:\Users\22817\.agents\skills\code-review\SKILL.md`，L6–L87。

### 本地新增 Skills

- **[L1] 产品闭环规划**：`C:\Users\22817\.codex\skills\plan-product-loop\SKILL.md`，L8–L171。
- **[L2] Git 托管**：`C:\Users\22817\.codex\skills\git-custody\SKILL.md`，L8–L58。
- **[L3] 交付编排**：`C:\Users\22817\.codex\skills\orchestrate-delivery-loop\SKILL.md`，L8–L156。
