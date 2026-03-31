# AIOps Platform Requirements

## 1. 文档目标

本文档用于定义 `aiops-platform` 项目的职责边界、MVP 范围、核心对象模型、主要页面能力、接口要求与非功能要求。

本文档重点回答以下问题：

1. `aiops-platform` 在三层架构中究竟负责什么。
2. 平台层为什么必须作为唯一主控层存在。
3. MVP 阶段平台必须先交付哪些核心能力。
4. 平台如何与 `aiops-workflow` 和 `aiops-tools` 协同。

本文档是平台层的正式需求清单，和 `workflow/`、`tools/` 下的文档共同构成当前架构的三层需求基线。

## 2. 项目定位

### 2.1 核心定义

`aiops-platform` 负责“事件如何流转、谁来审批、结果如何展示”。

它是整个系统的唯一主控层，持有核心业务状态，负责对事件、Incident、审批、执行、审计和展示进行统一编排。

平台层的核心职责是：

- 接收并管理 Incident 主流程
- 维护 Incident 生命周期与状态机
- 触发 `aiops-workflow` 获取结构化 RCA 和动作建议
- 调用 `aiops-tools` 执行查询代理和受控动作
- 提供页面、接口、审计、审批、时间线与运营视图

### 2.2 不负责的事情

`aiops-platform` 明确不负责：

- 具体 AI 工作流编排细节
- Prompt 管理、RAG 检索策略管理
- 直接对接底层 Prometheus / Loki / K8s / Ansible 协议
- 直接持有工具查询逻辑和执行器实现细节

### 2.3 为什么平台层必须独立

平台层必须独立存在，原因在于：

- Incident 主状态不能放在 workflow 层
- 审批和执行控制不能放在 AI 层
- 页面与业务对象不能散落在工具层
- 审计、时间线、审批、展示必须有统一业务中枢

如果没有独立平台层，系统会很快变成：

- 状态分散
- 页面和业务逻辑耦合不清
- AI 建议与真实执行缺少边界
- 审计链路断裂

## 3. 与其他两层的边界关系

### 3.1 与 `aiops-workflow` 的关系

平台层调用 `aiops-workflow` 获取：

- 结构化 RCA
- 证据链
- 影响范围
- 动作建议

但平台层始终保留以下控制权：

- Incident 状态是否推进
- 动作是否展示
- 动作是否需要审批
- 动作是否允许执行

### 3.2 与 `aiops-tools` 的关系

平台层调用 `aiops-tools` 获取：

- 受控查询结果
- 动作执行结果
- 工具调用与执行审计关联信息

但平台层始终保留以下业务控制：

- 谁可以发起执行
- 哪些动作需要审批
- 哪些结果进入时间线
- 哪些数据进入页面展示和 KPI 统计

### 3.3 平台层的绝对职责

只有平台层应当持有以下对象的业务主状态：

- `incident`
- `incident_timeline`
- `ai_rca_result`
- `remediation_action`
- `approval_record`
- `audit_log`

## 4. 目标用户

### 4.1 一线响应人

- NOC / L1 值班人员
- SRE / 运维工程师
- 应用负责人

核心诉求：

- 快速看到当前事件
- 快速进入 Incident 详情
- 快速理解 RCA、影响和建议动作
- 快速执行或提交审批

### 4.2 高级响应人与负责人

- L2 / L3 工程师
- Incident Commander
- 运维负责人

核心诉求：

- 管理重大 Incident
- 分配责任人
- 查看完整时间线
- 审核执行动作和审批状态

### 4.3 安全与审计角色

- 安全管理员
- 审批人
- 审计人员

核心诉求：

- 看到谁提了什么动作
- 看到谁审批了什么动作
- 看到执行结果与回滚入口
- 看到完整可追踪审计链路

## 5. MVP 成功标准

MVP 阶段，平台层至少应达到以下结果：

1. 能形成统一 Incident 主流程。
2. 能稳定维护 Incident 状态机。
3. 能展示结构化 RCA 与动作建议。
4. 能对动作执行进行审批与控制。
5. 能沉淀统一 Timeline 与审计记录。
6. 能输出基础运营指标。

建议的 MVP 指标：

| 指标 | 说明 | MVP 目标 |
| --- | --- | --- |
| Incident creation success rate | Incident 创建成功率 | 高于 99% |
| Timeline completeness | 时间线事件沉淀完整率 | 100% |
| Approval trace completeness | 审批链可追踪率 | 100% |
| RCA display availability | RCA 展示可用率 | 高于 99% |
| Action execution traceability | 动作执行可回溯率 | 100% |
| Dashboard data availability | 指标看板可用率 | 高于 99% |

## 6. 范围边界

### 6.1 MVP 必须范围

- Incident API
- Incident 状态机
- 事件列表页
- Incident 详情页
- Timeline
- RCA 结果展示卡
- 动作建议展示与执行面板
- 审批流基础能力
- 审计中心基础能力
- KPI / Dashboard 基础能力

### 6.2 MVP 可选增强

- 更强的筛选与搜索
- 更丰富的批量操作
- 更细粒度的角色与权限视图
- 通知中心
- Postmortem 入口联动

### 6.3 明确不在 MVP

- 完整 CMDB 平台
- 完整 ITSM / 工单替代
- 高复杂度多租户商业化后台
- 深度 AI 配置管理界面
- 复杂可视化编排中心

## 7. 核心设计原则

- 平台层是唯一主控层
- 状态和展示必须统一归平台持有
- AI 结果必须结构化接入平台
- 执行动作必须经过平台确认或审批
- 所有关键操作都要进入 Timeline 和审计
- 页面围绕 Incident 主流程组织，而不是围绕底层技术组件组织

## 8. 核心对象模型

建议平台层至少围绕以下对象建模：

| 对象 | 说明 | 关键字段 |
| --- | --- | --- |
| `Incident` | 主事件实体 | incident_id, title, status, severity, service, owner |
| `IncidentTimeline` | 时间线事件 | incident_id, event_type, payload, operator |
| `IncidentContext` | 事件上下文摘要 | service, env, topology, recent_changes |
| `AIRCAResult` | AI 诊断结果 | conclusion, confidence, impact_scope, evidence |
| `RemediationAction` | 建议动作或待执行动作 | action_id, risk, mode, approval |
| `ApprovalRecord` | 审批记录 | approval_id, approver, status, comment |
| `AuditLog` | 审计记录 | trace_id, actor, resource, operation, result |
| `DashboardMetric` | 指标聚合对象 | metric_name, value, time_range |

## 9. MVP 功能需求

## 9.1 Incident API

### 9.1.1 能力目标

平台层必须作为 Incident 的统一入口和唯一事实来源。

### 9.1.2 基础能力

- 创建 Incident
- 查询 Incident
- 更新 Incident 状态
- 关闭 Incident
- 重开 Incident
- 触发诊断
- 追加时间线事件

### 9.1.3 验收标准

1. Incident 状态只能通过平台层接口变更。
2. 所有状态变更都能被审计。
3. Incident 可以和 AI 结果、审批、执行记录关联。

## 9.2 Incident 状态机

### 9.2.1 推荐状态

- `new`
- `triaged`
- `diagnosing`
- `remediating`
- `resolved`
- `closed`

### 9.2.2 必须能力

- 校验合法状态跳转
- 拒绝非法状态变更
- 状态变更写入时间线
- 状态变更写入审计

### 9.2.3 验收标准

1. 平台能明确控制状态机流转。
2. 不能绕过平台直接改 Incident 主状态。

## 9.3 事件列表页

### 9.3.1 页面目标

这是平台的主入口，用于让用户快速掌握当前故障状态。

### 9.3.2 必须展示内容

- Incident 标题
- 状态
- 严重级别
- 服务
- 责任人
- 开始时间
- 最近更新时间
- 是否有 RCA
- 是否有待审批动作

### 9.3.3 必须操作

- 筛选
- 搜索
- 进入详情页
- 快速确认接手或关闭

## 9.4 Incident 详情页

### 9.4.1 页面目标

这是平台最核心的业务页面，应承载大部分响应、诊断查看、审批和执行入口。

### 9.4.2 页面模块建议

- 基础信息卡
- 状态与责任人
- RCA 卡片
- 证据链引用
- 影响范围
- 动作建议与执行面板
- Timeline
- 审批记录
- 审计摘要

### 9.4.3 必须操作

- 变更状态
- 触发诊断
- 查看和刷新 RCA
- 发起审批
- 执行动作
- 查看执行结果

## 9.5 RCA 展示模块

### 9.5.1 平台职责

平台层不负责生成 RCA，但必须负责承接、展示和落库 `aiops-workflow` 返回的结构化结果。

### 9.5.2 必须展示内容

- 诊断结论
- 置信度
- 影响范围
- 证据链
- 建议动作列表

### 9.5.3 验收标准

1. 平台无需解析自由文本即可展示核心结果。
2. 低置信度结果有明确提示。

## 9.6 动作建议与执行控制台

### 9.6.1 平台职责

平台层负责展示建议动作，并决定这些动作是：

- 仅展示
- 需要审批
- 可以执行

### 9.6.2 必须展示内容

- 动作名称
- 风险等级
- 模式 `auto | approval | suggest_only`
- 当前审批状态
- 当前执行状态
- 回滚入口提示

### 9.6.3 必须操作

- 提交审批
- 审批通过 / 驳回
- 执行动作
- 查看执行结果

### 9.6.4 验收标准

1. 平台能统一展示动作清单。
2. 平台能和 `aiops-tools` 的执行结果正确关联。
3. 高风险动作不会绕过平台控制直接执行。

## 9.7 审批流

### 9.7.1 MVP 范围

平台层需要有基础审批能力，但不必一开始就做复杂 BPM 引擎。

### 9.7.2 必须能力

- 为动作创建审批请求
- 指定审批人
- 审批通过
- 审批驳回
- 记录审批意见
- 审批记录进入 Timeline 和审计

### 9.7.3 验收标准

1. 动作执行前能校验审批状态。
2. 审批记录可在 Incident 详情页查看。

## 9.8 Incident Timeline

### 9.8.1 目标

Timeline 是平台层的关键基础设施，是 Incident 回放、审计联动和复盘的核心数据源。

### 9.8.2 必须进入 Timeline 的事件

- Incident 创建
- 状态变更
- 诊断触发
- RCA 结果写入
- 动作建议更新
- 审批发起
- 审批通过 / 驳回
- 执行动作
- 执行成功 / 失败
- Incident 关闭 / 重开

### 9.8.3 验收标准

1. 关键动作不会遗漏时间线记录。
2. Timeline 可作为 Incident 回放依据。

## 9.9 审计中心

### 9.9.1 平台职责

平台层负责持有业务主审计记录，并与工具层、workflow 层的执行记录关联。

### 9.9.2 必须支持

- 按 `incident_id` 查询
- 按操作人查询
- 按动作类型查询
- 按时间范围查询
- 导出审计结果

### 9.9.3 必须记录内容

- 操作人
- 资源对象
- 操作类型
- 请求摘要
- 结果摘要
- 时间
- trace 关联信息

## 9.10 Dashboard 与 KPI

### 9.10.1 MVP 范围

平台层需要提供最基础的运营指标视图。

### 9.10.2 建议指标

- Incident 数量趋势
- MTTA
- MTTR
- RCA 可用率
- 动作执行成功率
- 审批通过率
- 自动恢复率

### 9.10.3 目标用户

- 运维负责人
- 团队负责人
- 项目管理者

## 10. 页面与信息架构建议

## 10.1 首批页面建议

- Incident List
- Incident Detail
- Action Review Panel
- Audit Center
- Dashboard

### 10.2 页面组织原则

- Incident 是主导航中心
- RCA 和动作展示贴着 Incident 展开
- 审批和执行入口贴着动作展开
- 审计与 Dashboard 属于辅助管理视角

## 11. 与 `workflow` 的接口要求

### 11.1 触发诊断

平台需要能向 `aiops-workflow` 发起诊断请求。

建议接口语义：

- `POST /api/v1/incidents/{incident_id}/diagnose`

### 11.2 查询任务结果

平台需要支持异步拿回任务结果。

建议接口语义：

- `GET /api/v1/ai/tasks/{task_id}`

### 11.3 平台侧处理要求

- 平台负责把 workflow 返回结果落库
- 平台负责把 workflow 返回结果映射到页面对象
- 平台负责把关键结果写入 Timeline

## 12. 与 `tools` 的接口要求

### 12.1 查询类能力

平台可在必要时通过平台代理方式调用工具层，但不建议平台散落直接调用底层系统。

### 12.2 执行类能力

平台需要通过 `aiops-tools` 发起执行请求，并接收结构化结果。

建议接口语义：

- `POST /api/v1/incidents/{incident_id}/actions/{action_id}/execute`

### 12.3 平台侧处理要求

- 校验审批状态
- 校验执行条件
- 接收执行结果
- 写入 Timeline 和审计

## 13. 非功能要求

### 13.1 安全性

- 高风险动作必须审批
- 审计日志完整留痕
- 权限分层明确

### 13.2 可用性

- Incident 主流程高可用
- 查询与展示路径稳定
- 关键详情页响应可接受

### 13.3 可维护性

- 业务对象清晰
- 与 workflow/tools 边界清晰
- 平台逻辑不要下沉到工具层或 AI 层

### 13.4 可扩展性

- 可增加更多 Dashboard 视图
- 可增加更多审批规则
- 可增加更多动作类型与展示模块

## 14. 分阶段实施建议

### Phase 1

- Incident API
- Incident 状态机
- Incident List / Detail 基础页
- RCA 展示卡
- Timeline

### Phase 2

- 动作建议展示
- 执行控制台
- 基础审批流
- 基础审计中心

### Phase 3

- Dashboard
- 更强的筛选与检索
- 更强的审批与审计能力
- 和 Postmortem / Knowledge 回流联动

## 15. 推荐的仓库内文档与资产结构

如果后续 `aiops-platform` 独立成仓，可参考如下结构：

```text
platform/
  AIOps_Platform_Requirements.md
  pages/
  api-contracts/
  state-machine/
  approval/
  audit/
```

## 16. 结论

`aiops-platform` 的本质不是一个“展示层前端 + 普通后端 API”，而是整个系统的业务主控中枢。

如果 `aiops-workflow` 负责“如何做诊断”，`aiops-tools` 负责“如何安全地查和做”，那么 `aiops-platform` 负责的就是：

- 事件如何进入主流程
- 状态如何被维护
- RCA 如何被承接和展示
- 动作如何被审批和执行
- 所有结果如何进入时间线与审计

因此，平台层的第一优先级不是做最复杂的 AI 管理后台，而是先把以下主链路做稳：

- Incident
- 状态机
- RCA 承接
- 动作审批执行
- Timeline
- Audit

只有这条业务主链路稳住，workflow 和 tools 的价值才能真正落地。
