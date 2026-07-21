# SOC 安全运营中心

公司级 **AI-Native SOC** MVP：统一接入安全风险、资产、身份、零信任访问、云扫描与研发安全扫描结果，完成标准化 → 检测 → 告警 → 事件 → 受控自动化 → 复盘汇报。

体验层按 **NIST CSF 2.0** 组织为 **AI-SOC 智能态势感知指挥舱**（Govern → Identify → Protect → Detect → Respond → Recover）。

| 项 | 值 |
|---|---|
| **当前正式版** | `v0.5.0`（`VERSION` · `docs/release-plan.md`） |
| **入口** | UI `http://localhost:9011` · API `http://localhost:9010`（本地轻量常用 `:9012`） |
| **状态权威** | `docs/implementation-roadmap.md` |
| **架构权威** | `docs/architecture.md`（含 §6.1 IM 闭环分层） |
| **UI/IA 权威** | `docs/ui-command-cockpit-prototype.md`（v0.9 · §9.7 IM 回看） |
| **IM 闭环权威** | `docs/im-closed-loop-framework.md` |

---

## 1. 三份核心文档怎么读

README 记录**能力实现路径规划总览**；细节以三份权威文档为准（一文一责）：

```text
                    ┌─────────────────────────────────────┐
                    │  README（本文）· 能力实现路径总览    │
                    └──────────────┬──────────────────────┘
           ┌───────────────────────┼───────────────────────┐
           ▼                       ▼                       ▼
┌──────────────────┐  ┌────────────────────────┐  ┌─────────────────────┐
│ architecture.md  │  │ ui-command-cockpit-     │  │ implementation-     │
│ 系统架构 / 分层  │  │ prototype.md           │  │ roadmap.md          │
│ 数据流 / 边界    │  │ UI·IA·线框·交互范式    │  │ 唯一实现路径/状态   │
│ （设计视角）     │  │ （体验视角）           │  │ 前置/门禁/覆盖核对  │
└──────────────────┘  └────────────────────────┘  └─────────────────────┘
```

| 文档 | 回答的问题 | 不负责 |
|---|---|---|
| `docs/architecture.md` | 系统怎么分层、数据怎么流、六域能力边界是什么 | 排期、落地状态表 |
| `docs/ui-command-cockpit-prototype.md` | 指挥舱长什么样、六域怎么导航、Agent 怎么交互 | 后端实现状态、生产门禁 |
| `docs/implementation-roadmap.md` | **做到哪了、下一步做什么、缺什么前置** | 线框细节、组件选型长文 |
| `docs/release-plan.md` | 按版本（v0.3→v1.0）Demo / 发版公告口径 | 实施排期细节 |

完整文档地图见 `docs/context-map.md`。

---

## 2. 产品定位与架构骨架（摘自 architecture）

### 2.1 建设目标

| 目标 | 说明 | CSF 落位 |
|---|---|---|
| 统一数据接入 | API / 离线 / MQ / 日志网关 / 云日志适配 | Identify |
| 统一数据模型 | Asset、Identity、Finding、AccessLog、Alert、Incident | 数据平面 |
| 风险优先级 | 严重度、敏感度、暴露面、身份、业务、SLA | Identify / Detect |
| 检测与事件化 | 规则 DSL、ATT&CK、告警聚合 | Detect |
| 闭环运营 | 工单、通知、SLA、处置、例外、复盘、月报、Portfolio 反哺 | Respond / Recover / Govern |
| 智能运营 | 日志流、UEBA、TI、开源 SOAR 适配 | Detect / Respond |
| 指挥舱 IA | 六域导航 + 调查 / 编排体验 | 全域 |

### 2.2 四层底座

| 层 | 职责 | 关键实现 |
|---|---|---|
| **体验层** | CSF 六域指挥舱、Agent 共研、审批台账 | `frontend/`（React） |
| **智能平面** | 规则检测、UEBA/TI、Agent、提案回流 | DetectionEngine / SmartOps / Agent |
| **编排与动作平面** | Playbook、连接器、审批 HITL、ITSM/IM | Automation / Approvals |
| **数据平面** | 接入、标准化、存储、检索与流 | PG / OpenSearch / Redis·Kafka / MinIO |

### 2.3 端到端能力链路

```text
多源接入 → 标准化脱敏 → 资产/身份画像 → 风险评分 → 规则检测
    → 告警聚合 → Incident → Playbook/审批/证据 → 复盘/提案/月报
    → Govern 态势 / Portfolio 反哺
         ↘ 智能运营（UEBA/TI/日志分析）↗
```

原则摘要：MVP 可启动 · 标准化优先 · 上下文关联 · HITL 可审计 · CSF 骨架 · AI 可治理（L0–L3；**L4 未审批全自动明确不做**）。

---

## 3. 原型设计骨架（摘自 UI 原型 v0.9）

### 3.1 从「告警控制台」到「安全大脑指挥舱」

| 维度 | 传统控制台 | 本项目目标体验 |
|---|---|---|
| 组织方式 | 按技术模块 | 按 CSF 运营职能（GV/ID/PR/DE/RS/RC） |
| 分析师 | 翻列表拼上下文 | 调查台 + Agent 共研（Explain/Draft） |
| 管理层 | KPI 散落 | Govern：目标 ↔ 策略 ↔ 态势 |
| 处置 | 点按钮跑剧本 | Respond：辅助决策 → 受控自主（L2/L3） |
| 学习 | 复盘外挂 | Recover：案例 / 提案回流规则与策略 |

### 3.2 一级导航与全局壳

```text
Govern | Identify | Protect | Detect | Respond | Recover
治理指挥  识别图谱  保护基线  调查台    编排处置  恢复沉淀
```

全局常驻：顶栏（Agent / 审批铃铛 / 健康）· 态势 KPI · 右侧 Agent 抽屉（W-SH-01）· hash 路由 `#/domain[/scenario]`。

### 3.3 六域能力嵌入（实现路径对照）

| CSF 域 | 原型重心 | 主场景 ID（已落地） |
|---|---|---|
| **Govern** | 态势 / 策略 / 度量 / AI 治理 | `govern-situ` · `govern-policy` · `govern-metrics` |
| **Identify** | 代码仓风险 / Findings 台账 / 身份关系 / 攻击面与数据源 | `identify-risk` · `identify-findings` · `identify-identity` · `identify-surface` |
| **Protect** | 基线 / 例外防御 / 动作目录与 SOAR | `protect-baseline` · `protect-defense` · `protect-actions` |
| **Detect** | 调查台 / 狩猎信号 / 检测工程 / SmartOps | `detect-workbench` · `detect-hunt` · `detect-engineering` · `detect-smartops` |
| **Respond** | 处置审批 / 协同集成 | `respond-ops` · `respond-collab` |
| **Recover** | 复盘回流 / 演练恢复 | `recover-review` · `recover-drill` |

线框 W-GV-01…W-RC-01 / W-SH-01 与场景映射见 roadmap **§13.2**；子模块 G1–C5 全量表见 **§13.1**。

### 3.4 角色默认落地（演示开关，非真 SSO）

| 角色 | 默认域 | 说明 |
|---|---|---|
| 管理层 | Govern | 态势 / KPI / 月报 |
| SOC 分析师 | Detect | 调查台 / Agent |
| 安全工程师 | Protect | 基线 / 规则 / 数据源 |
| 值班长 | Respond | 处置 / 审批 / SLA |
| 业务 Owner | Identify | 资产与 Findings |
| Agent 服务账号 | Detect | 只读建议级 |

真 RBAC/SSO → roadmap §9（G-M4）。

---

## 4. 能力实现路径规划（总览）

> **权威细节**：`docs/implementation-roadmap.md`。  
> 规划分三轴并行：**UI 体验分期** · **技术建设 Phase 0–4** · **版本发版路径**；并用 **18 项六域服务能力** 衡量「有 / 占位 / 待前置」。

### 4.1 总路线一张图

```text
【已完成 · 可演示】                         【下一阶段 · 需外部前置】
UI: Phase A → B → C → D(体验补齐)          真数据源 / 真 ITSM·IM / 真下发
Tech: Phase 0 MVP → 1 契约 → 2 检测底座    SSO/RBAC · Secret · 审计 · CORS
      → 3 自动化台账 → 4 智能运营 MVP      → v0.6 真联动 → v0.7 门禁 → v1.0
版本: v0.3 → v0.4 指挥舱 → v0.5.0 Phase D
```

### 4.2 UI 轨：指挥舱体验分期

| 阶段 | 目标 | 状态 |
|---|---|---|
| **Phase A** | CSF 六域导航；原模块嵌入对应域 | **已完成** |
| **Phase B** | Detect 调查台 + Govern KPI + Identify 关系卡 | **已完成** |
| **Phase C** | Respond L2、Recover 提案回流 | **演示完成**（真下发/真图谱待 §9） |
| **Phase D** | Agent 抽屉、角色落地、Explain/Draft、Portal 深链、防御/SOAR 专页、Findings 台账、hash 路由、L3、CSF 雷达 | **已完成**（v0.5.0） |

指挥舱交付粒度（P0 → P1 → P2 → 深化）见 roadmap **§2.1**（调查台、策略中心、基线/例外、狩猎雷达、复盘提案、L2 证据篮、AI 治理开关等）。

**本阶段明确不做**：一次重写全部后端模型；未审批全自动封禁（L4）；聊天机器人作唯一入口。

### 4.3 技术轨：Phase 0–4

| 阶段 | 建设目标 | 核心交付 | 当前状态 |
|---|---|---|---|
| **Phase 0** | 本地 MVP 闭环 | 前端 / Go API / worker / PG / Redis / OpenSearch / MinIO | **已完成** |
| **Phase 1** | 数据接入标准化 | Kafka 契约、CMDB/身份/CI/CD/零信任/云扫描接入模型 | **契约已实现**；code_audit **已接**；其余待真实 Topic |
| **Phase 2** | 检测增强 | 规则 DSL、dry-run、聚合、反馈、ATT&CK、趋势 | **底座已实现**；待真实数据调优 |
| **Phase 3** | 自动化处置 | ITSM/IM、Playbook、审批、连接器、SLA、复盘月报 | **台账已实现**；待真 API |
| **Phase 4** | 智能运营 | Kafka/Flink、ClickHouse、UEBA、TI、开源 SOAR | **MVP 已实现**；Compose `phase4` 可选；待真组件 |

### 4.4 六域 18 项服务能力状态

| CSF | 能力 | 状态 |
|---|---|---|
| Govern | 态势感知与指挥大屏 | **有** |
| Govern | 安全策略即代码 | **有（占位）** |
| Govern | 合规框架映射 | **有（占位）** |
| Identify | 业务拓扑可视化 | **有**（关系卡） |
| Identify | 暴露面管理 ASM | **有**（轻量） |
| Identify | 漏洞与配置核查 | **有** |
| Protect | 策略编排中心（WAF/FW/EDR） | **有（占位）** |
| Protect | 零信任访问控制 | **有**（台账闭环） |
| Protect | 数据防泄漏 DLP | **有（占位）** |
| Detect | 告警聚合与降噪 | **有** |
| Detect | 智能调查画布 | **有** |
| Detect | 威胁狩猎（NL） | **有（占位）** |
| Respond | 剧本可视化编排 | **有（占位）** `graph_json` |
| Respond | 自动化工具箱 | **有**（部分） |
| Respond | 协同作战 IM 拉群 | **有（占位）** war-rooms；**IM 六 Bot 出站已通** |
| Recover | 灾备演练 RTO/RPO | **有（占位）** |
| Recover | AI Memory | **有（占位）** |
| Recover | 复盘报告 | **有** |

**统计**：已实现演示能力 **10** + 无外部依赖占位 **8**。真下发 / 真备份 / 强制 LLM 仍依赖外部；**IM 分类型告警出站已本地验证**（生产群与真源见 §4.6 / roadmap §14）。

### 4.5 版本发版路径（Demo / 公告口径）

| 版本 | 状态 | 能力跃迁 |
|---|---|---|
| v0.3.x | 已发布 | 本地 MVP 闭环 + 智能运营底座 |
| v0.4.0 | 已发布 | NIST CSF 指挥舱可演示 + 占位服务齐 |
| **v0.5.0** | **当前正式版** | Phase D 体验补齐（Agent 抽屉 / 角色 / Portal / L3 / 雷达） |
| **v0.5.x** | 进行中 | IM 六类 Bot 分路由出站 + code_audit 闭环（§4.6） |
| **v0.6.0** | 规划 | **真联动首发**：一类真实数据源持续输入 + 生产 IM 群 + 真 ITSM |
| **v0.7.0** | 规划 | 生产门禁过半：SSO/RBAC + 审计 |
| **v1.0.0** | 规划 | 共享环境可上线：真数据 + 真动作 + 门禁闭环 |

### 4.6 IM 六类告警与闭环（摘要）

> 出站：`im-data-flow.md` · 路由：`im-bot-routing.md` · 闭环：`im-closed-loop-framework.md` · 架构分层：`architecture.md` §6.1 · 路径：roadmap **§14 / §15**。

```text
多源输入 → SOC 台账/检测/Incident → 按风险类型路由出站（CL-1 深链 + CL-3 按钮）
  → @code_audit_upload_bot | @access_risk_bot | @data_security_bot
  → @cloud_security_bot | @threat_intel_bot | @soc_ops_bot
  → 人点按钮/命令（CL-4）或 API（CL-2）→ ApplyIMAction
  → Ticket/ITSM（CL-5）+ SourceSyncAdapter（code_audit→Portal；其余→Webhook/Noop）
```

| 路由 | 上游依赖（真实对接） | S8 反哺 |
|---|---|---|
| code_audit | Portal / CI `code-audit/*` Finding | Portal `/findings/soc-sync` |
| access_risk | 零信任/IdP AccessLog | `SOURCE_SYNC_WEBHOOK_ACCESS_RISK` |
| data_security | 库表评估 / DLP / 分级 | `SOURCE_SYNC_WEBHOOK_DATA_SECURITY` |
| cloud_security | CSPM / 云配置 Finding | `SOURCE_SYNC_WEBHOOK_CLOUD_SECURITY` |
| threat_intel | MISP / OpenCTI | `SOURCE_SYNC_WEBHOOK_THREAT_INTEL` |
| soc_ops | SLA/例外/人工升级 | `SOURCE_SYNC_WEBHOOK_SOC_OPS` |

| CL | 状态 | 要点 |
|---|---|---|
| CL-1～CL-4 | **已落地** | 深链 / `im/actions` / 按钮 / 六 Bot long-poll |
| CL-5 | **已落地** | ITSM 信封 + `SourceSyncAdapter` |
| CL-6 | 规划 | 生产告警群 + 降噪 |

指挥舱回看：Detect `#/detect`（Incident）· Respond `#/respond-ops`（Ticket）· Identify `#/identify-findings`（Finding）。  
回归：`tools/im-allbots-verify.ps1` · 样本：`docs/samples/im-allbots-risk-inputs.json`。

### 4.7 下一步实施顺序（建议）

1. **IM 生产化（CL-6）**：六 Bot 绑定告警群 chat_id；Secret 进部署系统（勿入库）  
2. **真源按序接入**：code_audit 流水线持续 → access 日志 → data/cloud/TI → soc_ops 值班  
3. **五类 S8 Webhook**：为非 code_audit 路由配置 `SOURCE_SYNC_WEBHOOK_*`  
4. **指挥舱增强**：IM 动作台账专页；Incident 级深链  
5. **外部前置集群**：真 Jira · 真设备下发 · SSO（见 §4.8）  

### 4.8 外部前置与生产门禁（摘要）

**需外部前置才从「占位」变「真」的能力**（节选）：SSO/RBAC · 真实日志 Topic · 真零信任/云写动作 · 真 Jira · **生产 IM 告警群与持续真源** · 真资产图谱 · 真 DLP/设备 API · 真 TI Feed · Memory 向量化 · RTO/RPO 备份联动。  
完整表 → roadmap **§9.2 / §9.3 / §14**。

**共享环境上线前门禁**（roadmap **§9.4**）：

1. SSO/RBAC  
2. 至少一类真实风险数据流（非仅 seed）  
3. 至少一类真 ITSM 或真可回滚封禁动作  
4. Secret 治理（不落前端、可轮换）  
5. 审批/高风险动作审计留痕  
6. CORS 收敛  
7. 存储组件不公网裸奔  

### 4.9 里程碑验收口径

| 里程碑 | 验收标准 |
|---|---|
| MVP | Compose 一键启动；样例生成告警与事件 |
| 数据接入 | 每源有映射、脱敏、运行历史与告警验证 |
| 检测增强 | 核心规则含场景/条件/ATT&CK/误报与聚合 |
| 自动化处置 | 严重事件可派单、通知、SLA、复盘、月报 |
| IM 六类出站 | 六路 `im_sent:{route}`；详情可读；不串台 |
| IM 闭环 CL-1～CL-5 | 深链 / 回流 API / 按钮命令 / ITSM+SourceSync；Detect/Respond/Identify 可回看 |
| 管理汇报 | 关键成功指标、趋势、覆盖、Portfolio 反哺 |
| 指挥舱体验 | 分析师 Detect / 管理层 Govern 主路径可达 |

细则：`docs/test-plan.md` · `docs/evaluation.md` · `docs/im-data-flow.md`。

---

## 5. 当前已交付能力清单（工程视角）

| 能力 | 状态说明 |
|---|---|
| 数据源接入 | API/Webhook、离线、Redis Stream、Kafka、Filebeat/syslog、云日志适配入口 |
| 标准化 | RawEvent → Asset / Identity / Finding / AccessLog |
| code_audit | `code-audit/{tool}`、Owner、聚合、Portal 深链 |
| 检测 | 规则 DSL、评分、聚合、误报、ATT&CK、趋势、共研 / Explain·Draft |
| 事件闭环 | Incident、SLA、例外、复盘、月报、证据篮 |
| IM | 六 Bot 出站 + 入站 poller；`im/actions`；`SourceSyncAdapter` |
| 自动化 | Playbook、审批 HITL、L2/L3、工单/通知台账、ITSM Webhook |
| 智能运营 | UEBA / TI / SOAR 台账；`phase4` 可选组件 |
| 指挥舱 UI | CSF 六域 + Phase D 全项 |
| 占位 API | 合规、防御包、DLP、狩猎、战情室、演练、Memory、Playbook graph |

代码落点：`frontend/src/{shell,phaseD,p0,p1,p2}/` · `backend-go/internal/soc/`（含 `*_placeholders.go`）。

---

## 6. 仓库结构

| 路径 | 说明 |
|---|---|
| `frontend/` | React + TS + Vite 指挥舱 |
| `frontend/src/shell/` | 壳层、场景、角色、hash、Agent 抽屉 |
| `frontend/src/phaseD/` | Phase D 体验面板 |
| `frontend/src/p0\|p1\|p2/` | 六域能力面板 |
| `backend-go/` | Go API / worker |
| `docker-compose.yml` | 本地完整栈 |
| `docs/` | 一文一责文档集 |

---

## 7. 快速启动

### 7.1 Docker（完整栈）

```powershell
cd D:\Projects\security-group\SOC
docker compose up --build -d
docker compose ps
```

改前端后须重建镜像，否则易看到旧 UI：

```powershell
docker compose build soc-frontend
docker compose up -d --no-deps soc-frontend
```

浏览器建议 Ctrl+F5。`phase4` profile 见 `docs/deployment-guide.md`。

### 7.2 本地轻量（不启 Docker）

```powershell
# 加载 SOC/.env 后启动 API（常用 :9012，避免与 Docker :9010 冲突）
cd D:\Projects\security-group\SOC\backend-go
# 确保 IM_* / CODE_AUDIT_PORTAL_* / IM_INBOUND_ENABLED 已在环境中
$env:PORT="9012"; go run ./cmd/soc-api

# UI :9011（代理 /api → 9012）
cd D:\Projects\security-group\SOC\frontend
$env:VITE_API_PROXY="http://127.0.0.1:9012"
npm install
npm run dev -- --host 0.0.0.0 --port 9011
```

勿与 Docker 前端同时占用 `9011`。

| 服务 | 地址 |
|---|---|
| 前端 | `http://localhost:9011` · 风险台账 `#/identify-findings` · 调查台 `#/detect` |
| API | `http://localhost:9010`（Compose）或 `:9012`（本地轻量）· `/health` |
| Portal | `http://localhost:9108`（code_audit） |
| OpenSearch | `http://localhost:9200`（Docker） |
| MinIO Console | `http://localhost:9003`（Docker） |

---

## 8. 验证

```powershell
Invoke-RestMethod http://localhost:9010/health
Invoke-RestMethod http://localhost:9010/api/v1/soc/dashboard

cd D:\Projects\security-group\SOC\backend-go
go test ./...
go vet ./...

cd D:\Projects\security-group\SOC\frontend
npm run build
```

OpenAPI：`docs/openapi.yaml`（`info.version` = 0.5.0）。

---

## 9. 文档索引

| 文档 | 用途 |
|---|---|
| `docs/context-map.md` | 文档职责地图与最小上下文包 |
| `docs/architecture.md` | 架构 / 数据流 / 设计边界（§6.1 IM 分层） |
| `docs/ui-command-cockpit-prototype.md` | UI / IA 原型（§9.7 IM 回看） |
| `docs/im-closed-loop-framework.md` | 六 Bot IM 九阶段与 CL 切片 |
| `docs/im-data-flow.md` / `im-bot-routing.md` | 出站流转与路由命名 |
| `docs/implementation-roadmap.md` | **唯一**实现路径 / 状态 / 门禁 |
| `docs/release-plan.md` | 版本特性与 Demo 公告 |
| `docs/feature-guide.md` | 用户视角功能与工作流 |
| `docs/technology-stack.md` | 技术选型 |
| `docs/openapi.yaml` | HTTP 契约 |
| `docs/changelog.md` | 变更记录 |
| 对接 / Agent / 运维 / 测试 | 见 `docs/context-map.md` |

Agent 接续会话摘要：仓库根目录 `../AGENTS.md` §9。

---

## 10. 安全注意事项

默认配置仅用于本地 MVP / 内部 demo，勿直接暴露共享网络或生产。推送 GitLab 前确认不提交原始日志、Token、AK/SK、OAuth code、用户四要素或生产库导出。

`.gitignore` 已忽略 `imports/*.csv|*.resp`、`backend-go/bin/`、`scripts/` 等；真实日志只提交脱敏样例。
