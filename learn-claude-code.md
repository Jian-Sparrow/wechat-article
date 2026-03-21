- [ ] [https://github.com/shareAI-lab/learn-claude-code/blob/main/README.md](https://github.com/shareAI-lab/learn-claude-code/blob/main/README.md)
- [ ] 写第一篇公众号文章，并发表（注意总结sop and skill）

---

今天给大家推荐一个非常硬核的开源项目——**learn-claude-code**。

<font style="color:rgba(0, 0, 0, 0.9);">这个开源项目总共有12节。</font>**<font style="color:rgba(0, 0, 0, 0.9);">每一节只引入一个核心概念，并配套：Python 实现、对应文档。通过学习这个项目，你可以</font>**

从根本上理解 AI 编程 Agent 是怎么工作的。另外还有一个附加好处，就是可以直观的看到Anthropic是如何理解agent的演进方向，并付诸实践的。

---

## 为什么要关注这个项目？

针对这个附加好处做个解释：

现在，大家对于agent的演进方向有不同的见解，比如对于"**<font style="color:#000000;">AI 智能体应该如何与外部工具和服务交互</font>**"这个问题，

一派是 **MCP派**。他们认为：

> 软件世界必须要有标准协议。
>
> 否则工具生态会一团乱。
>
> MCP 就像互联网的 HTTP，一旦统一，生态会爆炸。

另一派是 **CLI派**。他们认为：

> AI 不需要复杂协议。
>
> AI + shell 才是最自然的组合。

而Anthropic的Claude Code这个产品就是坚定的CLI派，通过这个项目，你就能直观看到为什么这么说的。

---

## 课程内容详解

### S01：最小Agent循环 — 万物之始

<font style="color:rgba(0, 0, 0, 0.9);">第一节教你基础 LLM 调用 + 工具调用 bash + 循环执行，构建一个最小 Agent 循环，类似于 ReAct Agent 这个概念。</font>

**核心理念**：一个退出条件控制整个流程。循环持续运行，直到模型不再调用工具。

这是最原始的Agent形态，代码量很少，但已经具备了Agent的核心特征：
- 能调用LLM
- 能执行工具（bash命令）
- 能自主循环直到完成任务

---

### S02：安全机制 — 给Agent装上护栏

在第一节的基础上，第二节引入了**安全机制（Safety）**。

<!-- 这是一张图片，ocr 内容为：S02-4 TOOLS,WITH SAFETY S01-1TOOL,NO SAFETY UPGRADE! WOW! LOOK AT JUST ONE THING EVERYTHING I CAN DO AT A TIME... AND I NOW!AND I'M FEEL VULNERABLE! PROTECTED! EDIT ICON U READ ICON BASH WRITE ICON TERMINAL ICON OUR POWER UP! SANDBOX/SAFE_PATH -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750606067-5a957c93-3b1d-446d-b963-f6b2be8556a1.png)

从S01到S02的升级：
- **S01**：1个工具，无安全机制 → Agent感觉"脆弱"，什么都能做
- **S02**：4个工具（Edit、Read、Bash、Write）+ 安全机制 → Agent感觉"被保护"

**关键概念**：
- **沙盒（Sandbox）**：限制Agent的执行范围
- **安全路径（Safe Path）**：只允许在特定目录下操作

这一节让Agent从"裸奔"变成了"有护栏"的状态，大大降低了误操作风险。

---

### S03：TodoManager — Agent的记忆辅助器

第三节提出了 **TodoManager** 这个概念。

<!-- 这是一张图片，ocr 内容为：S03-5 TOOLS,WITH PLANNING S02-4 TOOLS, NO PLANNING REMINDERS -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750628982-30499683-7347-4a39-a3cc-c43b21ee30c9.png)

**TodoManager本质**：Agent的"记忆辅助器"。

**解决的问题**：
- 之前（S02）：简单分发，Agent容易"忘记"自己在做什么
- 之后（S03）：带状态跟踪，通过 `ROUNDS_SINCE_TODO` 计数器持续追踪

**工作原理**：
1. 强制性的状态更新
2. 确保 Agent 在处理复杂任务时不会迷失方向
3. 向用户清晰地展示工作计划和进度

这就像给Agent配了一个"记事本"，让它能规划任务、追踪进度。

---

### S04：子Agent（Subagent）— 任务分身术

第四节引入了**子Agent机制**，让Agent能够"分身"处理任务。

<!-- 这是一张图片，ocr 内容为：COMPARISON TO S03 CHANGES PARENT-ONLY TASK TOOL SINGLE SHARED CONTEXT -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750644774-344e124d-4b62-43ff-b4e9-e71f04e55cc8.png)

**核心变化**：
- **S03**：5个工具，共享上下文，无子Agent
- **S04**：引入 `RUN_SUBAGENT()` 机制

**关键机制**：
- **父Agent**：负责调度和协调
- **子Agent**：负责执行具体任务
- **上下文传递**：父Agent向子Agent传递必要的上下文
- **结果返回**：子Agent执行完毕后返回摘要给父Agent

这实现了"分而治之"的思路，让复杂任务能被拆解成多个子任务并行处理。

---

### S05：Skills系统 — 能力扩展库

第五节引入了**Skills系统**，让Agent的能力可以灵活扩展。

<!-- 这是一张图片，ocr 内容为：TUTORIAL COMPARISON: S04 VS S05 SKILLS DESCRIPTION LIST -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750658769-976d823c-3049-4130-a2d3-e7231c640db0.png)

**对比S04和S05**：
- **S04**：Task Tool + 静态Prompt，无知识库
- **S05**：Load Skill + Skills Library，双层注入机制

**Skills的优势**：
- **模块化**：每个Skill是一个独立的 `.md` 文件
- **可插拔**：按需加载特定能力
- **可扩展**：用户可以自定义Skills

**工作流程**：
1. Agent识别需要特定能力
2. 通过 `LOAD_SKILL` 加载对应的 `skills/*.skill.md`
3. Skill描述被注入到Prompt中
4. Agent获得执行特定任务的能力

---

### S06：上下文压缩 — 突破Token限制

第六节解决了Agent的"记忆容量"问题。

<!-- 这是一张图片，ocr 内容为：S06:EVOLUTION OF S05 VS S05-5 TOOLS,NO CONTEXT COMPRESSION S06-COMPACT TOOL,3-LAYER COMPRESSION -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750663445-526b3294-6459-494f-bfba-2a8a15b336d2.png)

**核心问题**：随着对话进行，上下文越来越长，最终超出Token限制。

**S06的解决方案**：
- **Compact Tool**：新增压缩工具
- **三层压缩**：Auto-Compact → Micro-Compact → Threshold Trigger
- **自动保存**：压缩结果用占位符替换，原文保存到Transcripts

**压缩策略**：
1. **自动压缩**：达到阈值自动触发
2. **微压缩**：保留关键信息，压缩冗余内容
3. **结果占位符**：用简洁的占位符替代大量对话历史

这让Agent能处理更长时间的任务，不会因为"记忆"满了而卡住。

---

### S07：任务图与依赖关系 — 结构化管理

第七节将任务管理从"平铺列表"升级为"有向图"。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION:S06 VS S07 TASK GRAPH (DISK-BACKED) -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750671602-53716cdf-7a27-4699-99ba-c2a437f1fab1.png)

**关键升级**：
- **S06**：Flat List（内存），无依赖关系，压缩后丢失
- **S07**：Task Graph（磁盘持久化），有依赖关系，压缩后保留

**新增概念**：
- **BlockedBy**：任务A被任务B阻塞
- **Blocks**：任务A阻塞任务B
- **状态持久化**：重启后任务状态不丢失

这让Agent能处理有依赖关系的复杂任务流程，比如"先安装依赖，再运行测试"。

---

### S08：后台任务与并行执行 — 效率倍增

第八节引入了**后台任务**机制，实现真正的并行处理。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION: S07 VS S08 STREAMLINED GRAPH (EFFICIENCY FOCUS) -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750684724-4867edee-5d93-4e99-ab3c-315dfb8f07ad.png)

**核心改进**：
- **Foreground**：前台任务，需要等待完成
- **Background**：后台任务，可以并行执行
- **通知机制**：后台任务完成后通知主Agent

**工具优化**：
- S07：8个工具（全类型）
- S08：6个工具（精简核心 + 通知）

**典型场景**：
- 后台任务1：安装依赖包
- 后台任务2：拉取远程代码
- 前台任务：处理当前文件

多个任务同时进行，效率大幅提升。

---

### S09：多Agent协作 — 团队作战模式

第九节实现了真正的**多Agent协作**，从单打独斗变成团队作战。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION:S08 VS S09 MULTI-AGENT COORDINATOR TASK GRAPH (LEADER+N),LNBOX -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750694653-62f555fb-8fcb-43e3-bb8a-37ad5f19c973.png)

**架构变化**：
- **S08**：单Agent执行
- **S09**：Leader + N个Teammate，并发Agent循环

**关键机制**：
- **Multi-Agent Coordinator**：协调器，负责分配和调度
- **Inbox**：消息收件箱，Agent之间通过消息通信
- **JSONL持久化**：消息和状态持久保存

**协作模式**：
1. Leader接收任务
2. 拆分任务，分配给Teammate
3. Teammate各自独立执行
4. 结果通过Inbox返回给Leader
5. Leader汇总结果

---

### S10：Plan模式 — 用户审批机制

第十节引入了**Plan模式**，让用户能够审核和批准Agent的计划。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION:S09 VS S10 PLAN GATING -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750703944-bb1fc428-673d-4d04-82ca-89d056498149.png)

**核心概念**：
- **Plan Gating**：计划门控，执行前需要审批
- **状态流转**：Submitting → Reviewing → Approved/Rejected

**工作流程**：
1. Agent生成执行计划
2. 展示给用户审核
3. 用户选择批准或拒绝
4. 批准后Agent开始执行

**安全价值**：
- 用户对重要操作有最终决定权
- 避免Agent"自作主张"造成问题
- 提供执行前的"最后一道防线"

---

### S11：轮询与任务看板 — 自主运行模式

第十一节实现了Agent的**自主运行能力**。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION:S10 VS S11 ADVANCED POLLING INBOX TASK KANBAN -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750711038-ec8c8e4a-e823-4c38-8e13-ae4545115c99.png)

**核心升级**：
- **Polling**：轮询机制，Agent主动检查新任务
- **Task Kanban**：任务看板，可视化任务状态
- **Identity Reinjection**：身份重新注入，保持Agent一致性

**自主循环**：
```
[Idle] → [Claim Task] → [Execution] → [Completed] → [Idle/Polling]
```

**新增能力**：
- **自动认领**：Agent自动认领待处理任务
- **空闲轮询**：无任务时进入空闲状态，定期检查
- **看板管理**：任务状态一目了然

Agent开始具备"主动工作"的能力，不再只是被动响应。

---

### S12：Worktree绑定 — 完整工程化

第十二节是最终形态，引入了**Worktree绑定**机制。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION:S11 VS S12 WORKTREE BINDINGS -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750717510-b83b97a6-9f19-435c-8428-d01e07a04e0b.png)

**核心创新**：
- **Worktree绑定**：每个任务可以绑定独立的git worktree
- **隔离执行**：不同任务在不同目录下工作，互不干扰
- **状态恢复**：任务状态 + Worktree索引双重持久化

**工程价值**：
- **并行开发**：多个任务可以同时修改不同分支
- **安全回滚**：出问题可以快速清理worktree
- **事件追踪**：通过 `worktrees/events.jsonl` 记录完整执行日志

**生命周期**：
```
[Task Claim] → [Worktree Binding] → [Execution] → [Keep/Remove]
```

这时的Agent已经是一个成熟的"虚拟工程师"，能够：
- 自主认领和规划任务
- 在隔离环境中安全执行
- 与团队协作
- 接受人类审核
- 持久化状态，支持恢复

---

## 总结

这个项目通过12节课程，循序渐进地展示了如何从零构建一个生产级的AI编程Agent：

| 阶段 | 章节 | 核心能力 |
|------|------|----------|
| 基础 | S01-S02 | Agent循环 + 安全机制 |
| 规划 | S03-S04 | 任务管理 + 子Agent |
| 扩展 | S05-S06 | Skills系统 + 上下文压缩 |
| 协作 | S07-S09 | 任务图 + 后台执行 + 多Agent |
| 自主 | S10-S12 | Plan模式 + 轮询 + Worktree |

如果你想深入理解AI Agent的工作原理，或者想自己实现一个编程助手，这个项目是最好的学习材料。

**项目地址**：https://github.com/shareAI-lab/learn-claude-code

---

> 本文由 AI 辅助创作，转载请注明出处。