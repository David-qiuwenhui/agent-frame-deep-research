# 财经权限系统 AI Agent 框架选型报告

> **生成日期**: 2026-06-01 | **研究来源**: 30+ 网络来源 | **置信度**: 高

---

## 目录

1. [项目背景与需求分析](#1-项目背景与需求分析)
2. [系统架构全景图](#2-系统架构全景图)
3. [候选框架概览](#3-候选框架概览)
4. [六维深度对比](#4-六维深度对比)
5. [用户关注框架专项分析](#5-用户关注框架专项分析)
6. [技术架构推荐方案](#6-技术架构推荐方案)
7. [实施路线图](#7-实施路线图)
8. [风险与决策建议](#8-风险与决策建议)
9. [参考来源](#9-参考来源)

---

## 1. 项目背景与需求分析

### 1.1 业务场景

团队负责一个**财经权限系统**的开发，该系统以微前端形式运行于类似金蝶云的平台之上。平台提供了一个 Agent 侧边栏作为 AI 辅助作业的统一入口，实际路由到各子应用的 Agent 后台。

### 1.2 核心需求

| 需求维度 | 具体描述 | 优先级 |
|---------|---------|--------|
| **Skills** | 业务化技能：合同比对、合规检查、数据分析等 | P0 |
| **MCP** | 通过 MCP 协议接入后台数据源和 API | P0 |
| **Memory** | 会话记忆、文档上下文保持、历史分析记忆 | P1 |
| **Tools** | 访问财经后台数据的工具集 | P0 |
| **文档比对** | 合同 vs 电子流数据逐项比对 | P0 |
| **智能分析** | 数据趋势分析、异常检测、报告生成 | P0 |
| **报告输出** | 结构化分析报告和比对报告 | P0 |
| **独立部署** | 作为项目级 Agent 后台独立部署 | P0 |
| **Python** | 技术栈要求 Python | P0 |

### 1.3 需求架构图

```mermaid
graph TB
    subgraph Platform["金蝶云平台"]
        UI["统一前端入口"]
        Sidebar["Agent 侧边栏<br/>（统一入口）"]
    end

    subgraph OurSystem["财经权限系统（我们的应用）"]
        Frontend["财经前端"]
        Backend["财经后台"]
        AgentBackend["项目级 Agent 后台<br/>（本次选型目标）"]
    end

    subgraph AgentCore["Agent 核心能力"]
        Skills["Skills<br/>合同比对/合规检查/数据分析"]
        MCP["MCP Server<br/>后台数据访问"]
        Memory["Memory<br/>会话/文档/历史记忆"]
        Tools["Tools<br/>API 调用工具集"]
    end

    subgraph Output["输出"]
        Report["分析报告"]
        Comparison["比对报告"]
        Alert["异常告警"]
    end

    UI --> Frontend
    UI --> Sidebar
    Sidebar -->|"路由"| AgentBackend
    Frontend --> Backend
    AgentBackend --> Skills
    AgentBackend --> MCP
    AgentBackend --> Memory
    AgentBackend --> Tools
    MCP -->|"数据访问"| Backend
    Skills --> Report
    Skills --> Comparison
    Skills --> Alert
```

---

## 2. 系统架构全景图

### 2.1 Agent 与微前端集成架构

```mermaid
graph LR
    subgraph Browser["用户浏览器"]
        Shell["平台 Shell"]
        MicroApp1["子应用 A"]
        MicroApp2["财经系统"]
        MicroApp3["子应用 C"]
        AgentSidebar["Agent 侧边栏"]
    end

    subgraph AgentRouting["Agent 路由层"]
        Router["Agent Router"]
    end

    subgraph AgentBackends["各项目 Agent 后台"]
        AgentA["项目 A Agent"]
        AgentB["财经 Agent<br/>⭐ 我们的后台"]
        AgentC["项目 C Agent"]
    end

    subgraph FinanceBackend["财经后台服务"]
        API["REST API"]
        DB[("数据库")]
        FileStore["文件存储"]
    end

    Shell --> MicroApp1
    Shell --> MicroApp2
    Shell --> MicroApp3
    Shell --> AgentSidebar

    AgentSidebar -->|"WebSocket/HTTP"| Router
    Router -->|"路由到对应项目"| AgentA
    Router -->|"路由到对应项目"| AgentB
    Router -->|"路由到对应项目"| AgentC

    AgentB -->|"MCP Tools"| API
    API --> DB
    API --> FileStore
```

### 2.2 Agent 内部架构（目标态）

```mermaid
graph TB
    subgraph AgentSystem["财经 Agent 系统"]
        Entry["API Gateway<br/>接收平台路由请求"]

        subgraph Orchestrator["Agent 编排层"]
            Planner["Planner<br/>任务规划"]
            SkillRouter["Skill Router<br/>技能路由"]
        end

        subgraph SkillSet["业务 Skills"]
            S1["合同比对 Skill"]
            S2["合规检查 Skill"]
            S3["数据分析 Skill"]
            S4["报告生成 Skill"]
        end

        subgraph Infrastructure["基础设施"]
            MCPServer["MCP Server<br/>后台数据访问"]
            MemoryStore["Memory Store<br/>记忆存储"]
            ToolKit["Tool Kit<br/>工具集"]
        end
    end

    Entry --> Planner
    Planner --> SkillRouter
    SkillRouter --> S1
    SkillRouter --> S2
    SkillRouter --> S3
    SkillRouter --> S4

    S1 --> MCPServer
    S1 --> MemoryStore
    S2 --> MCPServer
    S3 --> ToolKit
    S4 --> MemoryStore

    MCPServer -->|"HTTP/gRPC"| FinanceAPI["财经后台 API"]
```

---

## 3. 候选框架概览

### 3.1 候选框架筛选

基于需求筛选出 **7 个候选框架** + 1 个排除项：

```mermaid
graph TD
    Input["用户提及: Deep Agent + OpenCode"] --> Filter{"筛选条件"}

    Filter -->|"Python 原生"| Python["✅ 保留"]
    Filter -->|"可独立部署"| Deploy["✅ 保留"]
    Filter -->|"Skills/MCP/Memory/Tools"| Cap["✅ 保留"]

    Filter -->|"Go 语言"| Exclude1["❌ OpenCode"]
    Filter -->|"编码助手,非框架"| Exclude2["❌ OpenCode"]
    Filter -->|"已归档"| Exclude3["❌ OpenCode"]

    Python --> F1["LangGraph + Deep Agents"]
    Python --> F2["Pydantic AI"]
    Python --> F3["CrewAI"]
    Python --> F4["Dify"]
    Python --> F5["Agno (Phidata)"]
    Python --> F6["Microsoft Agent Framework"]
    Python --> F7["smolagents"]

    style Exclude1 fill:#ff6b6b,color:#fff
    style Exclude2 fill:#ff6b6b,color:#fff
    style Exclude3 fill:#ff6b6b,color:#fff
    style F1 fill:#4ecdc4,color:#fff
    style F2 fill:#4ecdc4,color:#fff
    style F3 fill:#45b7d1,color:#fff
    style F4 fill:#45b7d1,color:#fff
```

### 3.2 候选框架总览表

| 框架 | GitHub | Stars | 许可证 | 语言 | 架构模式 | 维护状态 |
|------|--------|-------|--------|------|---------|---------|
| **LangGraph + Deep Agents** | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | ~10k+ | MIT | Python | 有向图/状态机 | 活跃 |
| **Pydantic AI** | [pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) | ~10k+ | MIT | Python | Agent-as-Capability | 活跃（快速迭代） |
| **CrewAI** | [crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) | ~25k+ | MIT | Python | 角色 Agent + Flows | 活跃 |
| **Dify** | [langgenius/dify](https://github.com/langgenius/dify) | ~100k+ | Apache 2.0 (修改) | Python | 可视化工作流平台 | 活跃 |
| **Agno (Phidata)** | [agno-agi/agno](https://github.com/agno-agi/agno) | ~15k+ | MPL-2.0 | Python | 平台 SDK | 活跃 |
| **Microsoft Agent Framework** | [microsoft/agent-framework](https://github.com/microsoft/agent-framework) | 新项目 | MIT | Python + .NET | 多 Agent 编排 | 活跃 (2026.4 v1.0) |
| **smolagents** | [huggingface/smolagents](https://github.com/huggingface/smolagents) | ~15k+ | Apache 2.0 | Python | Code Agent (~1k LOC) | 活跃 |
| ~~OpenCode~~ | [opencode-ai/opencode](https://github.com/opencode-ai/opencode) | ~120k+ | FSL | **Go** | 终端编码助手 | **已归档** |

> **关于 OpenCode**: 基于 Go 语言的终端 AI 编码助手（类似 Claude Code），不是通用 Agent 框架，无 Python API，无法作为后端服务部署。**不推荐用于本项目。**

> **关于 "Deep Agent"**: 最可能指 LangChain 于 2026年3月发布的 **Deep Agents** (`langchain-ai/deepagents`)——一个基于 LangGraph 的"开箱即用" Agent 运行时框架。

---

## 4. 六维深度对比

### 4.1 雷达图（综合评分）

```mermaid
graph LR
    subgraph Radar["六维评分 (1-5)"]
        direction LR

        subgraph LangGraphRow["LangGraph + Deep Agents"]
            LG_MCP["MCP: 4/5"]
            LG_Skill["Skills: 5/5"]
            LG_Mem["Memory: 5/5"]
            LG_Deploy["部署: 4/5"]
            LG_Doc["文档处理: 4/5"]
            LG_Eco["生态: 5/5"]
        end

        subgraph PydanticAIRow["Pydantic AI"]
            PA_MCP["MCP: 5/5"]
            PA_Skill["Skills: 4/5"]
            PA_Mem["Memory: 4/5"]
            PA_Deploy["部署: 4/5"]
            PA_Doc["文档处理: 3/5"]
            PA_Eco["生态: 4/5"]
        end

        subgraph CrewAIRow["CrewAI"]
            CR_MCP["MCP: 3/5"]
            CR_Skill["Skills: 4/5"]
            CR_Mem["Memory: 3/5"]
            CR_Deploy["部署: 4/5"]
            CR_Doc["文档处理: 3/5"]
            CR_Eco["生态: 4/5"]
        end

        subgraph DifyRow["Dify"]
            DF_MCP["MCP: 4/5"]
            DF_Skill["Skills: 3/5"]
            DF_Mem["Memory: 4/5"]
            DF_Deploy["部署: 5/5"]
            DF_Doc["文档处理: 5/5"]
            DF_Eco["生态: 5/5"]
        end
    end
```

### 4.2 详细评分矩阵

| 维度 | LangGraph + Deep Agents | Pydantic AI | CrewAI | Dify | Agno | MAF | smolagents |
|------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **MCP 支持** | 4 | 5 | 3 | 4 | 3 | 4 | 4 |
| **Skills 扩展** | 5 | 4 | 4 | 3 | 4 | 4 | 3 |
| **Memory 管理** | 5 | 4 | 3 | 4 | 4 | 3 | 2 |
| **独立部署** | 4 | 4 | 4 | 5 | 4 | 3 | 3 |
| **文档处理/RAG** | 4 | 3 | 3 | 5 | 3 | 3 | 3 |
| **生态成熟度** | 5 | 4 | 4 | 5 | 3 | 2 | 3 |
| **金融合规适配** | 4 | 5 | 3 | 3 | 3 | 3 | 2 |
| **Python 友好度** | 5 | 5 | 5 | 4 | 5 | 5 | 5 |
| **总分** | **36** | **34** | **29** | **33** | **29** | **27** | **25** |

### 4.3 各维度详解

#### MCP 支持

```mermaid
graph LR
    subgraph MCPSupport["MCP 支持级别"]
        Native["原生支持<br/>框架内置 MCP"]
        Adapter["适配器支持<br/>通过插件/扩展"]
        Community["社区支持<br/>需要自行集成"]
    end

    PA["Pydantic AI"] --> Native
    SA["smolagents"] --> Native
    DF["Dify"] --> Native

    LG["LangGraph"] --> Adapter
    MAF2["MS Agent Framework"] --> Adapter

    CR["CrewAI"] --> Community
    AN["Agno"] --> Community
```

| 框架 | MCP 集成方式 | 说明 |
|------|------------|------|
| **Pydantic AI** | 原生 Capability | MCP 作为可组合的 Capability，一行代码添加 |
| **smolagents** | 原生支持 | 任何 MCP Server 的工具可直接使用 |
| **Dify** | 平台内置 | 在管理界面配置 MCP Server |
| **LangGraph** | LangChain MCP Adapter | 通过适配器在工具节点中使用 |
| **CrewAI** | 自定义工具包装 | 需要自行封装 MCP 工具 |
| **Agno** | 上下文提供程序 | MCP 作为上下文源接入 |

#### Skills / Tools 扩展

```mermaid
graph TB
    subgraph SkillArch["Skills 架构对比"]
        direction TB

        subgraph LG_Skills["LangGraph 方案"]
            LG_T["工具节点: 任何 Python Callable"]
            LG_C["LangChain 工具生态: 1000+ 预构建工具"]
            LG_D["Deep Agents: 预置 write_todos 等工具"]
        end

        subgraph PA_Skills["Pydantic AI 方案"]
            PA_C2["Capabilities: 工具+钩子+指令打包"]
            PA_Y["YAML/JSON 定义: 无需改代码即可配置"]
            PA_A["人工审批: 金融合规必需"]
        end

        subgraph DF_Skills["Dify 方案"]
            DF_B["50+ 内置工具"]
            DF_API["自定义工具 API: HTTP/Webhook"]
            DF_Visual["可视化编排: 拖拽式"]
        end
    end
```

#### Memory 管理

| 框架 | 短期记忆 | 长期记忆 | 文件系统支持 | 持久化 |
|------|---------|---------|------------|--------|
| **LangGraph + Deep Agents** | 工作内存 | 持久化内存 | 文件系统后备 | Checkpointers |
| **Pydantic AI** | RunContext | 图状态 | Durable Execution | 支持 |
| **CrewAI** | Agent 内置 | RAG 知识库 | 有限 | 部分 |
| **Dify** | 会话管理 | 知识库 (RAG) | 内置 | 数据库 |
| **Agno** | 会话存储 | 知识存储 | 数据库存储 | 支持 |

---

## 5. 用户关注框架专项分析

### 5.1 Deep Agents (langchain-ai/deepagents)

> 这可能是用户提到的 "Deep Agent"

```mermaid
graph TB
    subgraph DeepAgents["Deep Agents 架构"]
        Core["Deep Agents 核心"]

        subgraph Features["核心特性"]
            Plan["Planning First<br/>自动生成 TODO 列表"]
            Ctx["Context Isolation<br/>上下文隔离"]
            Mem["Filesystem Memory<br/>文件系统记忆"]
            MCP2["MCP Auto-Discovery<br/>自动发现 MCP 工具"]
            Multi["Multi-Agent<br/>多 Agent 编排"]
        end

        subgraph Base["构建于"]
            LC["LangChain Core"]
            LG2["LangGraph Runtime"]
        end
    end

    Core --> Features
    Core --> Base
```

**评估**:
- **优势**: 开箱即用、MCP 原生支持、规划能力、记忆管理优秀
- **劣势**: 2026.3 才发布，较新；绑定 LangChain 生态
- **适用场景**: 长时间运行的复杂多步骤任务（如合同逐项比对）
- **GitHub**: [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents)
- **评分**: 4/5

### 5.2 OpenCode (opencode-ai/opencode)

**评估结论**: 不推荐用于本项目

| 维度 | 评估 | 说明 |
|------|------|------|
| 定位 | 不适合 | 终端编码助手，类似 Claude Code / Cursor |
| 语言 | 不适合 | Go 语言，非 Python |
| 部署 | 不适合 | 桌面/终端应用，无后端服务 API |
| Agent 框架 | 不适合 | 不是通用 Agent 编排框架 |
| 状态 | 不适合 | **已归档** (Archive) |
| 继任者 | 需注意 | Crush (charmbracelet/crush)，FSL 许可证 |
| MCP | 有限 | 支持作为 MCP 客户端，但无法作为服务端 |

### 5.3 其他值得关注的框架

#### Pydantic AI -- 金融系统最佳候选

```mermaid
graph TB
    subgraph PydanticAIArch["Pydantic AI 架构"]
        Agent["Agent: 定义工具+依赖+指令"]

        subgraph Capabilities["Capabilities 系统"]
            Tool["Tools: 函数工具"]
            Hook["Hooks: 生命周期钩子"]
            MCP3["MCP Capability: MCP 原生支持"]
            Human["Human Approval: 人工审批"]
        end

        subgraph Enterprise["企业特性"]
            OTel["OpenTelemetry: 可观测性"]
            Cost["Cost Tracking: 成本追踪"]
            Eval["Eval Framework: 评估框架"]
            Type["Type Safety: Pydantic 验证"]
        end
    end

    Agent --> Capabilities
    Agent --> Enterprise
```

**关键优势**:
- **类型安全**: Pydantic 验证是 Python 金融系统的事实标准
- **人工审批**: 工具调用可配置人工介入（金融合规必需）
- **可观测性**: OpenTelemetry + 成本追踪（生产环境必需）
- **评估框架**: 内置回归测试能力
- **银行示例**: README 即包含银行系统示例

#### Dify -- 最快交付路径

**关键优势**:
- **可视化工作流**: 拖拽式 Agent 编排
- **内置 RAG**: 原生文档处理 Pipeline
- **50+ 工具**: Google Search, DALL-E, Wolfram Alpha 等开箱即用
- **Docker 部署**: 一键 `docker-compose up`
- **100k+ Stars**: 社区最大

**关键劣势**:
- **平台而非库**: 定制化受限于平台能力
- **修改许可证**: 非 MIT，有附加条款
- **代码控制有限**: 复杂金融逻辑可能受限

---

## 6. 技术架构推荐方案

### 6.1 推荐方案矩阵

```mermaid
graph TB
    subgraph Decision["决策矩阵"]
        Q1{"团队偏好？"}
        Q1 -->|"代码优先,最大控制"| Code["方案 A: LangGraph + Deep Agents"]
        Q1 -->|"类型安全,金融合规"| Type["方案 B: Pydantic AI"]
        Q1 -->|"快速交付,低代码"| Low["方案 C: Dify"]
        Q1 -->|"多角色协作"| Role["方案 D: CrewAI"]

        Code --> Score1["评分: 5/5<br/>最全面的能力"]
        Type --> Score2["评分: 5/5<br/>最佳合规适配"]
        Low --> Score3["评分: 4/5<br/>最快交付"]
        Role --> Score4["评分: 3/5<br/>特定场景强"]

        Q2{"是否需要可视化？"}
        Q2 -->|"是"| Hybrid["方案 E: 混合架构<br/>Dify (RAG/文档) + LangGraph (Agent)"]
        Q2 -->|"否"| PureCode["纯代码方案 A/B"]
    end

    style Score1 fill:#4ecdc4,color:#fff
    style Score2 fill:#4ecdc4,color:#fff
    style Score3 fill:#45b7d1,color:#fff
```

### 6.2 推荐方案 A: LangGraph + Deep Agents（首推）

**适用场景**: 团队有较强 Python 能力，追求最大灵活性和可控性。

```mermaid
graph TB
    subgraph Recommended["推荐架构: LangGraph + Deep Agents"]
        Gateway["API Gateway<br/>FastAPI"]

        subgraph LangGraphApp["LangGraph Application"]
            subgraph Graph["State Graph"]
                Entry["Entry Node<br/>接收请求"]
                Planner2["Plan Node<br/>Deep Agents 规划"]
                Router2["Router Node<br/>技能路由"]

                subgraph Skills2["业务 Skills"]
                    Compare["合同比对<br/>LangGraph Tool Node"]
                    Compliance["合规检查<br/>LangGraph Tool Node"]
                    Analysis["数据分析<br/>LangGraph Tool Node"]
                    Report2["报告生成<br/>LangGraph Tool Node"]
                end

                Output2["Output Node<br/>格式化输出"]
            end
        end

        subgraph MCPServers["MCP Servers"]
            FinanceAPI["财经 API MCP<br/>访问后台数据"]
            DocParser["文档解析 MCP<br/>合同/发票解析"]
            VectorDB["向量 DB MCP<br/>语义检索"]
        end

        subgraph Storage["存储层"]
            Checkpoint["Checkpointer<br/>PostgreSQL"]
            MemoryDB["Memory Store<br/>Redis + PostgreSQL"]
        end
    end

    Gateway --> LangGraphApp
    Compare --> FinanceAPI
    Compare --> DocParser
    Analysis --> FinanceAPI
    Report2 --> VectorDB
    LangGraphApp --> Checkpoint
    LangGraphApp --> MemoryDB
```

**技术栈**:
- **框架**: LangGraph + Deep Agents
- **Web 框架**: FastAPI
- **MCP Server**: FastMCP (Python)
- **数据库**: PostgreSQL (持久化) + Redis (缓存)
- **向量库**: pgvector 或 Milvus (文档检索)
- **部署**: Docker + Kubernetes
- **可观测性**: LangSmith

### 6.3 推荐方案 B: Pydantic AI（金融合规最优）

**适用场景**: 金融合规要求严格，需要类型安全和人工审批机制。

```mermaid
graph TB
    subgraph PydanticAIApp["推荐架构: Pydantic AI"]
        API["FastAPI + Pydantic AI"]

        subgraph Agents["Agent 定义"]
            CompareAgent["合同比对 Agent<br/>YAML 定义 + Python 工具"]
            ComplianceAgent["合规检查 Agent<br/>带人工审批钩子"]
            AnalysisAgent["数据分析 Agent<br/>带成本追踪"]
        end

        subgraph Capabilities2["Capabilities"]
            MCP4["MCP Capability<br/>后台数据访问"]
            Approval["Human Approval<br/>金融合规审批"]
            OTel["OpenTelemetry<br/>全链路追踪"]
        end

        subgraph MCPs["MCP Servers"]
            BackendAPI["财经后台 API"]
            DocTool["文档处理工具"]
        end
    end

    API --> Agents
    Agents --> Capabilities2
    Agents --> MCPs
    MCP4 --> BackendAPI
```

### 6.4 技术选型决策流程

```mermaid
flowchart TD
    Start["开始选型"] --> Q1{"团队 Python 经验？"}

    Q1 -->|"资深"| Q2{"合规要求严格？"}
    Q1 -->|"一般"| Q3{"希望可视化编排？"}

    Q2 -->|"是,需要审计追踪"| Pydantic["选择 Pydantic AI<br/>类型安全+人工审批+OTel"]
    Q2 -->|"一般"| Q3

    Q3 -->|"是"| Dify["选择 Dify<br/>可视化+RAG+快速交付"]
    Q3 -->|"否"| Q4{"需要复杂状态管理？"}

    Q4 -->|"是,长时间任务"| LangGraph["选择 LangGraph + Deep Agents<br/>图编排+持久化+规划"]
    Q4 -->|"否,角色协作"| CrewAI2["选择 CrewAI<br/>角色定义+Flows"]

    Pydantic --> Deploy["独立部署: Docker/K8s"]
    Dify --> Deploy
    LangGraph --> Deploy
    CrewAI2 --> Deploy

    style Pydantic fill:#4ecdc4,color:#fff
    style LangGraph fill:#4ecdc4,color:#fff
    style Dify fill:#45b7d1,color:#fff
    style CrewAI2 fill:#45b7d1,color:#fff
```

---

## 7. 实施路线图

### 7.1 分阶段实施计划

```mermaid
gantt
    title Agent 框架选型与实施路线图
    dateFormat YYYY-MM-DD
    axisFormat %m/%d

    section Phase 1: 选型验证
    框架 POC 验证 (2周)           :p1t1, 2026-06-02, 14d
    技术评审与决策 (1周)          :p1t2, after p1t1, 7d

    section Phase 2: 基础搭建
    Agent 后台服务搭建 (2周)      :p2t1, after p1t2, 14d
    MCP Server 开发 (2周)         :p2t2, after p1t2, 14d
    Memory 和 Tools 集成 (1周)    :p2t3, after p2t2, 7d

    section Phase 3: 业务 Skills
    合同比对 Skill (2周)          :p3t1, after p2t3, 14d
    数据分析 Skill (2周)          :p3t2, after p2t3, 14d
    报告生成 Skill (1周)          :p3t3, after p3t1, 7d

    section Phase 4: 集成上线
    平台侧边栏对接 (2周)         :p4t1, after p3t3, 14d
    测试与优化 (2周)              :p4t2, after p4t1, 14d
    灰度发布 (1周)                :p4t3, after p4t2, 7d
```

### 7.2 POC 验证清单

```mermaid
graph LR
    subgraph POC["POC 验证项目"]
        T1["1. 框架安装与基本运行"]
        T2["2. MCP Server 集成测试"]
        T3["3. 合同比对 Skill Demo"]
        T4["4. Memory 持久化验证"]
        T5["5. 平台侧边栏对接验证"]
        T6["6. 性能与并发测试"]
    end

    T1 --> T2 --> T3 --> T4 --> T5 --> T6
```

| POC 项目 | 验证目标 | 通过标准 |
|----------|---------|---------|
| 框架基本运行 | 安装、配置、运行 Agent | 30 分钟内跑通 Hello World |
| MCP 集成 | 连接后台 API 的 MCP Server | 成功调用财经后台数据 |
| 合同比对 Skill | 实现一个简单的合同比对 | 输出结构化比对结果 |
| Memory 持久化 | 会话跨请求保持 | 重启服务后恢复上下文 |
| 平台对接 | Agent 侧边栏路由到后台 | 成功接收和响应消息 |
| 性能测试 | 并发 10 用户文档比对 | 响应时间 < 30s |

---

## 8. 风险与决策建议

### 8.1 风险矩阵

```mermaid
graph TB
    subgraph Risk["风险评估"]
        direction TB

        subgraph HighRisk["高风险"]
            R1["框架过新 (Deep Agents 2026.3)<br/>缓解: 有 LangGraph 底座保障"]
            R2["MCP 生态不成熟<br/>缓解: 自建 MCP Server"]
        end

        subgraph MedRisk["中风险"]
            R3["平台侧边栏对接兼容性<br/>缓解: 早期 POC 验证"]
            R4["Dify 定制化受限<br/>缓解: 评估混合架构"]
        end

        subgraph LowRisk["低风险"]
            R5["Python 技术栈适配<br/>缓解: 团队已有 Python 经验"]
            R6["框架性能瓶颈<br/>缓解: 异步+水平扩展"]
        end
    end
```

### 8.2 最终推荐

| 排名 | 方案 | 推荐理由 | 适用条件 |
|------|------|---------|---------|
| **1** | **LangGraph + Deep Agents** | 最全面的能力、MCP 原生、记忆管理优秀、生态成熟 | 团队有 Python 能力，追求可控性 |
| **2** | **Pydantic AI** | 金融合规最佳、类型安全、人工审批、可观测性 | 合规要求严格，需要审计追踪 |
| **3** | **Dify** | 最快交付、可视化、内置 RAG、100k+ 社区 | 团队偏好低代码，快速验证 |
| **4** | **CrewAI** | 多角色协作、MIT 许可、无 LangChain 依赖 | 多 Agent 角色分工场景 |

### 8.3 行动建议

1. **Phase 1 (第 1-2 周)**: 并行 POC LangGraph + Deep Agents 和 Pydantic AI
2. **Phase 1 末**: 技术评审会，团队投票选定框架
3. **Phase 2 (第 3-4 周)**: 基于选定框架搭建 Agent 后台 + MCP Server
4. **Phase 3 (第 5-8 周)**: 实现核心业务 Skills（合同比对、数据分析、报告生成）
5. **Phase 4 (第 9-12 周)**: 平台对接、测试、灰度发布

---

## 9. 参考来源

### 框架对比
1. [AI Agent Frameworks: LangGraph vs CrewAI vs AutoGen 2026](https://pecollective.com/blog/ai-agent-frameworks-compared/)
2. [Python AI Agent Frameworks Compared](https://pub.towardsai.net/i-compared-6-python-ai-agent-frameworks-so-you-dont-have-to-langgraph-vs-crewai-vs-pydanticai-vs-d8a5e6e43262)
3. [Comprehensive Comparison of Every AI Agent Framework in 2026](https://www.reddit.com/r/LangChain/comments/1rnc2u9/comprehensive_comparison_of_every_ai_agent/)
4. [AutoGen vs LangGraph vs CrewAI: Production Comparison](https://python.plainenglish.io/autogen-vs-langgraph-vs-crewai-a-production-engineers-honest-comparison-d557b3b9262c)
5. [CrewAI vs LangGraph vs AutoGen vs OpenAgents](https://openagents.org/blog/posts/2026-02-23-open-source-ai-agent-frameworks-compared)
6. [Top 7 AI Agent Frameworks for Developers in 2026](https://dev.to/thedailyagent/top-7-ai-agent-frameworks-for-developers-in-2026-3o63)

### Deep Agents
7. [LangChain Deep Agents Official](https://www.langchain.com/deep-agents)
8. [langchain-ai/deepagents GitHub](https://github.com/langchain-ai/deepagents)
9. [MCP Tools - LangChain Docs](https://docs.langchain.com/oss/python/deepagents/code/mcp-tools)
10. [LangChain Releases Deep Agents - MarkTechPost](https://www.marktechpost.com/2026/03/15/langchain-releases-deep-agents-a-structured-runtime-for-planning-memory-and-context-isolation-in-multi-step-ai-agents/)

### OpenCode
11. [opencode-ai/opencode GitHub](https://github.com/opencode-ai/opencode)
12. [OpenCode Developer Guide](https://lushbinary.com/blog/opencode-developer-guide-terminal-ai-coding-agent/)

### Dify
13. [langgenius/dify GitHub](https://github.com/langgenius/dify)
14. [7 Self-Hosted AI Tools](https://www.nocobase.com/cn/blog/7-self-hosted-ai-tools-build-business-app)

### MCP
15. [12 Frameworks to Build MCP AI Agents](https://blog.devgenius.io/12-frameworks-to-build-mcp-ai-agents-9fdfa3817e41)
16. [Agent Skills SDK GitHub](https://github.com/datalayer/agent-skills)

### Document Analysis
17. [FinAgent-RAG: Financial Document Analysis](https://arxiv.org/html/2605.05409v1)
18. [Financial Agentic RAG (LangGraph)](https://github.com/DeepakSilaych/Financial_Agentic_RAG)
19. [Agentic Document Extraction](https://medium.com/@vietexob/agentic-document-extraction-understanding-fe05fc2bba20)

### Other Frameworks
20. [Agno Official](https://www.agno.com/agent-framework)
21. [Microsoft Agent Framework GitHub](https://github.com/microsoft/agent-framework)
22. [Pydantic AI GitHub](https://github.com/pydantic/pydantic-ai)
23. [awesome-ai-agents-2026](https://github.com/ARUNAGIRINATHAN-K/awesome-ai-agents-2026)
24. [Best Open Source Agent Frameworks](https://www.ayautomate.com/blog/best-open-source-ai-agent-frameworks)

---

> **免责声明**: 本报告基于 2026 年 5-6 月的公开信息研究而成。各框架持续迭代，建议在决策前进行实际 POC 验证。部分 Star 数据为近似值，请以 GitHub 实时数据为准。
