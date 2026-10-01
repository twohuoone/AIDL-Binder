# 李博杰 AI Agent 教学笔记

> 基于公开资料整理：图灵《AI Agent 实战营》、开源书《深入理解 AI Agent：设计原理与工程实践》、以及 01.me 系列文章。  
> 本文是学习路径与知识点提纲，细节请以原文与实验代码为准。

---

## 一、作者与教学体系

**李博杰（Bojie Li）**：Pine AI 联合创始人 / 首席科学家，《图解大模型》《图解 DeepSeek 技术》译者；曾在华为 2012 实验室工作。2025 年在图灵开设《AI Agent 实战营》，并在国科大讲授 AI Agent 实践课。课程讲义经系统整理后开源为《深入理解 AI Agent》。

### 两套互补材料

| 材料 | 形态 | 适合 |
| --- | --- | --- |
| **图灵 AI Agent 实战营** | 9 周直播 + 实战项目 | 按周动手，做出通用 Agent |
| **《深入理解 AI Agent》** | 10 章正文 + 约 109 个实验 | 系统自学、工程深挖、复现实验 |

**核心目标**：不是只跑通 Demo，而是用可解释、可验证、可复用的工程原则设计 Agent，理解每项架构决策的能力边界与取舍。

---

## 二、一句话总纲

```text
Agent = LLM + 上下文 + 工具
```

| 要素 | 角色 | 工程含义 |
| --- | --- | --- |
| **LLM** | 大脑：决策与推理 | 选型、后训练、成本与能力权衡 |
| **上下文** | 眼睛/耳朵：Agent 能“看到”什么 | Prompt、历史、记忆、RAG、文件系统、Skills |
| **工具** | 双手：与世界交互 | Function Call、MCP、代码执行、Computer Use |

贯穿理念：

- **上下文是 Agent 的操作系统**（The context is the agent's operating system）
- **Harness 工程**（模型之外的脚手架）往往是产品竞争力所在
- **文件系统是终极上下文**：大观测结果落盘，上下文只留摘要与路径指针
- **代码是元工具**：能写代码就能创造新工具，通向自我进化

### 从 Chatbot 到 Agent

| | Chatbot | Agent |
| --- | --- | --- |
| 交互 | 问答循环 | 目标 → 规划 → 行动 |
| 能力 | 主要“说” | 既能想也能做 |
| 状态 | 弱状态 | 记忆、工具、经验闭环 |
| 典型技巧 | 提示工程 | 上下文工程 + 工具系统 |

核心循环（ReAct）：**观察 → 思考 → 行动 → 观察**。

生产级 Agent Loop 常见阶段：消息预处理（token/压缩）→ 系统提示 → 流式会话 → 工具检测与执行 → 是否继续循环。

---

## 三、推荐学习路径

### 路径 A：工程师快速落地（约对应书第 1–5 章）

1. 搭一个能联网搜索的最小 Agent（理解 Loop + 工具）
2. 加角色设定与用户长期记忆
3. 做简单 RAG（法律/文档问答）
4. 接入 MCP，做深度调研 Agent
5. 做一个迷你 Coding Agent（文档驱动 + 测试驱动）

### 路径 B：按实战营 9 周走

见下文「四、实战营周课提纲」。每周：**听课约 2h + 动手编程**。

### 路径 C：研究者 / 深挖

重点：**评估（第 7 章）**、**后训练（第 8 章）**、**持续进化（第 9 章）**、**多 Agent（第 10 章）**。

### 落地总项目建议（书中 FAQ）

从头做一个类似 Claude Code / Codex 的 Coding Agent：

- 第 1–5 章 → 可用骨架  
- 第 7、9 章 → 小评估集 + 从失败中改进  
- 第 8 章 → 介入模型本身（可选）  
- 第 6、10 章 → 语音 / Computer Use / 多 Agent  

---

## 四、实战营：9 周提纲

### 总览

| 周 | 主题 | 基础实战 | 进阶课题 |
| --- | --- | --- | --- |
| 1 | Agent 入门 | 联网搜索 Agent | 上下文缺失消融实验 |
| 2 | 上下文设计 | 角色 + 长期记忆 | 记忆整理 / 日记总结报告 |
| 3 | RAG 与知识库 | 法律知识问答 | 文件系统上下文 / 论文贡献分析 |
| 4 | 工具与 MCP | 深度调研 Agent | 从经验学习 / Sub-agent 专家能力 |
| 5 | 编程与代码执行 | 能开发 Agent 的 Agent | 自我进化（为未知问题造工具） |
| 6 | 评估与选型 | 评测集 + LLM-as-Judge | 并行采样 + 顺序修订 |
| 7 | 多模态与实时交互 | 语音电话 + browser-use | 快慢思考 + 智能打断 |
| 8 | 多 Agent 协作 | 边打电话边操作电脑 | Orchestration Agent |
| 9 | 集成与展示 | 通用 Agent 总装 | 四种“从经验学习”方式对比 |

### Week 1 · Agent 入门

**分类**

- **工作流式**：预定义流程，确定性强，适合固定业务自动化  
- **自主式**：动态规划与自我修正，适合开放式任务  

**实战要点**：任务分解 → 定义 `web_search` → 整合搜索结果生成答案。

**进阶实验**：分别去掉 thinking / tool call / tool result，观察失败模式：

- 无 thinking → 黑盒、难调试  
- 无 tool call → 行为不可追踪  
- 无 tool result → 无限重试 / 错误规划  

### Week 2 · 上下文设计

上下文来源：系统提示、工具说明、对话历史（观察与行动交替）、用户长期记忆。

**实战**：角色人设稳定 + 从对话抽取 JSON 记忆并回注 Prompt。

**进阶记忆整理**：去重合并、冲突以新为准、定期总结成更高层用户画像。

### Week 3 · RAG 与知识库

流水线：Chunking → Embedding → 混合检索 → Re-ranking → 带溯源回答。

**实战**：法律条文知识库，答案必须引用法条。

**进阶·文件系统作为终极上下文**

```text
错误：把整页网页/大文件塞进 Context
正确：完整内容写入沙箱文件 → Context 只留摘要+路径 → 按需 read_file
```

论文 Agent：主论文 + 参考文献并行入库，再做跨文档贡献对比。

### Week 4 · 工具调用与 MCP

工具封装方式：本地 Function Calling、HTTP API、Agent-as-Tool。

**MCP（Model Context Protocol）**：标准化工具接口、语言无关、即插即用、权限/沙箱。

**实战**：多 MCP Server 深度调研（规划 → 权威源 → 多工具串联 → 报告）。

**进阶·从经验学习**：成功轨迹入库 → 相似任务检索经验 → 复用“套路”（Prompt + 工具序列）；人物调研等场景用并行 Sub-agent + 歧义消解。

### Week 5 · Coding Agent

三大工程难点：

1. **代码库理解**：语义搜索、引用查询  
2. **可靠修改**：精确 diff / 字符串替换落地  
3. **一致执行环境**：同一终端会话、依赖预置  

**实战**：自然语言需求 → 设计文档 → TODO → 编码与测试 → 可运行应用。

**进阶·自我进化**：现有工具不够 → 搜开源实现 → 封装新工具 → 沙箱验证 → 写入永久工具库。

### Week 6 · 评估与选型

评估维度：智力、知识、幻觉、长文本、指令遵循、工具调用。

**LLM-as-Judge**：用强模型按 Rubric 打分，对比不同模型/是否开启 thinking。

安全护栏：输入过滤、输出过滤、高风险 Human-in-the-loop、token 预算。

**进阶**

- **并行采样**：多实例多角度探索，再选优  
- **顺序修订**：输出 → 自评 → 改进 → 终稿  

### Week 7 · 多模态与实时交互

**语音链路**：VAD → ASR → LLM → TTS；关键是端到端延迟与自然打断。

**Computer Use**：截图理解 UI → 映射点击/输入；可集成 `browser-use` 等框架。

**进阶·快慢思考（Mixture-of-Thoughts）**

- 快路径：低延迟小模型，维持对话流畅  
- 慢路径：强模型做复杂推理与工具调用  

交互管理：打断意图识别、发言权判断（Turn Detection）、沉默管理。

### Week 8 · 多 Agent 协作

单 Agent 局限：上下文膨胀、难并行、长上下文质量下降。

多 Agent 价值：并行、独立专注上下文、子结果摘要压缩、多角度探索。

**实战**：电话 Agent ↔ 电脑 Agent（A2A），边通话边操作网页。

**进阶·Orchestration**：编排 Agent 把专家 Sub-agent 当工具动态启动；允许 Sub-agent 间直连，不全程中转。

### Week 9 · 集成与四种学习方式

把 RAG、工具、语音、Computer Use、多 Agent 装进一个通用 Agent 并展示。

**Agent 从经验学习的四种方式**

| 方式 | 思路 | 优点 | 代价 |
| --- | --- | --- | --- |
| 长上下文 | 依赖模型吃完整历史 | 实现简单、细节全 | 依赖模型能力/成本 |
| RAG/文本经验 | 经验写成自然语言入库 | 可读、可维护、成本可控 | 依赖检索质量 |
| 后训练 SFT/RL | 经验写入权重 | 内化为“直觉”、泛化强 | 数据与算力贵、难即时反馈 |
| 抽象为代码 | 套路固化成工具/Sub-agent | 可靠高效 | 要求编码能力；工具过多难选 |

---

## 五、开源书：10 章对应关系

在线阅读：[bojieli.github.io/ai-agent-book](https://bojieli.github.io/ai-agent-book/)  
仓库：[github.com/bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book)（Apache-2.0）

| 章 | 主题 | 一句话 | 与实战营映射 |
| --- | --- | --- | --- |
| 1 | AI Agent 入门 | 三要素 + Harness | Week 1 |
| 2 | 上下文工程 | KV Cache、Skills、压缩 | Week 2 |
| 3 | 用户记忆和知识库 | 记忆 + RAG + 图谱 | Week 2–3 |
| 4 | 工具 | MCP、感知/执行/协作 | Week 4 |
| 5 | Coding / 通用 Agent | 代码即元能力 | Week 5 |
| 6 | 观察与动作空间扩展 | 异步、语音、GUI、机器人 | Week 7 |
| 7 | Agent 评估 | 基准、指标、显著性 | Week 6 |
| 8 | 模型后训练 | Pretrain / SFT / RL | Week 9 学习方式③ |
| 9 | 持续进化 | 轨迹 → 知识/指令/程序/参数 | Week 4/5/9 进阶 |
| 10 | 多 Agent 协作 | 共享/隔离上下文、Agent 社会 | Week 8 |

> 2.0 版将原“异步交互 + 多模态”重组为第 6 章；评估/后训练/进化依次后移为第 7–9 章。

### 各章关键洞察（提纲）

1. **入门**：竞争力常在 Harness，不只在基座模型。  
2. **上下文**：围绕 KV Cache 友好设计；压缩是不得已；TODO、错误即学习、动态 reminder、并行采样、顺序修订。  
3. **记忆**：双层——常驻结构化 JSON Cards + 按需上下文感知检索；主动服务常是二者涌现结果。  
4. **工具**：通用化设计（代码解释器优于专用计算器）；主动工具发现。  
5. **Coding Agent**：生产级需理解库、可靠改码、稳定沙箱；代码能创造新工具。  
6. **交互**：模态（文本/语音/GUI）× 时序（同步/异步/事件驱动）。  
7. **评估**：独立验证信号优于“再想一遍”；熟悉 Terminal-Bench、SWE-bench、GAIA、OSWorld、τ²-bench 等。  
8. **后训练**：何时 SFT、何时 RL；工具调用内化；样本效率。  
9. **进化**：经验学习、工作流外化为工具、提示/观察蒸馏进参数。  
10. **多 Agent**：每个设计决策都能在单 Agent 三要素里找到对应；信息隔离与共享是一等公民。

### 跑实验（摘要）

```bash
# 按章安装（Python 3.11–3.13）
uv sync --locked --extra ch1
uv run python chapter1/context/main.py
```

前置：能改中等复杂度 Python；用过主流 LLM 与至少一款 Coding Agent；了解 Git / JSON / REST。第 8 章后训练以外，数学要求不高。

学习方法建议：**先想清架构与原则 → 用 Coding Agent 自己实现 → 跑实验验收**；不必逐行精读书中示例代码。

---

## 六、上下文工程速查（高频实践）

来源：[从提示工程到上下文工程](https://01.me/2025/07/context-engineering/)、[Claude Context Engineering 实践](https://01.me/2025/12/context-engineering-from-claude/)

### 设计策略清单

1. **围绕 KV Cache 架构**：稳定的前缀、少改历史，利于缓存命中  
2. **上下文压缩**：接近窗口上限再压；保留问题、约束、计划、下一步  
3. **TODO List**：外部化任务状态，抗遗忘  
4. **错误即学习**：失败轨迹进上下文，避免重复踩坑  
5. **多层安全防护**  
6. **动态 System Reminder**  
7. **并行采样**  
8. **顺序修订**  
9. **极简工具面、最大能力**（少而通用的工具）  
10. **Sub-agent 控上下文**：子任务独立上下文，只回摘要  

### Prompts / MCP / Skills / Subagents

| | Prompts | MCP | Skills | Subagents |
| --- | --- | --- | --- | --- |
| 提供 | 即时指令 | 工具连接 | 程序性知识 | 任务委派 |
| 持久性 | 单次 | 持续连接 | 跨对话 | 跨会话 |
| 最适合 | 快速请求 | 数据/工具访问 | 组织专属流程 | 独立重任务 |

### 用户记忆双层（摘要）

来源：[从记忆到认知](https://01.me/2025/10/user-memory-for-ai-agent/)

- **常驻**：用户 JSON Cards → System Prompt  
- **按需**：上下文感知 RAG + `load_recent_conversations` 等检索工具  
- 评估：Rubric + LLM-as-Judge；能力层次从记事实 → 理解情境 → 跨会话关联 → 主动预见  

---

## 七、资源索引

### 官方 / 一手

| 资源 | 链接 |
| --- | --- |
| 实战营介绍文 | https://01.me/2025/08/ai-agent-bootcamp/ |
| 实战营 Slides PDF | https://01.me/files/ai-agent-camp-slidev/dist/slidev-exported.pdf |
| 开源书仓库 | https://github.com/bojieli/ai-agent-book |
| 在线阅读 | https://bojieli.github.io/ai-agent-book/ |
| 中文 PDF（latest） | https://github.com/bojieli/ai-agent-book/releases/download/latest/AI-Agents-in-Depth-zh-CN.pdf |
| 学习建议（仓库内） | `docs/zh-CN/LEARNING.md` |
| 思考题参考答案 | `book/reference-answers.md` |
| 作者博客 | https://01.me/ |

### 系列文章（建议阅读顺序）

1. [从提示工程到上下文工程](https://01.me/2025/07/context-engineering/)  
2. [AI Agent 实战营：9 周构建通用 Agent](https://01.me/2025/08/ai-agent-bootcamp/)  
3. [从记忆到认知：用户记忆](https://01.me/2025/10/user-memory-for-ai-agent/)  
4. [Claude Context Engineering 秘籍](https://01.me/2025/12/context-engineering-from-claude/)  
5. LLM API 选型与配置（书中常引用）：https://01.me/2025/07/llm-api-setup/  

### 姊妹书

- 《深入理解 AI Infra》：https://github.com/bojieli/ai-infra-book  

### 相关开源项目（课程/书中常提）

- TalkAct（边打电话边操作电脑）：https://github.com/19PINE-AI/TalkAct  
- browser-use、SWE-bench、OSWorld、τ²-bench、Terminal-Bench 等（见书附录克隆脚本）  

---

## 八、自测清单

学完后，你应能口头讲清并动手验证：

- [ ] 用三要素拆解任意 Agent 产品（Claude Code / Cursor / Manus 等）  
- [ ] 写出最小 ReAct 循环，并解释去掉 tool result 会发生什么  
- [ ] 设计 KV Cache 友好的上下文布局  
- [ ] 实现基础 RAG，并说明何时改用“文件系统指针”模式  
- [ ] 用 MCP 或 Function Calling 接一个外部工具  
- [ ] 说明 Coding Agent 的三大工程难点与应对  
- [ ] 搭一个 10+ 条任务的小评估集 + LLM-as-Judge  
- [ ] 说出并行采样 vs 顺序修订的适用场景  
- [ ] 画出 Orchestration + Sub-agent 的上下文隔离图  
- [ ] 对比四种“从经验学习”方式的成本与收益  

---

## 九、整理说明

- 本笔记为公开课程与开源书的**结构化导读**，非原文逐字转载。  
- 书稿与实验以 GitHub `bojieli/ai-agent-book` 最新版为准（当前公开说明为 **2.0** 版、约 **109** 个实验）。  
- 商业实战营视频课件若另有渠道分发，请遵守对应版权与使用条款；开源书正文与实验为 Apache-2.0。  

**建议用法**：把本文当目录与复习提纲，每周挑一个实战；细节跳进对应章节正文与实验仓库亲手跑通。
