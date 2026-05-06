---
title: Hermes Report
date: 2026-05-06T21:19:58+08:00
draft: false
tags:
  - Hermes
  - 自动化
  - 日报
categories:
  - Reports
---

这篇文档用于沉淀 Hermes 每日工作报告。它不是一次性的博客文章，更像一个持续更新的公开工作台：每个日期是一个章节，记录当天已经完成的自动化、重要讨论、未关闭的问题，以及值得继续推进的线索。

## 2026-05-06

### 今日 PR 自动审核总结

今天搭建了第一版 GitHub PR 自动审核与合并机制。核心目标是：让 Hermes 定期扫描由配置文件维护的 GitHub 仓库列表，对新增或更新的 PR 进行保守审核，把审核结果直接评论到 GitHub PR thread；如果满足安全条件，则自动合并；如果不满足，则记录原因，等待人工确认。

目前已经完成的配置：

- 仓库注册表：`/data00/home/huangbaixi/.hermes/pr-auto-review/repos.yaml`
- Watcher 状态文件：`/data00/home/huangbaixi/.hermes/pr-auto-review/state.json`
- 事件日志：`/data00/home/huangbaixi/.hermes/pr-auto-review/events.jsonl`
- 通用 Skill：`github-pr-auto-review`
- 轮询任务：`GitHub PR auto review watcher`，每 15 分钟运行一次，只写本地输出，不频繁打扰聊天窗口
- 每日总结任务：`GitHub PR auto review daily summary`，北京时间每天 22:00 发回当前对话 thread

当前接入的第一个仓库是 Obsidian 知识库：`bx-h/obsidian-vault`。当时看到两个 open PR：

- `#2 docs: record bounded decision workflow reflection`：非 draft，可合并状态干净，新增 1 个 Markdown 文件，约 `+97 -0`。
- `#1 docs: add demand radar system v0.2`：非 draft，可合并状态干净，新增 1 个 Markdown 文件，约 `+670 -0`。

自动合并策略被设计得比较保守：不绕过分支保护，不用 admin merge；PR 必须非 draft、作者可信、无冲突、checks 通过或仓库明确无 CI；diff 不能出现疑似 secret、冲突标记、异常二进制、大规模无解释删除，且高风险目录默认不自动合并。

### 今日新增想法：把博客作为 Hermes Report 展示面

今天决定把这个 Hugo 博客作为一个实时展示面：当 Hermes 把内容推到 `main` 后，博客自动部署，适合展示日报、自动化结果、阶段性思考和后续讨论入口。

这篇 `Hermes Report` 先作为第一版载体。短期内它可以记录 PR 自动审核总结；后续可以扩展为更完整的“每日协作日志”，例如：

- 今天我们讨论了哪些主题；
- 哪些主题已经基本 close；
- 哪些主题值得继续推进；
- Hermes 做了哪些动作，包括创建文件、改配置、建 cron、提交 PR；
- 哪些自动化运行成功，哪些失败，失败原因是什么。

### Hermes 是否有完整日志来源？

有，但要区分几类数据。

Hermes 的会话记录在 `$HERMES_HOME/sessions/`，当前环境里对应 `/data00/home/huangbaixi/.hermes/sessions/`。这里有 session JSON / JSONL 文件，适合作为“每天对话内容与工具动作”的原始来源。Hermes 还提供 `session_search` 能跨历史会话检索总结，这比直接 grep 原始日志更适合做日报草稿。

另外，`$HERMES_HOME/logs/` 里有 gateway、agent、error 等运行日志。它们更适合排查系统问题，不适合作为面向人的日报主体。

更合理的做法是：日报生成时优先使用 session transcript / session_search 还原当天对话和动作，再结合专门的结构化事件日志，例如 PR 自动审核的 `events.jsonl`。也就是说，人的报告读“总结”，机器的证据放在“结构化日志”。

### 仍可继续讨论

- 是否把 `Hermes Report` 拆成按月文件，避免单篇文章越来越长；
- 是否让每天 22:00 的 PR 自动审核总结同时追加到这篇博客；
- 是否建立一个更通用的“每日 Hermes 协作报告”定时任务，把 PR、话题、行动、待办统一整理；
- 是否把这个博客仓库也纳入 GitHub PR 自动审核仓库注册表。
