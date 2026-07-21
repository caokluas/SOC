# SOC 实施路线图

> **文档职责**：UI + 后端融合的**唯一**实现路径 / 落地状态 / 外部前置 / 生产门禁权威。  
> 不写：系统分层与数据流（→ `architecture.md`）、技术选型（→ `technology-stack.md`）、页面交互原型（→ `ui-command-cockpit-prototype.md`）、**按版本发版公告口径**（→ `release-plan.md`）。

## 1. 路线图定位

| 轨 | 内容 | 权威章节 |
|---|---|---|
| **UI 轨** | CSF 指挥舱壳层、六域面板、线框交付、体验分期 | 本文 §2 |
| **后端 / 技术轨** | 数据接入、检测、自动化、智能运营 Phase 0–4 | 本文 §3–§7 |
| **能力状态** | 18 项六域服务能力（有 / 有占位）与占位 API | 本文 §8 |
| **差距与门禁** | 外部前置、生产阻断、下一步顺序 | 本文 §9 |
| **覆盖核对** | 架构 + UI 原型全量能力 ↔ 本文实现/规划（防遗漏） | 本文 §13 |

姊妹文档：`architecture.md`（架构）· `ui-command-cockpit-prototype.md`（UI/IA v0.9）· `technology-stack.md`（技术栈）· `openapi.yaml`（API 契约）。

---

## 2. UI 轨：指挥舱落地

### 2.1 已完成交付

#### 指挥舱 P0

| 模块 | 交付 |
|---|---|
| D-M3/D-M4/D-M9 | Detect 调查工作台：聚合簇、调查画布、规则模板 / 可选 LLM 共研 |
| G-M1/G-M6 | Govern 指挥专区 + KPI 下钻 |
| I-M1/I-M5/I-M7 | Identify 代码仓视图、无 Owner 高危、资产速览 |
| R-M1/R-M2/R-M4 | Respond 处置认领、剧本运行、审批批准/驳回 |

#### 指挥舱 P1

| 模块 | 交付 |
|---|---|
| G-M2/G-M3 | 决策队列 + 策略中心（版本对比 / dry-run） |
| I-M2 | 业务关系卡（系统–资产–身份–仓） |
| P-M1/P-M6/P-M7 | 基线策略包（含影响面）、例外台账、误报/剧本有效性 |
| D-M5/D-M6 | 本地狩猎 + OpenSearch 桥 + UEBA/TI 信号雷达 |
| R-M3/R-M5 | 连接器动作→审批、工单/通知协同（真 API 仍待前置） |
| C-M2/C-M3/C-M4 | 复盘、案例、回流提案（/proposals） |

#### 指挥舱 P2（轻量）

| 模块 | 交付 |
|---|---|
| I-M6 / I-M4 | 数据分级标签、公网攻击面、暴露类 Finding |
| CSPM / XDR | 云配置/暴露与终端向信号聚合（非完整产品） |
| P-M4 | DLP/脱敏/密钥/访问策略目录 + 例外联动 |
| C-M1 / C-M5 / C-M7 | 按业务系统恢复看板、演练台账、对外通报草稿 |

#### 指挥舱深化一版（2026-07-17）

| 模块 | 交付 |
|---|---|
| R-M7 / R-M6 | L2 逐步确认 UX、证据篮上传 |
| G-M8 / G-M5 | AI 治理本地开关、值班表 |
| I-M3 / P-M8 | 身份画像专页、配置漂移视图 |
| D-M9+ | /agent/co-research（可选 LLM） |

线框 ID（W-GV-01 等）见 `ui-command-cockpit-prototype.md` §13；场景 ID 见同文档 §12.1。UI 子模块 G1–C5 与交付编号映射见本文 **§13.1**。

### 2.2 UI 体验分期（原原型 Phase A/B/C）

| 阶段 | 目标 | 状态 |
|---|---|---|
| Phase A 壳层重组 | CSF 六域导航；原模块嵌入对应域 | **已完成** |
| Phase B 调查台与指挥舱 | Detect 调查台 + Govern KPI + Identify 关系卡 | **已完成** |
| Phase C 编排与回流 | Respond L2、Recover 提案回流；真图谱/真下发待前置 | **演示完成**；真下发/真图谱待 §9 |
| Phase D 体验补齐 | W-SH-01 全局 Agent 抽屉、路由化、角色默认页、Explain/Draft、Portal 深链、L3 白名单、雷达热力 | **已实现**（§13.4；`scripts/verify-phase-d.js`） |

本阶段明确不做：一次重写全部后端模型；未审批全自动封禁；聊天机器人作唯一入口。

---

## 3. 技术轨阶段总览

| 阶段 | 建设目标 | 核心交付物 | 当前状态 |
|---|---|---|---|
| Phase 0 - 本地 MVP | Docker 一键启动与端到端闭环 | 前端、Go API、worker、PostgreSQL、Redis、OpenSearch、MinIO、示例数据 | 已完成 |
| Phase 1 - 数据接入标准化 | 关键数据源稳定接入 | Kafka、checkpoint、零信任契约、CMDB/身份/CI/CD/云扫描 | 契约已实现，待接真实 Topic |
| Phase 2 - 检测增强 | 真实数据调优规则 | 规则 DSL、dry-run、聚合、反馈、ATT&CK、趋势 | 底座已实现，待真实数据调优 |
| Phase 3 - 自动化处置 | 工单/通知/剧本/例外/复盘/月报/风险反哺 | ITSM/IM Webhook、Playbook、审批、连接器、SLA | 台账已实现，待接真实 API |
| Phase 4 - 智能运营 | 大规模日志、UEBA、TI、长期审计 | Kafka/Flink、ClickHouse、MISP、Shuffle 等 | MVP 已实现，待接真实组件 |

---

## 4. Phase 1：数据接入标准化

| 任务 | 交付内容 | 验收标准 | 优先级 |
|---|---|---|---|
| CMDB / 资产系统接入 | 资产 ID、业务系统、Owner、国家、环境、敏感/暴露等级 | 90% 以上风险能关联 Owner | P0 |
| 身份 / 账号系统接入 | 用户、部门、角色、权限、高权标记 | 零信任日志可关联身份 | P0 |
| CI/CD 安全扫描接入 | SAST/SCA/Secret/IaC/镜像 Finding | 严重/高危触发 Alert/Incident | **已实现**（code_audit） |
| 零信任网关接入 | SLS→Kafka→访问日志消费 | 可下钻身份/资产/会话/策略 | P0 |
| 敏感数据分级接入 | 库表字段、敏感类型、等级、Owner | 高敏资产风险分自动提升 | P1 |
| 云资产扫描接入 | 公网暴露、出站、安全组、存储桶 | 未审批公网可生成高优先级事件 | P1 |

---

## 5. Phase 2：检测增强

| 任务 | 交付内容 | 验收标准 | 当前状态 |
|---|---|---|---|
| 规则 DSL | 条件、频率、版本、dry-run、发布、聚合窗口 | 可维护/试运行/发布规则 | 已实现 |
| 告警聚合 | 按规则/资产/身份/窗口聚合 | 重复告警明显降低 | 已实现 |
| 误报反馈 | FP/TP 与规则质量统计 | 误报可关闭、可追踪 | 已实现 |
| ATT&CK 覆盖 | tactic/technique/命中 | 可见覆盖空白 | 已实现 |
| 风险趋势 | 新增/开放/关闭/误报/均分 | 月度可汇报 | 已实现 |
| 规则运营 | 真实数据调阈值/白名单 | 核心规则有回归用例、误报率下降 | 测试套件已有，待真实数据 |

---

## 6. Phase 3：自动化处置

| 任务 | 交付内容 | 验收标准 | 当前状态 |
|---|---|---|---|
| SLA 检查 | 超期识别、时长、升级通知 | 严重事件 SLA 可统计 | 已实现 |
| 工单联动 | 本地台账 + 可选 ITSM Webhook | 高风险可派单回写 | 已实现，待真 ITSM |
| 通知联动 | 本地台账 + **KN Chat 六类 Bot 分路由** + 可选通用 Webhook | 高风险按类型进对应机器人；详情可读 | **KN 出站已验证**；待生产群/真源输入（§14） |
| Playbook | 轻量 SOAR、手动/自动、幂等历史 | 常见事件可标准处置 | 已实现 |
| 事件调查台 | 评论、任务、时间线 | 研判与审批历史可见 | 已实现 |
| 审批台账 | 申请/批准/拒绝与范围 | 高风险动作前可校验 | 已实现，待 OA 同步 |
| 动作连接器 | ZT 撤销会话、封禁 IP、MFA 等 | 先审批台账，后真 API | 已实现，待真 API |
| 例外评审 | 接受、补偿、到期复审 | 例外有到期记录 | 已实现 |
| 事件复盘 | 复盘模板与改进项 | 高风险复盘完成率可统计 | 已实现 |
| 月报生成 | 自动汇总运营指标 | 月报材料可复用 | 已实现 |
| 项目风险反哺 | 同步 Portfolio Risk | 进入项目风险管理 | 已实现 |

---

## 7. Phase 4：智能运营

| 能力 | 推荐组件 | 当前实现 | 后续生产化 |
|---|---|---|---|
| 大规模日志流 | Kafka + Flink | 链路模型、API、看板、可选 Compose | 真实 Topic + Flink 任务 |
| 长期日志分析 | ClickHouse / OS / Data Lake | 存储台账、样例查询、可选组件 | TTL、冷热分层、容量监控 |
| UEBA | 可解释基线 | 身份-资产基线与异常发现 | 30–90 天真实日志调优 |
| 威胁情报 | MISP / OpenCTI | IOC 模型与匹配 API | Token 同步任务 |
| 开源 SOAR | Shuffle / TheHive / Cortex | 集成台账与同步 | 真 workflow / case |
| 权限体系 | SSO + RBAC + 审计 | 未实现（生产门禁） | 见 §9.4 |

---

## 8. 六域服务能力实现状态

> 架构职责与数据源设计见 architecture.md §11–§12。本节为**落地状态**权威。

| 域 | 服务能力 | 状态 |
|---|---|---|
| Govern | 风险态势大屏 | **有** |
| Govern | 策略即代码（PCI-DSS 等） | **有（占位）** /govern/compliance/* |
| Govern | 供应链风险雷达 | **有**（轻量） |
| Identify | 业务拓扑可视化 | **有**（关系卡） |
| Identify | 暴露面管理 ASM | **有**（轻量信号） |
| Identify | 漏洞与配置核查 | **有**（Finding / dry-run） |
| Protect | 策略编排中心（WAF/FW/EDR） | **有（占位）** /protect/defense/* |
| Protect | 零信任访问控制 | **有**（台账闭环） |
| Protect | 数据防泄漏 DLP | **有（占位）** /protect/dlp/* |
| Detect | 告警聚合与降噪 | **有** |
| Detect | 智能调查画布 | **有**（时间线+实体） |
| Detect | 威胁狩猎（自然语言） | **有（占位）** POST /detections/hunt |
| Respond | 剧本可视化拖拽编排 | **有（占位）** Playbook graph_json |
| Respond | 自动化工具箱 | **有**（部分动作） |
| Respond | 协同作战 IM 自动拉群 | **有（占位）** war-rooms；**出站告警已接 KN 六 Bot**（§14） |
| Recover | 灾备演练 RTO/RPO | **有（占位）** /recover/drills |
| Recover | AI 记忆库 Memory | **有（占位）** /agent/memory/* |
| Recover | 复盘报告生成 | **有** |

**统计**：有 10 + 有（占位）8；无外部依赖的后端台账已齐。真下发 / 真 IM / 真备份 / 强制 LLM 仍属外部依赖。

### 8.1 深化交付明细（原「现在可做」）

| 项 | 状态 | 说明 |
|---|---|---|
| R-M7 L2 UX | 已完成 | 逐步确认后运行 |
| G-M3 版本对比 | 已完成 | 双栏 snapshot + dry-run |
| R-M6 证据篮 | 已完成 | 列表 + multipart 上传 |
| I-M3 身份画像 | 已完成 | profile API 面板 |
| P-M1 dry-run 影响面 | 已完成 | 基线包批量评估 |
| C-M4 提案 API | 已完成 | /proposals |
| OpenSearch 桥 | 已完成 | /search，失败本地回退 |
| 可选 LLM 共研 | 已完成 | /agent/co-research + LLM_API_* |
| G-M8 / G-M5 / P-M8 | 已完成 | 本地开关、值班表、漂移视图 |
| 8 项占位后端 | 已完成 | 见上表「有（占位）」 |

---

## 9. 差距、前置与生产门禁

### 9.1 当前结论

| 结论 | 说明 |
|---|---|
| 演示可用 | CSF 六域壳 + P0/P1/P2 + 深化 + 占位后端可本地演示 |
| 非完整产品 | CSPM/XDR/真 ITSM/真下发等仍为信号聚合或本地占位 |
| 下一阶段 | 申请外部前置（§9.3）；生产阻断见 §9.4 |

### 9.2 需要外部前置

| 能力 | 前置条件 | 未满足时状态 |
|---|---|---|
| G-M4 SSO/RBAC | IdP、角色映射、会话 | 演示账号/开放模式 |
| 真日志量检测调优 | 真实 Topic / 日志管道 | 样例 + 规则底座 |
| 真零信任 / 云 API 动作 | 厂商凭证、网络、审批策略 | Webhook/本地审批 |
| 真 Jira / IM | JIRA_*；KN `KN_CHAT_*` 已通六 Bot 私聊 Demo | **KN 分路由出站已验证**；待生产告警群、Jira 真 API、war-room 真拉群 |
| 真资产图谱 | CMDB/血缘/服务目录 | 关系卡片 |
| 真数据分级源 | 分类分级系统 | 手工/演示标签 |
| 真 DLP / KMS / 设备下发 | DLP/密钥/WAF/EDR API | 策略/命中台账占位 |
| 完整 CSPM / XDR | 云扫描与 EDR 产品 | 信号聚合轻量视图 |
| 真 TI Feed | MISP/OpenCTI 或商业情报 | 信号雷达样例 |
| 生产 Agent | 模型网关、审计、写动作审批 | 规则模板共研 |
| 正式通报发布 | 法务/公关流程与渠道 | 草稿本地 |
| RTO/RPO 真联动 | 备份系统 API | 演练台账占位 |
| Memory 向量化 | 知识库 / Embedding | 本地 Memory 台账 |

### 9.3 占位后端之后优先（真外部依赖）

| 优先级 | 场景 | 建议前置 |
|---|---|---|
| P0 | 策略编排真下发 | 安全设备写 API、审批制度 |
| P0 | IM / Jira 真协同 | **KN 六 Bot 出站已通**；下一步生产群 chat_id + 真源输入（§14）+ Jira 凭证 |
| P1 | NL 狩猎增强 | 可选 LLM 网关 + 索引治理 |
| P1 | DLP 真命中 / 策略即代码深化 | DLP 产品 + 合规 DSL |
| P2 | RTO/RPO 真联动 | 备份系统 API |
| P2 | Memory 向量化 | 知识库 / Embedding |
| P2 | ASM 产品化 / 图拓扑 | 资产发现探针与图存储 |

### 9.4 生产阻断（共享环境上线前门禁）

1. **SSO/RBAC**（分析师 / 审批人 / 管理员）— 对应架构 §7「API 未鉴权」与 UI G4  
2. **真实日志或风险数据流**（至少一类，非仅 seed）  
3. **至少一类真 ITSM 或真零信任/封禁动作**（可回滚、可审计）  
4. **Secret 治理**（不落前端、可轮换；替换 Compose 默认密码）— 架构 §7  
5. **审计留痕**（审批、高风险动作、配置变更）  
6. **CORS 收敛到可信前端域名** — 架构 §7  
7. **OpenSearch / MinIO / PostgreSQL / Redis 不公网裸奔** — 架构 §7  

安全检查项细则见 `security-and-permissions.md`。

### 9.5 建议实施顺序（下一步）

1. 并行申请前置：Jira/IM 凭证、IdP、一类真实数据源  
2. 真对接完成后再将 R-M5b / 真 ZT 从占位升为 live  
3. ~~Phase D 体验补齐~~ **已完成**（§13.4；`scripts/verify-phase-d.js`）  
4. **外部前置（§9）**：真设备下发 / 真 IM / SSO / 真 SOAR API  
5. 可选：策略发布流、Playbook 拖拽前端对接 `graph_json`

---

## 10. 近期两周建议

| 周期 | 建议任务 | 产出 |
|---|---|---|
| 第 1 周 | CMDB、身份、CI/CD、零信任各接一份脱敏样例 | 字段映射、验收记录 |
| 第 1 周 | GitLab CI 加入依赖/Secret/SAST/镜像扫描 | Finding 接入样例 |
| 第 2 周 | 云资产扫描与数据库四要素管控样例 | 云风险事件闭环样例 |
| 第 2 周 | 配置 ITSM/IM Webhook 测试地址 | 派单/通知演示闭环 |
| Week-2+ | Jira 真对接（配置已在指挥舱占位） | 替代或补充 ITSM_WEBHOOK_URL |
| 第 2 周 | 整理首版 SOC 月报与误报反馈 | 汇报材料与调优清单 |

---

## 11. 风险与依赖

| 风险 | 影响 | 缓解 |
|---|---|---|
| 资产 ID 不统一 | 无法归属 Owner | 先定 CMDB asset_id 规范 |
| 原始日志含敏感字段 | 合规风险 | 接入前脱敏 |
| 规则误报高 | 信任度下降 | 反馈、聚合、白名单 |
| ITSM/IM 未打通 | 自动化停在本地台账 | Webhook 适配后再真系统 |
| 日志量暴涨 | OS/Redis 压力 | 评估 Kafka/ClickHouse/Data Lake |

---

## 12. 里程碑验收口径

| 里程碑 | 验收标准 |
|---|---|
| MVP 验收 | Compose 一键启动；样例能生成告警和事件 |
| 数据接入验收 | 每源有映射、脱敏、运行历史、标准化与告警验证 |
| 检测增强验收 | 核心规则有场景、条件、ATT&CK、误报与聚合 |
| 自动化处置验收 | 严重事件能派单、通知、SLA、复盘、月报 |
| 管理汇报验收 | 看板展示关键成功指标、趋势、覆盖、风险反哺 |
| 指挥舱体验验收 | 分析师 Detect / 管理层 Govern 主路径可达（细则见 test-plan / evaluation） |

详细用例见 `test-plan.md`；发布门禁指标见 `evaluation.md`。

---

## 13. 架构 × UI 原型覆盖核对（防遗漏）

> 对照源：`architecture.md`（建设目标 / §4 模块 / §7 安全 / §8 六域边界）与 `ui-command-cockpit-prototype.md`（G1–C5 子模块 / 场景 ID / 线框 W-* / AI 底座 / 壳层）。  
> 状态取值：**已实现**（演示可用）· **有（占位）** · **规划中**（无外部依赖可做）· **待前置**（需外部系统）· **明确不做（本阶段）**。

### 13.1 UI 子模块 G1–C5 ↔ 交付映射

| UI 子模块 | 映射交付 / 场景 | 状态 | 说明 |
|---|---|---|---|
| G1 指挥态势 | G-M1/G-M6 · `govern-situ` · W-GV-01 | 已实现 | KPI / Critical / SLA / 闭环率 |
| G2 策略中心 | G-M2/G-M3 · `govern-policy` · W-GV-02 | 已实现 | 版本对比 / dry-run；合规策略即代码见 §8 占位 |
| G3 目标管理 | Portfolio Risks + 业务系统字段 · `govern-situ` / Goal 深链 | 已实现（含 Phase D 深链） | 真业务目标库仍可选增强 |
| G4 合规与治理 | G-M4 · security 文档 + Approvals | 待前置 | SSO/RBAC/数据保留；Agent 开关已有 G-M8 |
| G5 平台治理 | Components Health · `govern-metrics` | 已实现 | 含数据源运行；值班表见 G-M5 |
| G6 管理层汇报 | G-M6 · 月报 / Portfolio Sync | 已实现 | |
| I1 资产图谱 | I-M2 · `identify-identity` · W-ID-01 | 已实现（关系卡） | 真图谱 → §9.2 / §9.3 |
| I2 代码资产视图 | I-M1 · `identify-risk` · W-ID-02 | 已实现 | code_audit |
| I3 风险发现台账 | `identify-findings` FindingsLedger | 已实现（Phase D 增强） | — |
| I4 访问与暴露 | I-M4 · `identify-surface` | 已实现（轻量） | |
| I5 数据源可见性 | Data Sources · `identify-surface` | 已实现 | |
| I6 Owner 治理 | 无 Owner 高危清单 | 已实现 | |
| I-M3 身份画像 | `identify-identity` | 已实现 | 深化一版 |
| I-M6 数据分级 | Identify P2 | 已实现（轻量） | 真分级源 → §9.2 |
| I-M7 资产速览 | Identify P0 | 已实现 | |
| P1 基线策略包 | P-M1 · `protect-baseline` · W-PR-01 | 已实现 | 含 dry-run 影响面 |
| P2 防御动作目录 | `protect-actions` DefenseActionCatalog | 已实现（Phase D 专页） | 真下发 → §9 |
| P3 例外管理 | P-M6 · `protect-defense` | 已实现 | |
| P4 有效性监测 | P-M7 | 已实现 | |
| P5 组件与连接器健康 | Health + SOAR Sync（嵌入）+ `protect-actions` SOAR 面板 | 已实现 | 真下发 / 真 SOAR → §9 |
| P-M4 DLP 目录 | `protect-defense` + `/protect/dlp/*` | 有（占位） | |
| P-M8 配置漂移 | Protect 深化 | 已实现 | |
| D1 调查队列 | D-M3 · `detect-workbench` · W-DE-01 | 已实现 | |
| D2 调查画布 | D-M4 · `detect-workbench` | 已实现 | |
| D3 检测工程 | Detection Panel · `detect-engineering` · W-DE-02 | 已实现 | 规则 DSL / dry-run / 发布 |
| D4 信号雷达 | D-M6 · `detect-hunt` / `detect-smartops` | 已实现 | |
| D5 反馈闭环 | Feedback API | 已实现 | |
| D6 Agent 共研 | D-M9 · Ask Agent · Explain/Draft | 已实现（含 Phase D） | Memory → §8 占位 |
| D-M5 狩猎 | `/detections/hunt` + `/search` | 有（占位） | NL 增强 → §9 |
| R1 处置队列 | R-M1 · `respond-ops` · W-RS-01 | 已实现 | |
| R2 剧本编排 | R-M2 · Playbook + `graph_json` | 有（占位） | 拖拽前端 → §9.5 / 后续小版本 |
| R3 半自动执行 | R-M3/R-M7 L2 | 已实现 | |
| R4 自主执行舱 | L3 白名单（`AutonomyL3Panel`）/ L4 | L3 已实现 / L4 明确不做 | L4 未审批全自动封禁 **明确不做** |
| R5 协同通道 | R-M5 · Tickets/Notifications · `respond-collab` | 已实现（台账） | 真 Jira/IM → §9 |
| R6 外部 SOAR | `protect-actions` SoarStatusPanel + SmartOps | 已实现（状态面板） | 真 SOAR API → §9 |
| R-M6 证据篮 | Respond 深化 | 已实现 | |
| C1 恢复看板 | C-M1 · `recover-drill` | 已实现（轻量） | |
| C2 复盘中心 | C-M2 · `recover-review` · W-RC-01 | 已实现 | |
| C3 知识库 | C-M3 + `/agent/memory/*` | 有（占位） | 向量化 → §9 |
| C4 能力回流 | C-M4 `/proposals` | 已实现 | |
| C5 汇报出口 | Monthly + Portfolio | 已实现 | |
| C-M5 演练台账 | `/recover/drills` | 有（占位） | 桌面演练记录 UI 已含轻量 |
| C-M7 对外通报草稿 | Recover P2 | 已实现（草稿） | 正式发布 → §9.2 |

### 13.2 线框 W-* / 场景 ID

| 线框 | 场景 ID（主） | 状态 |
|---|---|---|
| W-GV-01 | govern-situ | 已实现 |
| W-GV-02 | govern-policy | 已实现 |
| W-ID-01 | identify-identity | 已实现 |
| W-ID-02 | identify-risk | 已实现 |
| W-PR-01 | protect-baseline / protect-defense | 已实现 |
| W-DE-01 | detect-workbench | 已实现 |
| W-DE-02 | detect-engineering | 已实现 |
| W-RS-01 | respond-ops | 已实现 |
| W-RC-01 | recover-review / recover-drill | 已实现 |
| W-SH-01 | 全局壳（跨域 Agent 抽屉） | **已实现**（Phase D） |
| — | govern-metrics / identify-surface / detect-hunt / detect-smartops / respond-collab | 已实现 |

### 13.3 架构模块 / 壳层 / AI / 安全（原缺口补齐）

| 来源 | 能力 | 状态 | 落位 |
|---|---|---|---|
| architecture §1/§4 | 统一数据接入 / 标准化 / 风险评分 / 检测 / 闭环 / 智能运营 / 看板 / 指挥舱 IA | 已实现（MVP） | Phase 0–4 |
| architecture §4 #9–12 | 证据、Portfolio、SmartOps、合规与防御台账 | 已实现 / 有（占位） | §6–§8 |
| architecture §6 | code_audit 接入约定 | 已实现 | Phase 1；Portal 深链 **已实现**（Phase D） |
| architecture §7 | API 鉴权 / CORS / 默认密码 / 存储裸奔 / 日志脱敏 | 待前置 / 门禁 | §9.4 已逐条 |
| architecture §8.1–8.6 | 18 项服务能力设计边界 | 见 §8 状态表 | 全覆盖 |
| architecture §9 | 真 ITSM/IM、SOAR、UEBA、TI、日志湖、指挥舱深化 | 待前置 | §7 / §9 |
| UI §3.2 | 顶栏/态势条/底栏/右侧抽屉完整子项 | 已实现（含 W-SH-01） | — |
| UI §4 | 感知/推理/行动/治理/记忆；Ask/Suggest/Explain/Draft | 已实现（Explain/Draft） | Memory → §8 占位 |
| UI §11 | 六角色默认落地与权限裁剪 | **已实现**（演示角色开关） | 真 RBAC → G-M4 |
| UI §12 | 路由化 IA（hash `#/domain[/scenario]`） | **已实现**（Phase D） | — |
| UI §5.3 | CSF 雷达 / 业务热力 / 策略变更审计入口 | **已实现**（Phase D 可视化） | — |
| UI §8.6 | code_audit Portal 深链 | **已实现** | — |
| UI §9.5 | L0–L2 / L3 | L0–L3 已实现 | L4 明确不做 |
| UI §14–§15 | 产品成功标准 / 评审问题 | 规划中（验收引用） | §12 + evaluation；业务拍板跟踪 |

### 13.4 Phase D 体验补齐清单（无外部依赖）

| ID | 项 | 优先级 | 状态 |
|---|---|---|---|
| D-01 | W-SH-01 全局 Agent 抽屉 + 审批铃铛 | P0 | **已实现** |
| D-02 | 六角色默认落地页与权限裁剪（文档+前端开关，非真 SSO） | P0 | **已实现** |
| D-03 | Agent Explain / Draft 控件 | P1 | **已实现** |
| D-04 | code_audit「回到 Portal」深链 | P1 | **已实现** |
| D-05 | 防御动作目录专页（P2）+ 外部 SOAR 状态面板（R6） | P1 | **已实现** |
| D-06 | Findings 风险发现台账增强（I3） | P1 | **已实现** |
| D-07 | G3 目标管理深链增强；Govern 策略变更审计入口强化 | P2 | **已实现** |
| D-08 | 路由化 IA（hash） | P2 | **已实现** |
| D-09 | L3 白名单低风险自动（仍禁 L4） | P2 | **已实现** |
| D-10 | CSF 雷达 / 业务热力可视化增强 | P2 | **已实现** |

验证：`node SOC/scripts/verify-phase-d.js`（源码标记 + `frontend` build，不启 Docker）。

### 13.5 核对结论（2026-07-17，Phase D 后）

| 结论 | 说明 |
|---|---|
| 无遗漏（状态或规划） | 架构 §1/§4/§7/§8/§9 与 UI G1–C5、场景 ID、AI 底座、壳层、角色、路由均已落入本文 §2–§9 或 §13 |
| 最大缺口集群 | §9 真外部依赖（设备下发 / **生产 IM 群与真源** / SSO / 真 SOAR）；KN 六 Bot **出站 Demo 已通**见 §14 |
| 编号说明 | UI 用 G1–C5；交付轨用 G-M*～C-M*；映射以 §13.1 为准 |

---

## 14. KN Chat 六类告警出站（实现路径）

> 权威细则：`kn-chat-data-flow.md`（流转/效果/依赖）· `kn-chat-bot-routing.md`（命名/环境变量）。

### 14.1 状态（2026-07-20）

| 项 | 状态 |
|---|---|
| 六 Bot 分路由 `sendMessage` | **已实现并本地全量验证** |
| 详情告警文案 | **已实现** |
| findings/access-logs 导入后检测 + RunDue | **已实现** |
| code_audit ← Portal capital | **已验证** |
| 生产告警群 / 每路由独立 chat_id | **待配置** |
| 五类真源持续输入（访问/数据/云/情报/运营） | **待对接**（样例已具备） |
| war-room 真拉群 / 双向 Bot 指令 | **未做**（仍占位） |

### 14.2 六路由 ↔ 下一步真实输入

| 路由 | Bot | 下一步真实输入优先 |
|---|---|---|
| `code_audit` | `@code_audit_upload_bot` | Portal `SECURITY_SOC_PUSH` + 业务仓流水线 |
| `access_risk` | `@access_risk_bot` | 零信任/IdP 访问日志流 |
| `data_security` | `@data_security_bot` | 库表评估 / 分级结果 |
| `cloud_security` | `@cloud_security_bot` | CSPM / 云配置扫描 |
| `threat_intel` | `@threat_intel_bot` | MISP/OpenCTI Feed |
| `soc_ops` | `@soc_ops_bot` | 值班群 + SLA/例外运营 |

### 14.3 建议落地顺序

见 `kn-chat-data-flow.md` §6（样例回归 → code_audit 真仓 → access → data → cloud → TI → soc_ops 值班群 → 门禁）。

回归：`tools/kn-allbots-verify.ps1`。

---

## 15. IM 闭环运营框架（上报 → 处置 → 回流 → 工单）

> 权威：`im-closed-loop-framework.md`。以 **code_audit** 做深，其余五类复用同一九阶段机。

### 15.1 九阶段（摘要）

`S0 信号 → S1 接入 → S2 成案 → S3 IM 上报 → S4 IM 处置 → S5 回流 SOC → S6 工单 → S7 验证关闭 → S8 反哺`

### 15.2 成熟度

| 段 | 状态 |
|---|---|
| S0–S3 出站（六 Bot + 详情） | **已验证** |
| S4–S5 IM 命令/按钮回流 | **CL-1～CL-4 六路由共用** |
| S6 本地 Ticket + ITSM Webhook | **CL-5 ITSM 已接** |
| S7–S8 源系统反哺 | **`SourceSyncAdapter`**：Portal（code_audit）+ 五类 Webhook 插槽 |

### 15.3 落地切片

`CL-1 深链` → `CL-2 im/actions API` → `CL-3/4 KN 按钮与命令` → `CL-5 Ticket/Portal 同步` → `CL-6 生产群与降噪`  

code_audit 优先走通后，复制到 access / data / cloud / threat / soc_ops。

---

## 16. 安全专项最小闭环（2026-07-20 优先级调整）

> 在 KN/code_audit 通用闭环之上，优先支撑 **四类专项材料录入与聚合**；真源对接可后续增量接入。

| 优先级 | 板块 | 轨道 `track` | 系统能力（已实现） | 待你同步的材料 |
|---|---|---|---|---|
| **P0** | 业务安全 · 资金安全 | `fund_security` | 粘贴/TSV/JSON Lines 导入；按 `business_system` **业务风险地图** | 渗透攻击链、验证结论、业务系统清单 |
| **P0** | 业务安全 · 链路安全 | `link_security` | 按 `country` 追踪实施/验收；状态 open/in_progress/verified | 各国 APP 加固、API 验签进展与验收件 |
| **P1** | 办公安全 · 零信任 | `zero_trust` | 审计报告台账；预留 ZT 日志 `data-sources` 对接说明 | 审计报告草稿、日志字段映射（后续打通） |
| **P1** | 生产安全 | `production_security` | `sub_kind`: `db_sensitive` / `egress_control`；**cluster_key 聚类** | 库表扫描输出、出站策略/流量控制台粘贴 |

**API**

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/v1/soc/programs/artifacts?track=&business_system=&cluster_key=&country=&phase=` | 材料台账列表 |
| GET | `/api/v1/soc/programs/artifacts/{id}` | 单条查询 |
| POST | `/api/v1/soc/programs/artifacts` | 新增单条 |
| PUT | `/api/v1/soc/programs/artifacts/{id}` | 全量替换（表单保存） |
| PATCH | `/api/v1/soc/programs/artifacts/{id}` | 部分更新 |
| DELETE | `/api/v1/soc/programs/artifacts/{id}` | 删除单条 |
| POST | `/api/v1/soc/programs/artifacts/bulk-delete` | 按分组/聚类批量删 `{track,business_system\|country\|phase\|cluster_key}` |
| POST | `/api/v1/soc/programs/artifacts/bulk-update` | 按筛选批量改 `{filter,update}` |
| POST | `/api/v1/soc/programs/artifacts/import` | JSON `{items:[]}` 批量 |
| POST | `/api/v1/soc/programs/artifacts/paste` | 控制台粘贴 |
| GET | `/api/v1/soc/programs/fund-security/summary` | BIZ/贷后/财务 风险地图（聚合只读） |

**材料**：`docs/samples/fund-security-risk-map-import.json` · `docs/fund-security-program-guide.md`（映射 `业务安全板块-核心/资金安全项目`）

**UI**：`#/govern/govern-programs` · 组件 `ProgramTracksPanel.tsx`

**代码**：`models_programs.go` · `store_programs.go` · `api_programs.go` · `programs_parse.go`

**下一步（材料到位后）**：按专项批量 paste/import → 核对 summary 分组 → 高风险项可选升格 Finding/Incident 走 IM 闭环。
