今天给大家推荐一个非常硬核的开源项目——**learn-claude-code**。

<font style="color:rgba(0, 0, 0, 0.9);">这个开源项目总共有12节。</font>**<font style="color:rgba(0, 0, 0, 0.9);">每一节只引入一个核心概念，并配套：Python 实现、对应文档。通过学习这个项目，你可以</font>**从根本上理解 AI 编程 Agent 是怎么工作的。

从这个项目中，你能看到claude code的很多思考，比如什么是agent，calude code的产品理念认为：一个循环 加 bash工具 可以构建任意复杂的agent：

> _<font style="color:rgb(89, 99, 110);">"One loop & Bash is all you need"</font>_<font style="color:rgb(89, 99, 110);"> -- one tool + one loop = an agent.</font>
>

比如，calude code是如何理解agent的演进方向，并付诸实践的：

现在，大家对于agent的演进方向有不同的见解，比如对于"**<font style="color:#000000;">AI 智能体应该如何与外部工具和服务交互</font>**"这个问题，

一派是 **MCP派**。他们认为：

> 软件世界必须要有标准协议。
>
> 否则工具生态会一团乱。
>
> MCP 就像互联网的 HTTP，一旦统一，生态会爆炸。
>

另一派是 **CLI派**。他们认为：

> AI 不需要复杂协议。
>
> AI + shell 才是最自然的组合。
>

而Anthropic的calude code这个产品就是坚定的CLI派，通过这个项目，你就能直观看到Anthropic是如何使用CLI的。

---

## 课程内容详解
### S01：最小Agent循环 — 万物之始
<font style="color:rgba(0, 0, 0, 0.9);">第一节教你基础 LLM 调用 + 工具调用 bash + 循环执行，构建一个最小 Agent 循环，类似于 ReAct Agent 这个概念。</font>

**核心理念**：一个退出条件控制整个流程。循环持续运行，直到模型不再调用工具。

这是最原始的Agent形态，代码量很少，但已经具备了Agent的核心特征：

+ 能调用LLM
+ 能执行工具（bash命令）
+ 能自主循环直到完成任务

---

### S02：安全机制 — 给Agent装上护栏
在第一节的基础上，第二节给agent配置了**多个工具**，并引入了**安全机制（Safety）**。

从S01到S02的升级：

+ **S01**：1个工具，无安全机制 → Agent感觉"脆弱"，什么都能做
+ **S02**：4个工具（Edit、Read、Bash、Write）+ 安全机制 → Agent感觉"被保护"

**关键概念**：

+ **沙盒（Sandbox）**：限制Agent的执行范围
+ **安全路径（Safe Path）**：只允许在特定目录下操作

这一节让Agent从"裸奔"变成了"有护栏"的状态，大大降低了误操作风险。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750606067-5a957c93-3b1d-446d-b963-f6b2be8556a1.png)

---

### S03：TodoManager — Agent的记忆辅助器
第三节提出了 **TodoManager** 这个概念。**TodoManager本质**：Agent的"记忆辅助器"。

**解决的问题**：

+ 之前（S02）：简单分发，Agent容易"忘记"自己在做什么
+ 之后（S03）：带状态跟踪，通过 `ROUNDS_SINCE_TODO` 计数器持续追踪

**工作原理**：

1. 强制性的状态更新
2. 确保 Agent 在处理复杂任务时不会迷失方向
3. 向用户清晰地展示工作计划和进度

这就像给Agent配了一个"记事本"，让它能规划任务、追踪进度。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750628982-30499683-7347-4a39-a3cc-c43b21ee30c9.png)

---

### S04：子Agent（Subagent）— 任务分身术
该模式的核心理念是 “通过进程隔离实现上下文隔离” (Process isolation gives context isolation for free) 。

这种模式通过引入 Subagent (子代理) 机制，解决了长对话中常见的“上下文污染”和“Token 浪费”问题。以下是该模式的核心要点：

+ 当父代理 (Parent Agent) 遇到复杂子任务或需要大量文件探索时，它不会直接在当前对话中进行。而是交给子agent，子agent在执行过程中产生的数百行日志、中间搜索结果等冗余信息，都被限制在子代理的生命周期内。
+ 在子agent执行完毕后，它只向父代理返回最终的总结文字，而不返回执行过程中的工具调用细节。父代理的上下文保持干净、简洁，只保留关键结论，从而节省了大量的 Token，并防止模型被无关细节干扰。
+ 在代码中可以看到， CHILD_TOOLS 移除了 task 工具。这意味着子代理不能再派生子代理，从而避免了无限递归和难以调试的调用链。

**关键机制**：

+ **父Agent**：负责调度和协调
+ **子Agent**：负责执行具体任务
+ **上下文传递**：父Agent向子Agent传递必要的上下文
+ **结果返回**：子Agent执行完毕后返回摘要给父Agent

这实现了"分而治之"的思路，让复杂任务能被拆解成多个子任务并行处理。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750644774-344e124d-4b62-43ff-b4e9-e71f04e55cc8.png)

---

### S05：Skills系统 — 能力扩展库
第五节引入了**Skills系统**，让Agent的能力可以灵活扩展。

这个模式的核心理念是 “按需加载，避免臃肿的系统提示” (Don't put everything in the system prompt. Load on demand.) 。

它通过一种巧妙的 两层技能注入 (Two-layer skill injection) 机制，解决了在 System Prompt 中塞入过多指令会导致 Token 成本高昂和上下文混乱的问题。

+ 在程序启动时， SkillLoader 会扫描 skills/ 目录下的所有 SKILL.md 文件，只提取每个技能的 名称和简短描述 （即 SKILL.md 文件中 --- 分隔符内的元数据），这些简短的描述被整合到 SYSTEM 提示中，作为给 AI 的一个“技能菜单”；这一步非常“便宜”，每个技能只占用少量 Token，让 AI 知道“我有哪些可用的高级能”，而不会被细节淹没。
+ 当 AI 认为需要使用某个特定技能来解决问题时（例如，处理 PDF 文件），AI 会调用一个名为 load_skill 的工具，会读取对应 SKILL.md 文件的 完整内容 （即详细的步骤和指令）。这样，只有在需要时，AI 才会接收到完整的、详细的指令。这保证了上下文的清洁，并极大地节省了 Token。

**Skills的优势**：

+ **模块化**：每个Skill是一个独立的 `.md` 文件
+ **可插拔**：按需加载特定能力
+ **可扩展**：用户可以自定义Skills

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750658769-976d823c-3049-4130-a2d3-e7231c640db0.png)

---

### S06：上下文压缩 — 突破Token限制
第六节解决了Agent的"记忆容量"问题。

模式的核心理念是 “策略性遗忘以实现无限工作流” (The agent can forget strategically and keep working forever) 。该模式通过一个 三层压缩流水线 (Three-layer compression pipeline) ，解决了长对话中 Token 堆积导致的上下文溢出、性能下降和成本激增问题。

三层压缩设计十分精妙，包含了：

1. 微压缩 (Layer 1: micro_compact)
+ 触发时机 ：每一轮对话执行前。
+ 操作内容 ： micro_compact 会保留最近的 3 条工具调用结果，而将更早的结果内容替换为占位符（如 [Previous: used read_file] ）。
+ 核心价值 ：静默地清理掉不再需要的中间细节（如长达数千行的文件读取内容），同时保留操作历史的痕迹，确保上下文的高密度。
2. 自动摘要压缩 (Layer 2: auto_compact)
+ 触发时机 ：当估计的 Token 数量超过阈值（如 50,000）时。
+ 操作内容 ： auto_compact 会执行以下操作：
    1. 将完整的对话历史持久化到 .transcripts/ 目录。
    2. 请求 LLM 对当前对话进行高保真摘要（包括已完成的工作、当前状态、关键决策）。
    3. 用这一条摘要消息替换之前的所有历史消息。
+ 核心价值 ：在不丢失关键进度的前提下，瞬间释放 90% 以上的上下文空间，使 Agent 能够“重获新生”继续工作。
3. 手动触发压缩 (Layer 3: compact tool)
+ 触发时机 ：AI 意识到当前上下文过于杂乱，主动调用 compact 工具。
+ 操作内容 ：通过 PARENT_TOOLS 暴露给 AI，其执行逻辑与 auto_compact 相同。
+ 核心价值 ：赋予 AI 自我管理上下文的能力，让它在处理特别复杂的逻辑切换时能主动“整理思绪”。  
总结： 这个模式将上下文管理从“被动溢出”转变为“主动治理”。它建立了一个 “内存 vs 磁盘” 的机制：
+ 内存 (Context Window) ：只保留精简后的摘要和最近的细节。
+ 磁盘 (.transcripts/) ：保留完整的操作日志供人类回溯。

这种设计使得 Agent 能够处理理论上无限长度的任务，这让Agent能处理更长时间的任务，不会因为"记忆"满了而卡住。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750663445-526b3294-6459-494f-bfba-2a8a15b336d2.png)

---

### S07：任务图与依赖关系 — 结构化管理
第七节将任务管理从"平铺列表"升级为"有向图"。模式的核心理念是 “独立于对话之外的持久化状态” (State that survives compression -- because it's outside the conversation) 。

这种模式通过将任务进度和依赖关系存储在磁盘上，解决了复杂项目中 Agent 记忆丢失和逻辑混乱的问题。以下是该模式的核心要点：

+ 所有的任务都以 JSON 文件的形式存储。这样即使对话历史因为太长而被压缩（如 s06 模式）或者 Agent 进程重启，项目的“全局进度”依然保存在磁盘上。Agent 随时可以通过 task_list 找回当前的工作状态。
+ 允许 Agent 规划复杂的、具有先后顺序的多步工作流，而不仅仅是简单的待办清单。
+ 提供了专门的工具（如 task_create , task_update ）来强制 Agent 以结构化的方式思考任务。这迫使 Agent 在开始写代码前先进行“分解”和“建模”，将大问题拆解为互相关联的小任务，从而提高任务执行的成功率。
+ 任务系统将“正在做什么”和“做到了哪里”从易失的对话上下文（Context）中抽离出来，放到了稳定的文件系统（Filesystem）中。核心价值 ：这使得 Agent 能够处理跨越多个对话阶段、涉及多个子任务的超大型项目。

这让Agent能处理有依赖关系的复杂任务流程，比如"先安装依赖，再运行测试"。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750671602-53716cdf-7a27-4699-99ba-c2a437f1fab1.png)

---

### S08：后台任务与并行执行 — 效率倍增
第八节引入了**后台任务**机制，实现真正的并行处理。这个模式的核心理念是 “即发即忘，非阻塞执行” (Fire and forget -- the agent doesn't block while the command runs) 。

该模式通过引入后台线程和通知队列，解决了 Agent 在执行耗时命令（如编译、测试、启动服务器）时必须等待而无法执行其他任务的问题。

这个模式就像是给 Agent 增加了一个“多任务处理器”。

+ 分派任务 ：Agent 可以将耗时的任务“外包”给后台线程，然后立即去做别的事情。
+ 接收报告 ：当后台任务完成后，结果会像一份报告一样被送到 Agent 的“收件箱”（通知队列），Agent 会在方便的时候（下一轮对话开始前）统一处理。

这种“即发即忘”的异步并行机制，极大地提升了 Agent 的工作效率，使其能够同时处理多个任务，更接近人类程序员的工作方式。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750684724-4867edee-5d93-4e99-ab3c-315dfb8f07ad.png)

---

### S09：多Agent协作 — 团队作战模式
第九节实现了真正的**多Agent协作**，从单打独斗变成团队作战。

这个模式的核心理念是 “可以互相交谈的持久化团队成员” (Teammates that can talk to each other) 。它将 Agent 的概念从“临时工”升级为“全职员工”，构建了一个可以并行协作、异步通信的 Agent 团队。

与 Subagent (s04) 模式的根本区别：

+ Subagent (s04) ：一次性的，任务完成后即被销毁，像一个临时顾问。
+ Teammate (s09) ：持久化的，有自己的名字、角色和状态，可以长期存在，在空闲和工作状态间切换，像一个常驻团队成员。

核心机制解析：

+ 团队的结构（成员、角色、状态）被持久化存储在磁盘上。即使程序重启，团队的“组织架构”依然存在。Agent 不再是孤立的，而是属于一个有明确身份和状态的集体。
+ 异步消息收件箱 (Asynchronous Message Inbox)这是该模式最关键的创新。通过 MessageBus ，每个 Agent（如 alice , bob ）都在 .team/inbox/ 目录下拥有一个专属的 .jsonl 文件作为其 收件箱 。通信是 异步 和 解耦 的，这完美模拟了真实世界中的邮件或即时通讯系统，允许 Agent 之间进行非阻塞的、可追溯的交流。
+ 每个创建的 Agent 都在一个独立的后台线程 ( threading.Thread ) 中运行自己的 agent_loop 。这使得多个 Agent 可以真正地 并行工作 。例如，一个“前端工程师” Agent 可以在实现 UI 的同时，另一个“后端工程师” Agent 正在编写 API，而“项目经理” Agent 则在规划下一步任务。

agent_teams 模式构建了一个微型的、分布式的 Agent “公司”。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750694653-62f555fb-8fcb-43e3-bb8a-37ad5f19c973.png)

---

### S10：Plan模式 — 用户审批机制
第十节引入了**Plan模式**，让用户能够审核和批准Agent的计划。这个模式的核心理念是 “为团队协作建立结构化的沟通协议” (Protocols for structured team collaboration) 。

它在 s09的“自由交谈”基础上，引入了一套正式的、基于状态机的 “沟通协议” ，使得 Agent 团队能够执行更复杂、需要共识和协调的动作，比如“共同制定计划”和“安全关闭团队”。

如果说 s09 模式是建立了一个可以自由聊天的“工作群”，那么 s10就是在这个群里引入了 “会议制度” 和 “下班流程” 。

它通过定义明确的、基于请求-响应的协议，将 Agent 团队的协作水平从简单的“任务分配”提升到了“ 集体决策 ”和“ 协同治理 ”的高度。这使得 Agent 团队能够以一种更加规范、可靠和可预测的方式来执行复杂的多阶段任务。

<!-- 这是一张图片，ocr 内容为：TASK MANAGEMENT EVOLUTION:SO9 VS S10 S10 S09 TASK GRAPH ADVANCED STREAMLINED GRAPH (DISK-BACKED,S09) (S10) REQUEST ID TGG #REQ-1234 S09-9 TOOLS 12 TOOLS S09 MACL (+SHUTDOWN_REG/ SEQUENTIAL EXECUTION, 60S RESP +PLAN) SIMPLE EXIT S09 O TASH CTBAKE PLAN PLAN NEW LINKAGE TASK SHUTDOWN [RESPONSE HAND] [REQUEST HAND] HANDSHAKE [APPROVED/REJECTED [GATING (S09) [SUBMITTING/REVIEWING]] PLAN GATING PENDING COMPLETED 2060 DOCUMENT] DOCUMENT] [CONCURRENCY LOOP 1] [COMPLETION] [CONCURRENCY LOOP 2 SURVIVES RESTART, S09 S09-9 TOOLS,FULL GRAPH, SEQUENTIAL S10-12 TOOLS(+NEW SPECIALIZED),F FORMATTED EXECUTION, SIMPLE E LIFECYCLEPLAN GATINGCONCURRENCY,LIN CY,LINKED STATE E EXIT(NATURAL) -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773758861369-225b71aa-adc7-43df-89a4-a8eb40cbb86d.png)

### S11：轮询与任务看板 — 自主运行模式
第十一节实现了Agent的**自主运行能力**。这个模式的核心理念是 “自主寻源与身份持久化” (The agent finds work itself & Identity re-injection) 。它在 s10 团队协议的基础上，进一步提升了 Agent 的 主动性 和 健壮性 ，使其从“听令行事”转变为“主动觅食”。

核心机制解析：

+ 当一个 Teammate 完成手头任务进入 idle 状态时，它不会直接退出。它会启动一个 轮询机制 ，每隔 5 秒检查一次收件箱和任务板。Agent 变成了“自驱动”的。Lead Agent 不需要给每个成员分配每一个细小任务，成员会根据任务板上的待办事项主动分担压力。
+ 在长对话中，为了节省 Token 可能会进行上下文压缩（如 s06 模式）。压缩后，Agent 可能会“忘记”自己是谁、属于哪个团队、担任什么角色。该模式确保 Agent 在长期的、跨越多次压缩的工作流中，始终保持清晰的 自我认知 和 团队归属感 。

如果说 s09 和 s10 构建了一个“听指挥的团队”，那么 s11则是构建了一个 “自组织、自愈合的精英小队” 。

+ 自组织 ：成员会盯着任务板（ .tasks/ ），发现活儿就干，不需要时刻被催促。
+ 自愈合 ：通过身份重注入，即便记忆被压缩或“重置”，Agent 也能瞬间找回自己的定位。

这种模式下的 Agent 已经非常接近于一个 具备主观能动性 的数字员工，能够独立、稳定地在复杂的工程环境中生存并产出价值。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750711038-ec8c8e4a-e823-4c38-8e13-ae4545115c99.png)

---

### S12：Worktree绑定 — 完整工程化
第十二节是最终形态，引入了**Worktree绑定**机制。这个模式的核心理念是 “目录级隔离与并行任务执行” (Isolate by directory, coordinate by task ID) 。

它通过结合 任务系统 (Tasks) 和 Git 工作树 (Worktrees) ，解决了在同一个代码库中同时处理多个冲突任务或进行高风险实验时的干扰问题。

如果说之前的模式（如 s07 ）是给 Agent 增加了“任务列表”，那么 s12_worktree_task_isolation.py 就是给 Agent 增加了 “多个并行的实验室” 。

+ 任务系统 是实验手册，记录了实验目的和进度。
+ Worktree 是一个个独立的无菌实验室。
+ 事件日志 是实验记录仪。

这种模式使得 Agent 能够处理极具挑战性的、多线程的工程任务，例如： 一边修复紧急 Bug，一边进行底层的架构重构 ，而这两者在代码上完全物理隔离，确保了整个系统的稳定性和开发效率。

这时的Agent已经是一个成熟的"虚拟工程师"，能够：

+ 自主认领和规划任务
+ 在隔离环境中安全执行
+ 与团队协作
+ 接受人类审核
+ 持久化状态，支持恢复

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1489077/1773750717510-b83b97a6-9f19-435c-8428-d01e07a04e0b.png)

---

## 总结
这个项目通过12节课程，循序渐进地展示了如何从零构建一个生产级的AI编程Agent：

| 阶段 | 章节 | 核心能力 |
| --- | --- | --- |
| 基础 | S01-S02 | Agent循环 + 安全机制 |
| 规划 | S03-S04 | 任务管理 + 子Agent |
| 扩展 | S05-S06 | Skills系统 + 上下文压缩 |
| 协作 | S07-S09 | 任务图 + 后台执行 + 多Agent |
| 自主 | S10-S12 | Plan模式 + 轮询 + Worktree |


如果你想深入理解AI Agent的工作原理，或者想自己实现一个编程助手，这个项目是最好的学习材料。

**项目地址**：[https://github.com/shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)

