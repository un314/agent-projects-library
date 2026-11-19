# 13 电商运营 Agent

> 选品 / 定价 / 营销 / 分析四类 Agent 组成的运营中台，编排器按意图路由到对应工作流。

## 项目定位

把电商运营的三个核心决策——卖什么、卖多少钱、怎么吆喝——拆成三个独立的 Agent 工作流，用统一编排器调度，所有任务落表可追溯。

## 要解决的问题

- 选品依赖经验，缺乏数据支撑
- 定价静态，无法跟随竞品与库存变化
- 营销策划靠人工，创意与产能都不稳定
- 运营动作分散，缺少统一的任务记录与效果归因

## 技术栈与平台

| 层 | 技术 |
| --- | --- |
| 后端 | FastAPI + Python |
| Agent | LangChain 0.1.17+ + LangGraph 0.0.39+ |
| 检索 | RAG 知识增强 |
| 数据库 | PostgreSQL（asyncpg） |
| 缓存 | Redis |
| 前端 | Vue 3 + TypeScript |
| 部署 | Docker Compose + Nginx |

## 核心流程与架构

五层：客户端层 → 应用层（API Gateway：认证授权、限流）→ Agent 核心引擎（编排器 + 四类 Agent）→ 数据存储层 → RAG 知识增强。

请求进入后由编排器路由：

- **选品工作流**：品类分析 → 竞品监控 → 市场调研 → 选品推荐
- **定价工作流**：成本定价 → 竞品参考定价 → 动态调价策略 → 利润优化
- **营销工作流**：文案生成 → 活动策划 → 受众分析 → A/B 测试
- **分析 Agent**：汇总各工作流产出，出数据看板

Agent 统一继承基类，工具与工作流分目录组织（`agents/selector.py`、`agents/pricer.py`、`agents/marketer.py`、`agents/orchestrator.py`，工作流在 `workflows/product_selection.py` 等）。每次调用写入 `agent_tasks` 表，记录任务类型、Agent 名称、输入、输出、状态、用户与耗时。

## 可复用点

- **Agent 任务表设计**：task_type / agent_name / input / output / status，是所有 Agent 系统的审计底座
- **按 Agent 与工作流分目录的工程结构**：新增一个业务 Agent 只需加两个文件
- **编排器模式**：意图 → 工作流的路由层，与具体业务解耦
- **完整部署配置**：Docker Compose + Nginx + 前后端 Dockerfile

## 实现难度

**高**。82 页，业务面最宽，一个项目覆盖三条业务线。但架构高度模板化，先做通一条工作流再复制另外两条，是可行的推进方式。
