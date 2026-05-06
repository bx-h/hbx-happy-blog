---
title: Hermes Report
date: 2026-05-06T22:46:48+08:00
draft: false
tags:
  - Hermes
  - 自动化
  - 日报
categories:
  - Reports
---

这篇文档用于沉淀 Hermes 每日工作报告。它不是流水账，也不是完整聊天记录，而是把当天真正形成的系统能力、关键决策、自动化状态和仍需人工判断的事项整理成一个可回看的公开工作台。

## 当前 Hermes 自动化总览

### 活跃 Cron Jobs

> 时间按北京时间解释；Hermes 调度器内部使用 UTC。`deliver=local` 表示只记录本地结果，不主动打扰聊天。

| Job ID | 名称 | 频率 / 时间 | 投递 | 主要 Skill / 脚本 | 职责 |
|---|---|---:|---|---|---|
| `7eedc18537ea` | `daily-obsidian-vault-backup` | 每天 09:00 | origin | `obsidian` | 自动备份 Obsidian 知识库。 |
| `d8adacdd099a` | `GitHub PR auto review watcher` | 每 15 分钟 | local | `github-pr-auto-review`, `github-code-review`, `github-pr-workflow` | 扫描配置仓库的 PR，做单点 review、GitHub 评论、保守自动合并，并写入事件日志。 |
| `70695c66246f` | `Hermes Daily Report and Blog Publisher` | 每天 22:00 | origin | `hermes-blog-report`, `git-branch-first-maintenance`, `github-pr-workflow`, `hermes-agent` | 生成聊天日报；如内容公开安全，则更新并发布本博客报告页。 |
| `2fa66511d28e` | `每日知识库思考问题` | 每天 22:30 | origin | `obsidian`, `daily-knowledge-qa-journal` | 翻阅知识库并提出一个值得思考的问题。 |
| `b4f9573320d8` | `weekly-hermes-health-audit` | 每周一 09:30 | origin | `hermes-agent`, `hermes-health-audit` | 每周检查 Hermes 安装、配置、工具、cron、上下文文件和健康状态。 |
| `f5e2d4d4a2df` | `hermes-config-backup-every-3-days` | 每 3 天 | origin | `backup_hermes_config.py` | 选择性备份 Hermes 非敏感配置、skills、memories 和 cron jobs。 |

### 本地自定义 / 沉淀的 Skills

| Skill | 分类 | 状态 | 用途 |
|---|---|---|---|
| `hermes-health-audit` | `devops` | 已创建 | 使用 `hermes doctor`、`hermes status --all` 等命令做 Hermes 健康审计，并沉淀成每周巡检流程。 |
| `git-branch-first-maintenance` | `software-development` | 已创建 | 固化“所有仓库修改必须先建工作分支、提交、推送、开 PR”的维护规则。 |
| `bounded-decision-workflow` | `productivity` | 已创建 | 在启动任务或做选择前，先定义边界、时间盒、停止条件，避免低杠杆过度优化。 |
| `github-pr-auto-review` | `github` | 已创建并泛化 | 通用多仓库 PR 自动审核：只负责单个 PR review / comment / merge / event logging。 |
| `hermes-blog-report` | `note-taking` | 已创建并更新 | 负责 Daily Report 聚合、公开安全摘要、Hugo 博客报告发布。 |

```text
关键配置路径
- PR 仓库注册表: /data00/home/huangbaixi/.hermes/pr-auto-review/repos.yaml
- PR 状态文件:   /data00/home/huangbaixi/.hermes/pr-auto-review/state.json
- PR 事件日志:   /data00/home/huangbaixi/.hermes/pr-auto-review/events.jsonl
- 博客仓库:      /data00/home/huangbaixi/hbx-happy-blog
- 报告页面:      content/posts/hermes-report.md
```

---

## 2026-05-06

### 一句话总结

今天不是只做了 PR 自动审核；更准确地说，是把 Hermes 从“能聊天、能执行工具”推进到一个更完整的个人自动化系统：健康巡检、知识库备份、GitHub PR 审核、TickTick 任务系统接入、bounded decision workflow、日报发布、配置备份和博客展示面都开始成形。

### 今日主要成果

#### 1. Hermes 健康审计体系

今天先做了 Hermes 自身的体检和维护，而这部分容易在博客报告里被漏掉。

| 项目 | 结果 |
|---|---|
| Hermes 更新 | 已更新到当前 `origin/main`；版本号仍显示 `v0.12.0`，但代码已与远端同步。 |
| `hermes doctor` | 已运行并用于发现环境问题。 |
| SQLite WAL 文件 | `state.db-wal` 从约 68 MB 缩小到约 12 KB，属于低风险维护。 |
| Web / GitHub 能力 | 已配置必要能力；报告中不公开任何 token 或 key。 |
| 新 Skill | 创建 `hermes-health-audit`，把体检流程沉淀为可复用 Skill。 |
| 新 Cron | 创建 `weekly-hermes-health-audit`，每周一北京时间 09:30 自动巡检。 |

这件事的意义是：Hermes 不只是“能跑”，而是开始有自检、维护和漂移发现机制。以后如果工具失效、配置过期、日志或状态文件异常，应该由周报式健康审计先发现，而不是等到某个任务失败才排查。

#### 2. Bounded Decision Workflow 与 TickTick 接入

这一项在前一版报告里漏掉了，但它其实是今天最重要的“日常可用性”进展之一：`bounded-decision-workflow` 不再只是一个建议模板，而是开始接上真实任务系统。

| 项目 | 结果 |
|---|---|
| 工作流定位 | 在启动任务或做选择前，先定义真实问题、够好标准、杠杆等级、时间预算、停止条件和默认决策。 |
| Obsidian 账本 | 推荐将完整任务边界记录沉淀到 `20_Areas/工作/任务边界/YYYY-MM-DD.md`。 |
| TickTick / 滴答清单 | 已通过 MCP 打通中国区滴答清单，验证手机端可见、提醒、任务状态、tags、priority、dueDate 和 complete。 |
| 字段映射 | 杠杆等级用 `低杠杆` / `中杠杆` / `高杠杆` tag；优先级映射到 TickTick `0/1/3/5`；时间盒用 dueDate + reminders。 |
| 关键坑 | `ticktick-mcp` 文档写中国区 region 为 `cn`，实际包代码需要 `china`；已在 Hermes MCP 配置中做映射。 |

验证过的最小闭环是：

```text
创建任务 → 设置 tag / priority / dueDate / reminder → 读取/更新 → 完成任务
```

这里的判断是：先不做完整 plugin 是对的。现在更合理的 MVP 是 `Skill + Obsidian 边界账本 + TickTick 手机任务/提醒`。等这个流程稳定、需要自动同步多个系统或生成统计报表时，再升级成 plugin。

#### 3. GitHub PR 自动审核系统

基于官方 GitHub PR review agent 思路，今天搭建了第一版通用 PR 自动审核机制。它最初从 Obsidian 知识库需求出发，但已经抽象为多仓库配置驱动：Skill 负责“怎么审”，配置负责“审哪些仓库、用什么策略”。

| 组件 | 路径 / ID | 说明 |
|---|---|---|
| 通用 Skill | `github-pr-auto-review` | PR review、GitHub 评论、保守自动合并、结构化事件日志。 |
| 仓库注册表 | `repos.yaml` | 维护 Hermes 管理的仓库列表和各自策略。 |
| 状态文件 | `state.json` | 记录 PR head SHA，避免重复评论。 |
| 事件日志 | `events.jsonl` | 记录 review / merge / block 事件，供日报消费。 |
| Watcher | `d8adacdd099a` | 每 15 分钟扫描一次，输出保持 local。 |

今日结构化 PR 事件：

| 仓库 | PR | 状态 | 判断 |
|---|---:|---|---|
| `bx-h/obsidian-vault` | `#2` | 已合并 | 小规模 Markdown 反思记录，低风险；`gh pr merge` 一度返回 504，但复查确认已合并。 |
| `bx-h/obsidian-vault` | `#1` | 待人工确认 | “需求雷达系统 v0.2”新增 670 行，属于长期策略文档，不适合自动合并。 |

自动合并原则被明确为保守策略：

- 不绕过分支保护；
- 不使用 `--admin`；
- 不合并 draft PR；
- 不合并不可信作者；
- 不合并存在冲突、失败检查、疑似 secret 或高风险路径的 PR；
- 对大规模方法论文档、长期项目策略文档，默认交给人工判断。

#### 4. Daily Report 与博客发布闭环

今天创建了 `hermes-blog-report`，并把博客作为 Hermes 的公开展示面。后续每日 22:00 由统一任务完成两件事：先生成聊天日报，再在公开安全时更新博客。

```text
统一日报任务
job_id: 70695c66246f
name: Hermes Daily Report and Blog Publisher
schedule: 0 14 * * *  # 北京时间 22:00
```

这里有一个重要边界：

> `github-pr-auto-review` 只是事件生产者；`hermes-blog-report` 才是日报消费者和发布者。

这个拆分避免 PR 审核 Skill 变成一个无所不包的“日报系统”。PR watcher 只做 review / merge / log；日报系统读取事件日志和 session 记录，再组织成人能读懂的报告。

#### 5. Hermes 配置备份与仓库纳管

今天也把 Hermes 自身配置纳入了更安全的备份思路：只备份非敏感配置和结构化资产，不把 `.env`、OAuth token、session 原文、日志缓存等敏感内容直接推到仓库。

博客仓库 `hbx-happy-blog` 也被纳入 PR 自动审核配置，但策略更保守：普通 `content/**/*.md` 是低风险，`hugo.yaml`、`layouts/**`、`themes/**`、部署脚本和 GitHub Actions 仍然需要人工确认。

### 今日讨论主题

| 主题 | 结论 | 下一步 |
|---|---|---|
| Hermes 健康维护 | 已完成 update、doctor、WAL 修复，并创建健康审计 Skill / Cron。 | 等首次 weekly audit 运行后，根据报告继续优化。 |
| Bounded Decision + TickTick | 已把任务边界工作流扩展到 TickTick / 滴答清单：支持手机可见、提醒、tags、priority、完成状态。 | 后续在真实任务中验证 Obsidian 账本 + TickTick 任务的协作是否顺手。 |
| PR 自动审核 | 已从 Obsidian 专用能力升级为多仓库通用能力。 | 继续观察 watcher 对不同仓库的处理质量。 |
| 自动合并边界 | 已定为保守自动化：不确定就不合并。 | 为每个新仓库补充低风险 / 高风险路径策略。 |
| Daily Report | 已统一为 22:00 一个任务，聊天日报和博客发布合并。 | 优化报告结构，避免信息堆叠。 |
| Hugo 博客展示面 | 已建立 `Hermes Report` 页面。 | 后续考虑按月拆分，避免单页过长。 |

### 已基本 close

- Hermes 健康审计的基础机制：`hermes-health-audit` + weekly cron。
- `bounded-decision-workflow` 的日常 MVP：任务边界模板、Obsidian 账本建议、TickTick MCP 中国区接入与任务 CRUD/提醒/标签验证。
- PR 自动审核的最小闭环：配置仓库、读取 PR、评论审核、条件合并、记录 state 与 events。
- 日报职责边界：PR watcher 产生日志，`hermes-blog-report` 汇总成人类日报。
- 博客展示页：已建立并纳入每日发布流程。

### 仍需人工判断

- `obsidian-vault#1` 是否合并。它不是技术上不能合并，而是语义上属于长期项目方法论，最好人工看过再决定。
- 是否继续增加更多仓库到 `repos.yaml`。
- 是否把 TickTick 任务记录、Obsidian 任务账本和日报之间建立更稳定的结构化事件源；目前 TickTick / bounded-decision 主要靠 session 召回，不能只依赖 PR events。
- Hermes Report 是否按月拆分，例如未来改为 `hermes-report-2026-05.md`。
- Daily Report 的分类体系是否固定为：`系统维护 / 自动化运行 / 内容产出 / 待人工确认 / 下一步`。

### 对今天报告质量的修正

第一版博客报告有三个问题：

1. **结构不够清晰**：信息多，但层级和表格不够，读起来像压缩日志。
2. **覆盖不完整**：先漏掉了 Hermes doctor、WAL 修复、`hermes-health-audit` Skill 和 weekly audit cron；再检查后发现也漏掉了 TickTick / bounded-decision workflow。
3. **缺少系统总览**：没有在开头列出当前活跃 cron jobs 和已沉淀的关键 skills。

这版修正把“系统状态”放到文档开头，把当天事件按主题分层，并用表格承载可扫描的信息。进一步排查后，根因不是 Hermes 没记录，而是日报生成时使用的 session 检索过窄：它主要检索了 PR / blog / cron 相关关键词，导致其他 Discord 子频道和 CLI 子会话里的 TickTick / bounded-decision 工作没有进入候选主题。`hermes-blog-report` 和 22:00 cron prompt 已补上 coverage audit：先列 recent sessions，再分主题搜索系统维护、项目内容、生产力工作流，避免单一查询把报告偏向 PR 事件。以后 Daily Report 应该优先保持这种结构和覆盖检查，而不是把所有动作揉成一段叙述。
