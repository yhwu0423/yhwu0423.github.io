---
layout: post
title: "AI 应用与 Agent 开发：5 个月学习与求职计划"
subtitle: "以开源项目为教材，从最小 Agent Loop 到生产级 Coding Agent"
date: 2026-08-20 12:00:00
author: "wyh"
header-style: text
catalog: true
tags:
  - AI
  - Agent
  - LLM
  - RAG
  - 学习计划
---

> 这是一份以项目为导向的 AI Agent 开发 5 个月学习计划，面向目标岗位为 **Agent 开发 / AI 应用开发**（大厂实习或校招）的初学者。适用背景：有 Python 基础、距离集中投递还有约半年时间的在校生（例如研一升研二、大三）。计划的核心方法只有一条：**先看懂标杆开源项目是怎么实现的，再动手写自己的代码**。每个阶段都以真实开源项目为教材——learn-claude-code、Aider、DeepSeek Harness、Claude Code、OpenCode 等——不允许凭想象闭门造车。文中所有开源项目链接与版本状态均在 2026 年 8 月核实。
>
> 默认投入：每周 20～25 小时，共 20 周。如果每周只有 10～15 小时，把计划延长到 7～8 个月即可，但不要删减测试、评估、日志和部署环节——这些恰恰是校招/实习面试中区分"玩过 Demo"和"做过工程"的地方。

## 一、目标与核心路线

五个月后应达成的目标：

1. 不依赖 Agent 框架，直接调用模型 API，实现流式输出、结构化输出和工具调用。
2. 使用 FastAPI、PostgreSQL、Redis 和 Docker 开发、测试并部署 AI 后端。
3. 独立实现一个带检索、引用、权限、评估和反馈闭环的 RAG 系统。
4. 使用 LangGraph 或 OpenAI Agents SDK 实现带状态、工具、重试、人工审批和可观测性的 Agent。
5. 使用 MCP Python SDK（v2）编写 MCP Server 和 Client，并处理认证、超时和权限边界。
6. 能解释架构、模型选择、评估指标、失败案例、成本和延迟，而不只是展示聊天界面。
7. GitHub 上至少有 3 个可运行项目，其中 1 个达到完整求职作品集标准。

核心路线：

```text
跟学 learn-claude-code：从 30 行 Agent Loop 到完整 Harness
  -> Python 与后端工程
  -> 模型 API 原语：Prompt / Structured Output / Tool Calling / Streaming
  -> RAG：数据处理 / 检索 / 重排 / 引用 / 评估
  -> Agent：状态机 / 工具 / 记忆 / 重试 / Human-in-the-loop
  -> MCP：标准化工具与资源接入
  -> 可观测性 / 自动评估 / 安全 / 成本 / 部署
  -> 行业级综合项目与求职材料
```

学习原则，三条都不可省略：

1. **先读开源，再动手写。** 每个阶段先从标杆开源项目看懂实现方式（读源码、跑示例、画调用链），再写自己的代码。想实现什么之前，先回答"开源项目是怎么做的"。
2. **每学一个概念，都要落到代码、测试、指标或文档**；没有可验证产出的"看懂了"不计入完成。
3. **只维护一个主仓库，层层叠加能力**（见第四节），不做一堆互不相干的 Demo。

## 二、技术优先级

### P0：必须掌握

| 技术领域 | 具体内容 | 掌握标准 |
| --- | --- | --- |
| Python 3.11+ | 类型标注、异常、dataclass、Protocol、生成器 | 能写出结构清晰、可测试的业务模块 |
| 工程化 | `uv`、`pyproject.toml`、Ruff、Pyright、pytest、Git | 一条命令完成安装、检查、测试和启动 |
| 异步编程 | `asyncio`、并发限制、超时、取消、重试 | 能并发调用模型和工具且不失控 |
| Web 后端 | HTTP、REST、SSE、WebSocket、FastAPI、Pydantic | 能实现流式聊天和结构化 API |
| 数据库 | PostgreSQL、SQLAlchemy、迁移、事务、索引 | 能保存用户、会话、文档、反馈和任务状态 |
| 缓存与任务 | Redis、幂等、限流、后台任务 | 能处理缓存、会话和异步任务状态 |
| 基础设施 | Linux、Docker、Docker Compose、环境变量 | 能本地和云端复现项目 |
| LLM 原语 | Token、上下文、结构化输出、Tool Calling、Streaming | 不依赖框架完成完整工具调用循环 |
| Agent Harness | Agent Loop、工具调度、任务规划、上下文压缩、权限与隔离 | 能手写最小 Harness 并讲清每一层为什么存在 |
| RAG | 清洗、切分、Embedding、召回、重排、引用、评估 | 能用数据证明检索效果变化 |
| 测试与安全 | 单元/集成测试、Prompt Injection、权限、密钥管理 | 覆盖成功、失败和攻击路径 |
| 算法与数据结构 | LeetCode Hot 100、常见手撕题（贯穿 20 周，每周 3～5 题） | 中等题 30 分钟内 bug-free，笔试不被卡 |
| LLM 原理（讲得清即可） | Transformer、Attention、KV Cache、采样参数、上下文窗口、SFT/RLHF 概念 | 面试能白板讲清推理过程，不要求会训练 |

### P1：Agent 岗位的差异化能力

| 技术 | 学习重点 | 建议项目 |
| --- | --- | --- |
| LangGraph | 显式状态、节点、条件分支、持久化、恢复、人工审批 | 复杂长流程 Agent |
| OpenAI Agents SDK | Agent、Tool、Handoff、Guardrail、Session、Tracing | 快速理解 Agent 抽象 |
| MCP Python SDK v2 | Server、Client、Tool、Resource、Prompt、stdio、Streamable HTTP | 企业工具接入层 |
| Langfuse | Trace、Span、Prompt 版本、Dataset、Evaluation | 调试和评估 AI 应用 |
| LiteLLM | 多模型统一接口、路由、预算、重试、Fallback | 模型网关和成本控制 |
| Qdrant 或 pgvector | Filter、索引、混合检索、元数据权限 | RAG 向量层 |
| Ragas / Promptfoo | RAG 指标、回归测试、模型和安全测试 | 自动评估流水线 |

### P2：主线完成后再学

- Dify、RAGFlow：用于速读式的 AI 平台架构，不作为前两个月的起点。
- Kubernetes、vLLM、GPU 推理优化：转向 AI 基础设施时再学。
- LoRA、量化、微调：有明确数据和业务需求时再学。
- 多 Agent 群舞和自动规划：先证明单 Agent 和确定性工作流不够用。**但注意**：多智能体协作在 2026 年大厂 JD（字节 Aime、腾讯广告、阿里）中出现频率不低，是资深岗的核心方向——本计划的策略不是跳过它，而是在第 12 周后自然会遇到"单 Agent 何时不够用"的问题，届时能讲清 Supervisor/Handoff 的边界和通信成本，比简历上堆"CrewAI"四个字有说服力得多。
- A2A（Agent-to-Agent）协议：部分 JD 已把"MCP/A2A"并列。先把 MCP 学透——两者解决的都是标准化互操作问题，MCP 打通"Agent↔工具"，A2A 打通"Agent↔Agent"，理解了前者，后者面试时能举一反三。

## 三、开源项目地址与优先顺序（2026-08 已核实）

### 入门主线教程：先学这个，再谈其他

#### 1. learn-claude-code（第 1 个月的主教材）

- 地址：[https://github.com/shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)
- 在线阅读：[https://learn.shareai.run/zh/](https://learn.shareai.run/zh/)
- 项目性质：不是教"怎么使用 Claude Code"，而是**从零复刻一个 Claude Code 级别的 Agent Harness**——12 课递进式教程，每课一个独立目录（`s01_agent_loop/` 到 `s12`），包含中/英/日三语 README、一个可独立运行的 `code.py` 和架构示意图。GitHub 上已超过 2 万 star。
- 内容主线：从不到 30 行的最小 Agent Loop（"One loop & Bash is all you need"）出发，逐课叠加工具调用、任务规划、技能（Skill）加载、三层上下文压缩、后台任务、多 Agent 协作和 Worktree 隔离。
- 为什么放在第一位：它把"Agent 为什么能工作"拆成了 12 个可以亲手运行的最小实验。初学者最大的问题不是不会调 API，而是不知道一个真实 Coding Agent 由哪些层构成、每层解决什么问题——这份教程正好回答这个问题，而且全部代码可运行、可修改、可破坏。
- 使用方式：克隆仓库，`pip install -r requirements.txt`，配置 `ANTHROPIC_API_KEY`，逐课运行 `python s01_agent_loop/code.py`；每课做三件事——跑通、改一处行为、写一段笔记。
- 时间：第 1～2 周跟学完全部 12 课，第 2 周后半脱离教程写自己的最小 Agent。

### 第一优先级：必须动手使用并阅读关键源码

#### 2. FastAPI

- 地址：[https://github.com/fastapi/fastapi](https://github.com/fastapi/fastapi)
- 用途：AI 后端、流式接口、依赖注入、认证和 API 文档。
- 重点：路由、依赖注入、生命周期、异常处理、SSE/WebSocket。
- 方式：先读官方示例，再追踪一次请求从路由到参数校验的路径。
- 时间：第 1 周开始，贯穿 5 个月。

#### 3. OpenAI Python SDK

- 地址：[https://github.com/openai/openai-python](https://github.com/openai/openai-python)
- 用途：理解模型调用、流式响应、结构化输出和工具调用原语。
- 重点：请求/响应类型、异步客户端、超时、重试、Streaming、错误类型。
- 原则：先用原生 SDK 写完模型调用循环，再接 Agent 框架。learn-claude-code 的前几课正是这样做的，两者互为印证。

#### 4. OpenAI Agents SDK for Python

- 地址：[https://github.com/openai/openai-agents-python](https://github.com/openai/openai-agents-python)
- 文档：[https://openai.github.io/openai-agents-python/](https://openai.github.io/openai-agents-python/)
- 用途：理解 Agent、Tool、Handoff、Guardrail、Session 和 Tracing。
- 现状核实：MIT 许可证，支持工具、MCP、人工介入、Session 和 Tracing，也支持多种模型提供商。**2026 年 4 月的大版本更新引入了 in-distribution harness 和 Sandbox 能力**（让 Agent 在受控隔离环境中执行代码），且 Python 优先落地——这正是生产级 Agent 的方向，值得重点关注。
- 重点：`src/agents`、`examples`、`tests`，理解运行循环，不只复制 Quickstart。
- 时间：第 9～10 周。

#### 5. LangGraph

- 地址：[https://github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)
- 文档：[https://docs.langchain.com/oss/python/langgraph/](https://docs.langchain.com/oss/python/langgraph/)
- 用途：构建长时间运行、带状态、可中断和可恢复的 Agent 工作流。
- 现状核实：**已发布 1.x 稳定版（当前 1.2.x），承诺 2.0 前无破坏性变更**；核心卖点是 durable state——进程重启后从断点恢复执行。注意 `langgraph.prebuilt` 已废弃，预置 Agent 能力迁移到了 `langchain.agents`，看旧教程时要留意。
- 重点：State、Node、Edge、Conditional Edge、Checkpoint、Persistence、Human-in-the-loop、Subgraph。
- 时间：第 11～13 周，复杂项目主框架会锁定它。

#### 6. LangChain

- 地址：[https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)
- 文档：[https://python.langchain.com/docs/](https://python.langchain.com/docs/)
- 用途：模型、Prompt、输出解析器、Retriever、Tool 和常见供应商的集成层。
- 现状核实：1.x 稳定版，`create_agent` 抽象构建在 LangGraph 运行时之上，新增了中间件系统（human-in-the-loop、摘要、PII 脱敏）。
- 学习边界：只学能帮助当前项目交付的 20%，不要把"会调用 LangChain API"当成 Agent 能力。
- 顺序：在原生 SDK 写完一个最小 Agent 后跟进，先理解 LangChain，再用 LangGraph 管理复杂状态。

#### 7. MCP Python SDK v2

- 地址：[https://github.com/modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk)
- 文档：[https://py.sdk.modelcontextprotocol.io/](https://py.sdk.modelcontextprotocol.io/)
- 用途：将数据库、文件和内部 API 以标准方式暴露给 AI 应用。
- 现状核实：**v2.0.0 已是稳定版，`pip install mcp` 默认安装 2.x**。v2 对应 2026-07-28 协议版本，最大变化是协议核心改为无状态请求/响应模型（不再需要粘性会话，可以直接负载均衡），OpenTelemetry 追踪默认开启。旧教程大多基于 v1，学习时务必核对版本；如果维护 v1 项目，需固定 `mcp>=1.28,<2`。
- 重点：Server/Client、Tool、Resource、Prompt、stdio、Streamable HTTP、结构化结果、错误处理。
- 时间：第 14 周。

### 第二优先级：项目工程化时接入

#### 8. Qdrant 与 qdrant-client

- 地址：[https://github.com/qdrant/qdrant](https://github.com/qdrant/qdrant) / [https://github.com/qdrant/qdrant-client](https://github.com/qdrant/qdrant-client)
- 用途：向量检索、过滤、混合检索和 RAG 元数据权限。
- 重点：Collection、Payload、Filter、HNSW、稠密/稀疏检索、Top-K。

#### 9. pgvector

- 地址：[https://github.com/pgvector/pgvector](https://github.com/pgvector/pgvector)
- 用途：在 PostgreSQL 内存储和检索向量，降低小项目基础设施复杂度。
- 重点：向量类型、距离函数、HNSW/IVFFlat 索引、SQL Filter。
- 原则：Qdrant 和 pgvector 选一个深入，不要同时浅学两个。

#### 10. Langfuse

- 地址：[https://github.com/langfuse/langfuse](https://github.com/langfuse/langfuse)
- 文档：[https://langfuse.com/docs](https://langfuse.com/docs)
- 用途：记录模型调用、检索、Agent 动作、成本、延迟和评估结果。
- 重点：Trace/Span、Session、Prompt Management、Dataset、LLM-as-a-judge、自托管。
- 要求：最终项目能从失败回答追踪到检索结果、Prompt、模型调用和工具返回值。

#### 11. LiteLLM

- 地址：[https://github.com/BerriAI/litellm](https://github.com/BerriAI/litellm)
- 文档：[https://docs.litellm.ai/](https://docs.litellm.ai/)
- 用途：统一模型接口、模型路由、Fallback、预算和调用日志。
- 重点：Proxy、Router、重试、Fallback、并发限制、预算和密钥隔离。

#### 12. Ragas

- 地址：[https://github.com/explodinggradients/ragas](https://github.com/explodinggradients/ragas)
- 用途：构建 RAG 测试集并评估检索、忠实性和回答相关性。
- 注意：LLM-as-a-judge 结果必须人工抽样校验，不能直接当真值。

#### 13. Promptfoo

- 地址：[https://github.com/promptfoo/promptfoo](https://github.com/promptfoo/promptfoo)
- 用途：Prompt/模型回归测试、批量对比、红队和安全测试。
- 重点：Test Case、Assertion、Provider、CI 集成、Prompt Injection 测试。

### 第三优先级：第 5 个月带着问题速读

#### 14. RAGFlow

- 地址：[https://github.com/infiniflow/ragflow](https://github.com/infiniflow/ragflow)
- 速读：文档抽取、任务队列、检索链路、服务拆分和部署。
- 不建议：第一个月只会部署和点击界面，这不能替代 RAG 原理和代码实现。

#### 15. Dify

- 地址：[https://github.com/langgenius/dify](https://github.com/langgenius/dify)
- 速读：API、Worker、模型网关、Workflow、插件和权限模块。
- 目标：画出核心服务关系，说明自己的项目与 Dify 的范围区别和取舍。

#### 16. OpenTelemetry Python

- 地址：[https://github.com/open-telemetry/opentelemetry-python](https://github.com/open-telemetry/opentelemetry-python)
- 用途：理解标准 Trace、Metric 和 Log。MCP SDK v2 和 Langfuse 都已对接 OTel，这是通用可观测性的公共语言。

### 用于对照学习的 Coding Agent 项目

**Aider**（[https://github.com/Aider-AI/aider](https://github.com/Aider-AI/aider)）：约 4 万 star 的开源 CLI 结对编程工具，Git 集成和最小终端交互做得极好，是理解"读代码→改代码→跑测试→继续修"闭环的最佳入门样本。

**DeepSeek Harness**（[https://github.com/deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)）：2026 年 8 月 13 日发布，MIT 许可证，发布两天 star 数逼近 9 万、登顶 Hacker News。核心理念是"模型（大脑）+ Harness（身体）= Agent"，且"Everything is a Plugin"——模型适配器、工具注册表、会话日志、沙箱乃至 Agent Loop 本身都是插件，底层基于 Cordis 插件内核；四种运行模式（标准 / PTC / 极简 / 创造）中的极简模式只有 Bash 和文件编辑两个工具，恰好印证 learn-claude-code 第一课"一个 Loop 加 Bash 就够"的论断。注意两点：它是 **TypeScript 技术栈**（CLI 命令为 `dsh`），且目前是开发者预览版、官方明确会有破坏性变更——所以学它的架构和插件设计思想，不要绑定具体 API。

**OpenCode**（[https://github.com/anomalyco/opencode](https://github.com/anomalyco/opencode)）：超过 16 万 star 的终端 AI 编程 Agent，MIT 许可证，由 SST 团队（现更名 Anomaly）维护，原 `sst/opencode` 仓库会自动跳转。重点学它的 Provider 抽象（75+ 模型提供商）、Build/Plan 双模式（写权限隔离）和 LSP 集成（把编译器诊断反馈给模型）。

**OpenHands**（[https://github.com/All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands)）：生产级 Coding Agent 的代表，2026 年 4 月起从 V0 架构迁移到基于 [software-agent-sdk](https://github.com/OpenHands/software-agent-sdk) 的 V1。用于第 4 个月对照学习 Sandbox、Runtime 和任务状态设计，读 V1 SDK 即可，不要碰 V0 遗留代码。

**Claude Code**（[https://github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)）：学习插件、Command、Skill 和 Hook 如何包装 Agent 能力，不作为核心运行时源码依赖。配合 learn-claude-code 使用效果最佳：教程复刻它的内部机制，官方仓库展示它的外部扩展面。

## 四、项目主导的五个月路线（按此执行）

这一节是实际执行顺序。上面的技术拆解作为"遇到问题补什么"的索引，**不需要先学完再开始项目**。

### 4.1 只维护一个主仓库，不做孤立 Demo

整个五个月只维护一个主仓库（例如 `agent-lab`）。每个阶段在同一仓库上增加能力，并打 Tag 保存里程碑：

```text
v0.1 最小工具循环
  ↓ 保留：模型适配器、工具协议、命令行入口、测试夹具
v0.2 插件与权限系统
  ↓ 保留：工具注册接口；新增 Plugin/Policy 层
v0.3 状态化 Agent
  ↓ 保留：同一批工具；新增 State、Checkpoint、Approval
v0.4 MCP 工具层
  ↓ 保留：同一批工具；增加 MCP Server/Client 适配器
v0.5 代码库理解与 RAG
  ↓ 保留：Agent Loop 和 MCP；增加 Repo Index、检索、引用
v1.0 生产化 Coding Agent
  ↓ 保留：所有业务能力；增加 Sandbox、Trace、评估、模型路由和部署
```

每次升级必须满足三个条件：

1. 上一版本的核心测试仍然通过。
2. 新能力通过接口接入，而不是复制一套新的 Agent Loop。
3. README 写清楚"本版本新增什么、解决什么问题、引入什么代价"。

### 4.2 六个里程碑与依赖关系

| 阶段 | 项目版本 | 在上一版本上新增 | 必须复用的产物 | 不要提前引入 |
| --- | --- | --- | --- | --- |
| 1 | `v0.1` Mini Coding Agent | 模型调用、文件读写、测试执行、Git Diff | 无 | LangChain、LangGraph、多 Agent |
| 2 | `v0.2` Plugin Agent | 插件注册、工具 Schema、权限策略、人工确认 | v0.1 的 Tool 接口和测试 | MCP、向量库 |
| 3 | `v0.3` Stateful Agent | LangChain Adapter、LangGraph 状态图、Checkpoint、Resume | v0.2 的工具和权限层 | 复杂多 Agent |
| 4 | `v0.4` MCP Agent | MCP Server/Client、远程工具、会话和错误边界 | v0.3 的 State、Approval 和 Tool 测试 | RAGFlow、Dify |
| 5 | `v0.5` Repo-Aware Agent | 代码库索引、检索、Rerank、引用和变更影响分析 | v0.4 的 Agent Loop、MCP、权限 | 微调模型 |
| 6 | `v1.0` Production Agent | Sandbox、Langfuse、LiteLLM、自动评估、Docker 部署 | v0.5 的全部业务能力和测试集 | Kubernetes、微服务拆分 |

对照项目怎么用（先读、再仿、最后才是自己写）：

- **learn-claude-code** 是 v0.1 的直接教材：先逐课跑通它的 12 个 `code.py`，理解每一层为什么存在，再脱离教程写自己的最小 Agent。
- **Aider** 用来理解 v0.1 的代码编辑闭环，不是要把 Aider 全部重写一遍。
- **DeepSeek Harness** 用来指导 v0.2 的插件、工作区和权限设计。
- **LangChain** 只替换 v0.3 的模型/工具集成层，不能吞掉业务代码。
- **LangGraph** 只负责 v0.3 的状态和流程编排，工具仍然来自 v0.2。
- **MCP** 把 v0.3 已有工具标准化，不重新设计工具业务逻辑。
- **RAG** 在 v0.5 为 Coding Agent 增加代码库理解能力，不单独另开知识库 Demo。
- **OpenHands/OpenCode** 用于对照 v1.0 的 Sandbox、事件流、Provider 和终端交互设计。
- **Claude Code 官方仓库** 用于学习插件、Command、Skill 和 Hook 如何包装能力。

### 4.3 通过验收时，回答问题而不是列框架

| 里程碑 | 必须回答的问题 |
| --- | --- |
| v0.1 | Agent 如何读懂代码、修改文件、运行测试？ |
| v0.2 | 哪些工具允许执行？谁能批准高风险操作？ |
| v0.3 | 进程重启后如何恢复？失败如何重试而不重复写入？ |
| v0.4 | 远程工具如何认证？MCP 是否绕过了业务权限？ |
| v0.5 | Agent 如何找到正确代码？回答如何引用证据？ |
| v1.0 | 如何知道系统变差了？一次任务成本多少？如何安全部署？ |

如果当前阶段的问题还回答不了，就不要进入下一阶段。这样 LangChain、LangGraph、MCP 和 Langfuse 都会成为解决实际问题的工具，而不是需要背诵的框架名。

## 五、逐月计划

### 项目主线总览

| 阶段 | 主项目 | 直接学习的开源项目 | 阶段产出 |
| --- | --- | --- | --- |
| 第 1 个月 | `agent-lab v0.1 → v0.2` | learn-claude-code、Aider、DeepSeek Harness、Claude Code/OpenCode | 最小 Agent + 插件 + 权限 |
| 第 2 个月 | `agent-lab v0.3 → v0.4` | LangChain、LangGraph、MCP SDK | 同一批工具的状态化和标准化 |
| 第 3 个月 | `agent-lab v0.5` | LangChain/LlamaIndex、Qdrant、Ragas | 在 Coding Agent 上增加代码库 RAG |
| 第 4 个月 | `agent-lab v1.0-rc` | OpenHands、OpenCode、Langfuse、LiteLLM | Sandbox、事件流、多模型、Trace 和成本控制 |
| 第 5 个月 | `agent-lab v1.0` | 前面所有阶段 + Dify/RAGFlow 架构速读 | 可部署、可评估、可演示的完整项目 |

### 第 1 个月：先做出 Mini Coding Agent

本月的执行顺序严格遵守"先读开源、再动手写"：前两周以 learn-claude-code 为主教材看懂 Harness 的每一层，再写自己的代码。

**第 1 周：跟学 learn-claude-code 前半部分（s01～s06），并用 Aider 体验真实闭环。** 克隆 [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)，逐课运行 `code.py`：从不到 30 行的最小 Agent Loop 开始，依次叠加工具调用、任务规划、技能加载和三层上下文压缩。每课做三件事：跑通、改一处行为（比如换一个工具、改一处 Prompt）、写一段笔记说明这一层解决了什么问题。同时在一个小型 Python 仓库中运行 [Aider](https://github.com/Aider-AI/aider)，跟踪它如何读取文件、构造上下文、生成修改、应用 Patch 和运行 Git Diff，至少修复 3 个真实小 Bug。产出：12 课中前 6 课的学习笔记、一张 Aider 调用链图。遇到问题时补：Python 文件读写、Git、Patch、Shell 命令和模型 API。

**第 2 周：跟学后半部分（s07～s12），然后脱离教程写 `v0.1`。** 完成后台任务、多 Agent 协作和 Worktree 隔离等进阶课程后，**合上教程**，不使用任何 Agent 框架，用原生模型 SDK 手写自己的最小 Coding Agent：文件读取、文件写入、Shell 测试、Git Diff、最大轮数和人工确认，先用命令行，暂不做前端。写完后回头对比：自己的实现和 learn-claude-code 的对应课程差在哪？把差距记在 README 里。产出：`agent-lab v0.1`，包含 10 个工具调用测试和一段演示视频。遇到问题时补：`asyncio`、超时、重试、JSON Schema、Pydantic。

**第 3 周：研究 DeepSeek Harness 并写插件。** 跟随 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 官方示例，定位 Agent Loop、工具注册、工作区、权限、会话和插件加载路径（注意它是 TypeScript 技术栈，读架构即可，不必逐行读实现）；重点理解"一切皆插件"如何把 v0.1 里硬编码的能力拆成可插拔组件。给自己的 `agent-lab` 写一个"代码仓库统计"或"高风险命令审批"插件，即 `v0.2`。产出：插件代码、架构笔记、一次对 Harness 的源码级阅读记录。再次提醒：该项目是开发者预览版，学习架构和插件设计，不要绑定具体 API。**备选方案**：如果届时它迭代过快导致示例跑不通，就改为精读 OpenCode 的 Provider/Tool 层或 Aider 的编辑循环源码，本周目标不变——看懂一个真实 Agent 的插件与权限设计。

**第 4 周：对比 Claude Code 与 OpenCode 的扩展方式。** 速读 [Claude Code](https://github.com/anthropics/claude-code) 的插件与命令机制，对比 [OpenCode](https://github.com/anomalyco/opencode) 的 Provider、Session 和 Tool 设计。产出：一张对比表，至少回答"工具如何注册、能力如何扩展、会话如何保存、命令如何触发"；据此确定自己的 Mini Coding Agent 借鉴哪种插件和命令结构。

**第 1 个月验收：** 能讲清一个 Agent Harness 由哪些层构成（Loop、工具、规划、上下文管理、权限与隔离），能解释一个 Coding Agent 的完整请求链路，并能独立实现"读懂代码→修改代码→运行测试→根据错误继续修复"。

### 第 2 个月：补 Agent 框架和工具协议

**第 5 周：用 LangChain 重构模型与工具层。** 只学 ChatModel、Prompt Template、Structured Output、Runnable、Tool 和 Retriever；将模型调用和工具定义抽成 Adapter，使原生 SDK 与 LangChain 两个实现可切换，并有同一组测试。

**第 6 周：用 LangGraph 加入状态、暂停和恢复。** 将"分析→修改→测试→修复"改成显式 StateGraph，增加最大轮数、Checkpoint、Interrupt、Resume 和人工审批。产出：状态图、恢复测试、失败案例表（对应 `v0.3`）。遇到问题时补：状态机、持久化、事务、幂等和任务队列。

**第 7 周：用 MCP 暴露工具。** 用 [MCP Python SDK v2](https://github.com/modelcontextprotocol/python-sdk) 将文件查询、测试运行、Git Diff 和代码搜索改成 MCP Tools，至少实现一个 Resource；同时保留普通 Python 函数工具，比较两种接入方式。产出：MCP Server、Client、权限测试和错误处理（对应 `v0.4`）。遇到问题时补：HTTP、stdio、认证、Schema、超时和权限边界。

**第 8 周：框架对比与小版本发布。** 对比原生 SDK、LangChain、LangGraph 和 MCP 的职责，不继续加框架；为项目增加 README、Docker Compose、CI 和 30 条评估任务。产出：`v0.4` 发布，一篇"为什么每一层都存在"的架构说明——写这篇文章时回头对照 learn-claude-code 的 12 课，检验自己是否真理解了每一层的动机。

**第 2 个月验收：** 能在不混淆职责的情况下说明 LangChain、LangGraph、MCP 和 Agent Runtime 的区别，并且同一批工具仍能在新框架下复用。

### 第 3 个月：在 Coding Agent 上增加代码库 RAG

**第 9 周：文档与代码摄取。** 用 LangChain 完成主线，用 [LlamaIndex](https://github.com/run-llama/llama_index) 速读和比较 RAG 数据抽象（不要两个都深入）。对代码文件、README、Issue 和项目规范做清洗、切分、Embedding、元数据和增量更新。产出：可重复执行的摄取流水线。

**第 10 周：向量检索、重排与引用。** 在 [Qdrant](https://github.com/qdrant/qdrant) 和 [pgvector](https://github.com/pgvector/pgvector) 中二选一深入，实现 Top-K、Filter、Hybrid Search、Rerank、引用和无证据拒答。产出：`agent-lab v0.5` 最小可用版本。遇到问题时补：Embedding、BM25、索引、SQL、检索指标和成本核算。

**第 11 周：评估和错误分析。** 用 [Ragas](https://github.com/explodinggradients/ragas) 和 [Promptfoo](https://github.com/promptfoo/promptfoo) 建立 80～100 条测试集，区分未召回、噪声、上下文丢失、幻觉和权限错误。产出：Baseline、优化后指标、人工抽检和失败案例。

**第 12 周：API、缓存和部署。** 加入 FastAPI、PostgreSQL、Redis 和 Docker Compose，实现文档上传、索引状态、问答、反馈和管理接口。产出：一个可通过 Docker 启动的 Repo-Aware Coding Agent，**完成第一版简历并开始投递日常实习**——大厂日常实习全年滚动招聘，此时的项目已经够格，不要等 v1.0。

**第 3 个月验收：** Agent 能引用文件路径、符号和行号，能对 Issue 定位相关代码，并具备权限、拒答、评估、反馈和部署说明。

### 第 4 个月：研究生产级 Coding Agent

**第 13 周：OpenHands（注意学 V1）。** [OpenHands](https://github.com/All-Hands-AI/OpenHands) 于 2026 年 4 月废弃了 V0 架构，正在迁移到基于 [software-agent-sdk](https://github.com/OpenHands/software-agent-sdk) 的 V1。**学习目标对准 V1 SDK**：Agent 抽象、Runtime/Sandbox、工具调用和任务状态；不要花时间读 V0 的 Agent Controller 和 Event Stream 遗留代码。产出：完整架构图，找到并运行一个相关测试，写一个小型源码改动。遇到问题时补：Docker Sandbox、事件驱动、WebSocket、进程隔离。

**第 14 周：安全和执行环境。** 给自己的 Coding Agent 加入命令 Allowlist、工作区隔离、路径检查、超时、日志脱敏和人工审批；构造 Prompt Injection、越权文件读取、危险命令和重复写入测试。产出：安全测试报告和威胁模型。

**第 15 周：Langfuse 与 LiteLLM。** 用 Docker Compose 自托管 [Langfuse](https://github.com/langfuse/langfuse)，接入 [LiteLLM](https://github.com/BerriAI/litellm) Proxy 配置两个模型、超时、重试和 Fallback；把分类、抽取、路由等简单任务迁移到低成本模型。产出：成功率、P95、Token/任务和成本对比 Dashboard。

**第 16 周：CI、自动评估、压测与部署。** PR 自动执行 Ruff、Pyright、pytest 和 AI 回归测试；测试 1、10、30 并发；部署到云服务器或本地可访问环境。同时对比 OpenCode 和 Claude Code 的扩展设计，把自己的 Agent 能力打成一个可安装的插件或命令。

**第 4 个月验收：** Agent、MCP、模型网关和可观测性形成完整链路；能解释 Coding Agent 为什么需要 Sandbox、状态、事件流、权限和可观测性，而不是只有"调用模型 + 执行命令"。

### 第 5 个月：综合项目、开源贡献和求职

**第 17 周：定题与架构。** 综合项目固定为 `agent-lab v1.0`：代码仓库分析与自动修复测试失败 Agent。最低功能：LangGraph 状态工作流、至少 3 个工具、1 个 MCP Tool、RAG、权限、人工审批、幂等、Langfuse Trace、自动评估、Docker 部署。产出：一页 PRD、架构图、时序图、状态图、数据模型、100 条评估任务、两周开发排期。

**第 18 周：实现主流程。** 先打通端到端最小路径，再加持久化、权限、错误处理、评估和 Trace；不新增框架，只补当前阻塞点。每天记录一个技术决策和一个失败案例。

**第 19 周：评估、压测和开源回馈。** 跑 100～200 条固定测试集，定位 Top 5 失败原因；完成压测、成本统计和安全测试；速读 [RAGFlow](https://github.com/infiniflow/ragflow) 和 [Dify](https://github.com/langgenius/dify) 的相关链路并画出架构图；至少向其中一个项目（learn-claude-code 的中文化补充、文档勘误也是有效贡献）提交文档、测试或修复 PR，如果暂时不适合提交，至少开一个可复现 Issue。

**第 20 周：发布与求职。** README 写清问题、架构、启动、评估、失败案例、成本、限制和后续工作；录制 5 分钟演示（用户问题、Agent 规划、工具调用、审批、结果和 Trace）；准备 15 分钟架构讲解和 10 个常见面试问题。

简历表达模板：

```text
设计并实现基于 FastAPI、LangGraph、PostgreSQL 和 Qdrant 的代码仓库分析与修复 Agent；
构建 100+ 条离线评估集，将任务成功率从 X% 提升至 Y%；
通过检索重排和模型分级路由，将 P95 延迟降低 X%、单任务成本降低 Y%；
实现 MCP 工具接入、人工审批、幂等写入、失败恢复与 Langfuse 全链路追踪。
```

**第 5 个月验收：** 1 个综合项目、2 个重点阶段产物、1 个实验仓库，累计至少 200 条评估样例、100 个有效测试用例、3 篇源码阅读笔记，一份简历和 5 分钟项目演示。

## 六、项目遇到问题时的技术补充规划

| 项目遇到的问题 | 立刻补的技术 | 不需要提前学的内容 |
| --- | --- | --- |
| 工具调用失败 | JSON Schema、Pydantic、重试、超时 | 深入训练模型 |
| Agent 状态混乱 | 状态机、LangGraph、持久化、幂等 | 多 Agent 群舞 |
| RAG 找不到内容 | 切分、Embedding、BM25、Rerank、评估 | 微调大模型 |
| 服务卡顿 | asyncio、并发限制、缓存、队列、P95 | Kubernetes 全套 |
| 命令执行危险 | Sandbox、Allowlist、权限、审计 | 复杂多租户架构 |
| 费用不可控 | LiteLLM、模型路由、预算、Token 统计 | 自建 GPU 集群 |
| 问题无法定位 | Langfuse、Trace、结构化日志、回归测试 | 复杂监控平台 |

## 七、每周时间安排

以每周 22 小时为例：

| 时间 | 任务 | 时长 |
| --- | --- | ---: |
| 周一至周四 | 文档、原理、最小实验 | 8 小时 |
| 周五 | 测试、重构、技术笔记 | 2 小时 |
| 周六 | 主项目开发 | 6 小时 |
| 周日 | 开发、评估和复盘 | 4 小时 |
| 零散时间 | 源码、Issue、英文文档 | 2 小时 |

每周日复盘四个问题：本周可运行成果是什么？哪个指标变好或变差？一个可复现失败是什么？哪些内容只是看过但没有代码验证？

## 八、项目验收评分表

每个重点项目按 100 分评分，低于 75 分不要急着开始新项目。

| 维度 | 分值 | 及格标准 |
| --- | ---: | --- |
| 业务问题与范围 | 10 | 用户、目标和非目标清晰 |
| 架构与代码 | 15 | 分层合理、类型清晰、无成片重复 |
| 核心功能 | 15 | 主流程稳定可复现 |
| 测试 | 15 | 核心模块和失败路径有测试 |
| AI 评估 | 15 | 固定 Dataset、指标、错误分析 |
| 可靠性与安全 | 10 | 超时、重试、权限、审批、幂等 |
| 可观测性 | 8 | Trace、日志、延迟、Token、错误 |
| 部署与复现 | 7 | Docker/文档可在新环境运行 |
| README 与演示 | 5 | 架构、结果、限制表达清楚 |

## 九、暂时不要投入过多时间的内容

- 不要同时学习 LangGraph、CrewAI、AutoGen、Semantic Kernel 等多个 Agent 框架。
- 不要把"能运行 Dify/RAGFlow"误认为会开发 AI 应用。
- 不要一开始做多 Agent，先建立单 Agent 的成功率和成本 Baseline。
- 不要为了追随模型新闻频繁重写项目，模型放在 Adapter/Provider 层即可替换。
- 不要只调 Prompt 而不维护测试集和失败分类。
- 不要在没有真实流量时过早使用 Kubernetes、微服务或复杂消息系统。
- 不要把 API Key、私密文档或用户数据提交到公共仓库。

## 十、大厂 JD 对照：这份计划到底练对了没有

本节内容来自对 2026 年字节、腾讯、阿里、美团、快手、蚂蚁等大厂校招/实习 JD 的调研（2026-08 核实）。结论：**计划主线与应用/工程岗高度匹配，但大厂 JD 暴露了三个纯做项目补不上的短板**，已合并进上面的 P0 表和周计划。

### 各家在招什么（应用/Agent 方向）

- **字节跳动**：2026 校招 5000+ Offer，2559 个岗位中 1205 个与 AI 直接相关；筋斗云计划八大领域含"大模型应用、AI Coding"。其 Agent 产品（如 Aime）工程岗 JD 关键词：上下文组装与压缩、Memory 机制（短期/长期/情景记忆）、Skill 体系与插件化生态、Agent 端到端效果调优——注意"不限技术栈，全栈或跨方向者优先"。
- **腾讯**：微信 WXG 招大模型后台开发实习生（可转正），工作内容明确写着 RAG 系统、Agent 系统的工程设计和开发；广告与混元团队有多个 Agent 构建方向。
- **阿里**：淘天集团直接开设"AI Agent 应用开发工程师"校招岗；另有 AI Agent 优化工程师（训练/数据/评测方向）、阿里云 AI Agent 研发（AI Coding 方向）。评测（Evaluation）在阿里被单独设岗，足见其权重。
- **美团**：LongCat 校招提前批 + 基础研发平台 Agent 实习生；官方 JD 写明"设计和实现高效的 Agentic 工作流""构建面向 Agent 的评估体系与方法"。注意其提前批面试会手撕困难题并深挖 Transformer 细节。
- **快手**：2026 秋招 200+ 职位类型，大模型岗位需求同比翻倍；商业化 AI 应用岗偏好 Java/Python 双栈，且要求 SFT/LoRA/RAG 有实际落地。
- **蚂蚁**：2026 春招技术岗占 85%，其中超 70% 与 AI 直接相关（AI 研究、AI 应用、AI Infra）。

一份基于 200+ 份 2026 校招 JD 的统计还给出两个关键数字：**LangChain 出现在 34.3% 的 Agent JD 中仍是第一框架，而 LangGraph + MCP 已是生产环境标配，89% 的团队做可观测性**——这直接验证了本计划把 LangGraph、MCP、Langfuse 放进主线的选择。同一份分析的忠告与本计划的理念一致：简历不要堆框架名，要能讲出"为什么选这个框架"。

### JD 验证了计划的哪些设计

| JD 高频要求 | 计划中的对应 |
| --- | --- |
| Agentic 工作流工程化落地（美团、淘天） | v0.1→v1.0 整条主线 |
| Agent 评估体系（美团、阿里评测岗） | 第 11、16、19 周的评估流水线，200+ 条评估集 |
| RAG 系统工程（腾讯 WXG、快手） | 第 3 个月整月 + Hybrid Search/Rerank |
| Memory/上下文压缩（字节 Aime） | v0.3 State 设计；learn-claude-code 的上下文压缩课 + Agent Loop 的上下文组装 |
| Skill 体系与插件化生态（字节 Aime） | learn-claude-code 的技能加载课 + v0.2 插件层 + Claude Code 扩展机制对照 |
| MCP/工具生态（多家） | 第 7、14 周，MCP SDK v2 |
| 沙盒校验、权限隔离（多家安全治理方向） | v0.2 权限层 + 第 14 周安全周 |
| GitHub 开源项目/技术博客加分（多家明示） | 单仓库作品 + 第 19 周开源 PR + 公开的技术博客 |

### JD 暴露的三个短板（已并入计划）

1. **算法笔试是硬门槛。** 所有大厂校招/实习第一关都是笔试和手撕代码（美团提前批甚至到困难题级别），这是项目做得再好也绕不过的关卡。已在 P0 表新增"算法与数据结构"一行：**LeetCode 每周 3～5 题，贯穿 20 周**，从第 1 周就开始，不要留到面试前突击。
2. **LLM 原理要能"讲清"，即使不做算法岗。** 字节、美团的工程岗面试同样会问 Transformer、Attention、KV Cache；牛客面经显示 CoT/ReAct 等推理范式是高频题。已在 P0 表新增"LLM 原理（讲得清即可）"一行——目标是白板讲清推理过程，不是学会训练。做 v0.1 时对上下文窗口、Token 成本的实际体感会让这部分事半功倍。
3. **投递窗口比计划终点更早。** 字节 8 月初启动校招、美团提前批 8 月初截止、蚂蚁春招 3 月启动——大厂"金九银十 + 提前批更早"的节奏意味着：**不要等 v1.0 完成才投简历**。第 12 周（约第 3 个月末）完成 Repo-Aware Agent 后就具备投日常实习的资格，日常实习转正或积累经历后再冲下一轮，比憋大招更符合校招现实。已相应调整第 12 周产出："完成第一次作品集整理"升级为"完成第一版简历并开始投递日常实习"。

### 岗位甄别提醒

- 本计划瞄准**应用/工程岗**。字节 Seed、快手商业化等 JD 里的 SFT/RLHF/预训练要求属于**算法岗**路线，不要看到"大模型"三个字就投，先分清 JD 属于哪条线。
- 少数 Agent 岗要求 Java/Spring Boot 或 Go 技术栈（腾讯部分后台岗、快手商业化偏 Java）；Python 主线不冲突，但如果有 Java 基础，简历里保留它。
- "Agent 工程师"至少有 6～7 个细分方向（应用开发、平台、评测、安全、多模态、Infra），投递前读完整 JD，把简历措辞对准那个方向的关键词。

## 十一、岗位匹配与最终检查

优先搜索岗位：大模型应用开发工程师、AI 应用工程师、LLM Engineer、AI 后端工程师、Agent/智能体开发工程师、RAG 工程师、AI Workflow Engineer、AI 平台工程师（应用平台方向）。

范围说明：本计划瞄准的是**应用/工程岗**，刻意不覆盖算法岗要求的 SFT/RLHF/微调/推理优化（那是另一条至少同样长的路线）。上一节的大厂 JD 对照已详细说明两类岗位的区分和投递注意事项。

简历关键词必须来自真实实现：`Python`、`FastAPI`、`asyncio`、`PostgreSQL`、`Redis`、`Docker`、`RAG`、`Hybrid Search`、`Rerank`、`LangGraph`、`Tool Calling`、`MCP`、`Langfuse`、`Evaluation`、`Human-in-the-loop`、`Idempotency`、`Prompt Injection Defense`。

最终检查清单：

- [ ] 能手写 30 行级别的最小 Agent Loop，并讲清 Harness 每一层（工具、规划、上下文压缩、权限、隔离）为什么存在。
- [ ] 能用原生 SDK 完成 Streaming、Structured Output 和 Tool Calling。
- [ ] 能解释并实现异步并发、Timeout、Retry、Cancellation。
- [ ] 有含检索评估、引用和权限的 RAG 项目。
- [ ] 有含状态、恢复、审批和安全测试的 Agent 项目。
- [ ] 编写过 MCP Server 和 Client（基于 v2 SDK），理解 MCP 与 REST 的差异。
- [ ] Langfuse 能追踪模型、检索、工具和 Agent 节点。
- [ ] 有自动 AI 回归测试，而不是只做人工聊天测试。
- [ ] 项目可以通过 Docker Compose 启动。
- [ ] 核心流程有单元、集成和失败路径测试。
- [ ] README 展示架构、指标、失败案例和成本参考。
- [ ] 能完成 5 分钟项目演示和 15 分钟架构讲解。
- [ ] LeetCode 累计 60～100 题，中等题 30 分钟内 bug-free。
- [ ] 能白板讲清 Transformer 推理过程、KV Cache 和 ReAct/CoT 范式。
- [ ] 第 12 周起已投递日常实习，简历措辞对准目标 JD 关键词。

> 一句话执行建议：第一个月跟着 learn-claude-code 把 Harness 的每一层看懂、再手写一遍，第二个月补框架与协议，第三个月给 Agent 装上代码库理解能力，第四个月补齐 MCP、评估、可观测性和部署，第五个月只用一个真实场景打磨成功指标、失败分析、可运行的求职作品。

---

*本文初版写于 2026-08-17，2026-08-20 修订。文中所有开源项目链接、版本号、项目状态以及字节/腾讯/阿里/美团/快手/蚂蚁的校招 JD 信息均在发布当日核实。如果在几个月后读到这篇文章，请注意 AI 工程领域迭代极快——DeepSeek Harness 尚在预览期、MCP 规范每年多次修订、各厂招聘批次随时开启，动手前请再次确认官方文档和招聘官网。*
