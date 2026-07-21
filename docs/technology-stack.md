# SOC Technology Stack

> **文档职责**：技术选型、组件职责、代码包与可选组件。  
> **不写**：排期与落地状态（→ `implementation-roadmap.md`）、系统分层架构图（→ `architecture.md`）、端口运维细节（→ `deployment-guide.md`）、实体字段（→ `data-contracts.md`）。

## 1. 文档定位

供研发、测试、运维与 Agent 快速理解「用了什么技术、各自干什么」。四层底座见 `architecture.md` §3；Compose 端口与启动见 `deployment-guide.md`。

## 2. 技术栈总览

| 层级 | 技术 | 职责 | 目标底座层 |
|---|---|---|---|
| 前端 | React 18、TypeScript、Vite、Recharts、Nginx | CSF 六域指挥舱 + 域内面板 | 体验层 |
| 后端 API | Go HTTP API | 接入、查询、检测、事件、自动化、组件健康、证据、Portfolio | 智能 / 编排 / 数据平面入口 |
| Worker | Go worker | 定时数据源、检测、自动化、SLA、月报、风险同步 | 编排 + 数据平面调度 |
| 主数据库 | PostgreSQL 16 | 实体快照存储 | 数据平面 |
| 消息队列 | Redis 7 / Redis Stream | 内部队列与日志流消费 | 数据平面 |
| 检索存储 | OpenSearch 2.15 | 检测快照、告警、事件、检索日志 | 数据平面 / Detect |
| 对象存储 | MinIO | 证据附件、扫描报告、截图 | 数据平面 |
| 可选日志流 | Kafka | 大规模 / 云日志接入 | 数据平面 |
| 可选实时计算 | Flink | 清洗、窗口聚合、异常流 | 智能平面 |
| 可选长期分析 | ClickHouse | 长期审计与历史检索 | 数据平面 |
| 可选 SOAR | Shuffle / TheHive / Cortex | workflow / case / analyzer 适配 | 编排平面 |
| 部署 | Docker Compose | 本地与内部测试一键启动 | 平台 |

## 3. 前端实现

目录 `frontend/`，入口 `src/App.tsx`。一级导航为 NIST CSF 六域；域内挂载 `p0/` `p1/` `p2/` 面板，原明细/接入/检测等按域嵌入。默认落地：分析师 Detect、管理层 Govern。

IA 与场景 ID 见 `ui-command-cockpit-prototype.md`；用户可见功能见 `feature-guide.md`。

| 视图族 | 职责概要 | CSF 域 |
|---|---|---|
| Govern | KPI、决策、策略、AI 治理、值班、月报/Portfolio | Govern |
| Identify | 代码仓、关系卡、身份画像、分级/攻击面、数据源 | Identify |
| Protect | 基线、例外、有效性、漂移、数据防护目录 | Protect |
| Detect | 调查台、狩猎、UEBA/TI、规则工程 | Detect |
| Respond | 剧本、证据、审批、连接器、工单/通知 | Respond |
| Recover | 复盘、提案、演练、通报草稿 | Recover |

## 4. 后端实现

目录 `backend-go/`：`cmd/soc-api`、`cmd/soc-worker`。

| 文件 | 职责 |
|---|---|
| `api.go` / `api_placeholders.go` | HTTP 路由与占位能力 API |
| `models.go` / `models_placeholders.go` | 领域与请求/响应模型 |
| `store.go` / `store_placeholders.go` | 内存/快照实体存储 |
| `agent_research.go` | Agent 共研（模板 + 可选 LLM） |
| `ingestion.go` / `normalization.go` | 接入与标准化 |
| `detection.go` / `rule_dsl.go` | 检测、评分、告警与事件 |
| `automation.go` / `operations.go` | 工单、通知、Playbook、例外、复盘、月报、SLA |
| `connectors.go` / `components.go` | 外部组件与检索桥 |
| `integration.go` | Portfolio 风险同步 |
| `profiles.go` / `risk.go` | 画像与风险聚合 |
| `table_query.go` | 列表排序/筛选/分页 |

## 5. 可选组件与演进

| 场景 | 建议 |
|---|---|
| 内部测试 | Compose + `.env` 密钥；补 API Token / 反向代理 |
| 共享测试 | PostgreSQL / OpenSearch / MinIO 持久化与备份策略 |
| 生产化 | SSO/RBAC、集中日志与指标（门禁见路线图） |
| 规模化 | Kafka / Flink / ClickHouse / Data Lake |

`phase4` profile 可本地启动 Kafka、Flink、ClickHouse、Shuffle，用于智能运营预留演示。
