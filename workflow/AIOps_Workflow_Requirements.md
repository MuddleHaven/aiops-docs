# AIOps Workflow Requirements

## 1. 文档目标

本文档用于定义 `aiops-workflow` 项目的职责边界、MVP 需求范围、核心对象模型、输入输出契约与实施优先级。

这份文档基于主架构文档中对三层项目域的划分，重点回答以下问题：

1. `aiops-workflow` 在整体 AIOps 架构中的定位是什么。
2. `aiops-workflow` 在 MVP 阶段必须交付哪些能力。
3. 它和 `aiops-platform`、`aiops-tools` 的边界如何清晰切开。
4. 如何保证 workflow 输出稳定、可解释、可评测、可演进。

本文档默认面向 `workflow-first` 路线，而不是 multi-agent 优先路线。

## 2. 项目定位

### 2.1 核心定义

`aiops-workflow` 负责“如何做诊断、如何组织证据、如何输出结构化 RCA 与建议”。

它不是平台主控层，也不是底层工具适配层，而是位于两者之间的智能工作流中枢。

其核心职责是：

- 接收 `aiops-platform` 提供的 Incident 上下文。
- 按既定 workflow 执行诊断、检索与推理。
- 调用 `aiops-tools` 暴露的查询类工具补齐证据。
- 输出平台可直接消费的结构化结果。

### 2.2 不负责的事情

`aiops-workflow` 明确不负责：

- 持有 `Incident` 主状态
- 审批流转
- 执行动作控制
- 页面展示
- 直接对生产环境进行写操作
- 直接耦合底层监控系统协议细节

### 2.3 核心设计原则

- Workflow 优先于 multi-agent
- 查询能力可以直接调用 tools
- 执行能力必须回到平台决策
- 输出必须结构化，不能只返回自由文本
- 所有 workflow 都要可版本化、可回放、可评测

## 3. 目标用户与调用方

### 3.1 直接调用方

- `aiops-platform`
- 后续可能的复盘任务调度器
- 评测回归任务调度器

### 3.2 主要使用者

- AI 工程师：维护 workflow、Prompt、Schema、评测集
- 平台后端工程师：对接调用契约与任务状态
- SRE / 运维专家：审核知识库、诊断逻辑、建议动作质量

## 4. MVP 成功标准

MVP 阶段，`aiops-workflow` 至少应达到以下结果：

1. 可稳定执行 `incident_diagnosis` 工作流。
2. 可调用 2-3 个查询类工具补齐证据。
3. 可返回统一格式的 `RCA Result` 与 `actions`。
4. 输出结果可被平台直接落库和展示。
5. 具备基础评测回归能力，避免工作流更新后质量失控。

建议的 MVP 指标：

| 指标 | 说明 | MVP 目标 |
| --- | --- | --- |
| Workflow 成功率 | 工作流执行完成率 | 高于 95% |
| Schema 合法率 | 输出满足平台 schema 的比例 | 高于 99% |
| 诊断 P95 时延 | 从触发到输出结果的时延 | 小于 60 秒 |
| 引证覆盖率 | 结果中包含有效 evidence 的比例 | 高于 90% |
| RCA Top1 准确率 | 在评测集上的首选结论命中率 | 持续追踪 |

## 5. 范围边界

### 5.1 MVP 必须范围

- Workflow 注册与版本管理
- `incident_diagnosis` 工作流
- `action_planning` 工作流
- 查询类工具编排
- RAG 检索策略基础能力
- 结构化输出 schema
- 任务状态查询
- 评测样例与回归检查
- 失败与降级策略

### 5.2 MVP 可选增强

- `postmortem_summary` 工作流
- Prompt 模板变量管理界面
- 多知识域检索重排
- 更细粒度的步骤级 tracing

### 5.3 明确不在 MVP

- Multi-agent 编排框架
- 执行类动作直连生产系统
- 复杂可视化管理后台
- 全自动根因图谱推理平台
- 训练/微调平台

## 6. 核心对象模型

建议 `aiops-workflow` 先围绕以下对象建模：

| 对象 | 说明 | 关键字段 |
| --- | --- | --- |
| `WorkflowDefinition` | 工作流定义 | id, name, purpose, owner |
| `WorkflowVersion` | 工作流版本 | version, status, schema_ref, prompt_refs |
| `SkillDefinition` | 可复用能力单元 | id, workflow_ref, input_schema, output_schema |
| `PromptAsset` | Prompt 模板资产 | id, version, variables, content |
| `KnowledgeCollection` | 知识集合 | id, source_type, tags, index_ref |
| `RetrievalPolicy` | 检索策略 | top_k, filters, rerank_policy |
| `ToolContract` | 工具调用契约 | tool_name, input_schema, output_schema |
| `WorkflowRun` | 一次工作流执行实例 | run_id, workflow_id, status, started_at |
| `WorkflowStepRun` | 步骤运行实例 | step_name, tool_name, status, duration_ms |
| `EvaluationCase` | 评测样例 | case_id, input, expected_output |
| `SchemaContract` | 输出契约 | fields, required, enum_constraints |

## 7. MVP 功能需求

## 7.1 Workflow Registry

### 7.1.1 能力目标

系统应能管理工作流定义与版本，而不是把 workflow 当成散乱脚本。

### 7.1.2 必须能力

- 注册 workflow
- 维护 workflow 名称、用途、负责人
- 管理版本状态
  - draft
  - active
  - deprecated
- 记录 workflow 依赖的 Prompt、Schema、Knowledge、Tools

### 7.1.3 验收标准

1. 可查看某个 workflow 的当前生效版本。
2. 更新 workflow 时可保留历史版本。
3. 平台调用时可明确命中哪个版本。

## 7.2 `incident_diagnosis` 工作流

### 7.2.1 目标

这是 MVP 最核心的工作流，用于根据 Incident 上下文给出结构化诊断结果。

### 7.2.2 输入要求

最小输入建议包含：

- `incident_id`
- `title`
- `severity`
- `service`
- `env`
- `fingerprint`
- `time_window`
- `recent_changes`
- `related_incidents`
- `topology`

### 7.2.3 执行步骤

建议固定为以下链路：

1. 解析 Incident 上下文
2. 决定需要调用的查询工具
3. 调用 metrics / logs / k8s / changes 查询
4. 调用 RAG 补充 SOP 和历史案例
5. 汇总证据并生成候选结论
6. 输出结构化 RCA

### 7.2.4 输出要求

必须返回：

- `summary`
- `rca.conclusion`
- `rca.confidence`
- `rca.impact_scope`
- `rca.evidence[]`
- `actions[]`

### 7.2.5 验收标准

1. 输出必须满足平台约定 schema。
2. 结果中至少包含一条 evidence。
3. 低置信度时不能输出高风险自动执行模式。

## 7.3 `action_planning` 工作流

### 7.3.1 目标

根据已形成的 RCA 与上下文，给出可执行但受控的动作建议。

### 7.3.2 必须能力

- 根据 RCA 生成动作建议列表
- 为每个动作给出风险等级
- 为每个动作给出推荐模式
  - `auto`
  - `approval`
  - `suggest_only`
- 提供回滚建议

### 7.3.3 约束

- 仅输出建议，不直接执行
- 不允许 workflow 自行持有执行凭证
- `confidence` 低于阈值时自动降级

### 7.3.4 验收标准

1. 每个动作都带 `risk` 和 `mode`。
2. 中高风险动作默认不能为 `auto`。
3. 建议动作可以被平台直接渲染展示。

## 7.4 `postmortem_summary` 工作流

### 7.4.1 目标

该工作流建议作为 P1 能力，用于在 Incident 关闭后生成复盘初稿。

### 7.4.2 建议输出

- Incident summary
- 关键时间点摘要
- 主要证据
- 初步改进建议
- 待人工确认的问题清单

### 7.4.3 MVP 处理方式

MVP 可先保留设计，不强制交付。

## 7.5 Prompt 资产管理

### 7.5.1 必须能力

- Prompt 版本化
- Prompt 与 workflow 绑定
- 变量占位支持
- 记录 Prompt 更新时间和负责人

### 7.5.2 设计要求

- Prompt 不应散落在代码或平台表单中不可追踪地修改
- Prompt 变更必须可回溯，并可在评测中验证

## 7.6 RAG 管理能力

### 7.6.1 范围

`aiops-workflow` 需要管理“如何检索”，不一定亲自实现底层向量数据库服务。

### 7.6.2 MVP 必须能力

- 管理知识集合引用
- 定义检索过滤条件
- 定义召回数量 `top_k`
- 定义是否进行 rerank
- 返回引用证据

### 7.6.3 知识来源建议

- SOP 文档
- 历史 Incident / Postmortem
- 变更处理手册
- 常见故障 FAQ

### 7.6.4 验收标准

1. workflow 输出中能引用知识来源。
2. 检索异常时可降级，不致使整体任务崩溃。

## 7.7 Tools 编排能力

### 7.7.1 必须支持的查询类工具

- `query_metrics`
- `query_logs`
- `query_k8s`
- `query_changes`

### 7.7.2 调用原则

- workflow 只调用标准化工具契约
- 不直接关心 Prometheus / Loki / K8s 原生协议差异
- 工具异常要回传为结构化错误

### 7.7.3 明确禁止

- 直接调用生产执行接口
- 绕过平台审批去执行变更
- 将 tool 返回的自由文本直接当最终结论

### 7.7.4 验收标准

1. tools 调用记录可关联到 workflow run。
2. tool 错误会降低置信度并触发降级逻辑。

## 7.8 输出 Schema 契约

### 7.8.1 必须能力

- 明确定义 output schema
- 校验 required 字段
- 校验枚举字段
- 拒绝不合法输出进入平台主链路

### 7.8.2 核心字段建议

```json
{
  "summary": "数据库连接池耗尽导致 API 延迟升高",
  "rca": {
    "conclusion": "连接池配置与突发流量不匹配",
    "confidence": 0.86,
    "impact_scope": ["api-gateway", "order-service"],
    "evidence": [
      {"type": "metric", "ref": "promql://latency_p95"},
      {"type": "log", "ref": "loki://timeout_errors"}
    ]
  },
  "actions": [
    {
      "name": "临时扩容 order-service",
      "risk": "low",
      "mode": "auto",
      "rollback": "scale down deployment/order-service"
    }
  ]
}
```

### 7.8.3 验收标准

1. schema 校验失败时返回明确错误码。
2. 平台侧不需要解析自由文本才能使用结果。

## 7.9 运行态与任务管理

### 7.9.1 必须能力

- 异步任务运行
- 任务状态查询
- 步骤级状态展示
- 超时控制
- 取消任务

### 7.9.2 任务状态

- `queued`
- `running`
- `succeeded`
- `failed`
- `timeout`

### 7.9.3 验收标准

1. 平台可以查询任务当前状态。
2. 失败任务能返回失败原因。
3. 超时任务进入明确降级路径。

## 7.10 评测与回归能力

### 7.10.1 必须能力

- 保存评测样例
- 定义期望输出或期望结论
- workflow 更新后可批量回归
- 对比关键指标变化

### 7.10.2 核心评测维度

- RCA 是否正确
- evidence 是否充分
- schema 是否合法
- 动作建议是否越权
- 置信度是否异常偏高

### 7.10.3 验收标准

1. 每次重要 workflow 变更前后都能跑回归集。
2. 回归结果可沉淀为版本发布依据。

## 7.11 可观测性与审计

### 7.11.1 必须记录的信息

- workflow 名称与版本
- run_id
- 输入摘要
- tools 调用记录
- token / cost 使用量
- 执行耗时
- 输出摘要
- 错误信息

### 7.11.2 边界说明

`aiops-workflow` 可以记录 workflow 运行审计，但 Incident 主审计仍由 `aiops-platform` 持有。

## 7.12 失败与降级策略

### 7.12.1 必须实现

- LLM 超时降级
- tool 不可用降级
- 检索失败降级
- schema 不合法降级

### 7.12.2 建议行为

- 返回部分结果 + 低置信度标记
- 阻断自动动作建议
- 明确提示人工接管

### 7.12.3 错误码建议

- `AI_TIMEOUT`
- `TOOL_UNAVAILABLE`
- `SCHEMA_INVALID`
- `RAG_UNAVAILABLE`
- `WORKFLOW_FAILED`

## 7.13 安全要求

### 7.13.1 必须约束

- 不保存生产执行凭证
- 不直接调用写操作工具
- 不暴露敏感原始凭证到 Prompt
- 对输入上下文中的敏感字段进行最小化传递

### 7.13.2 合规要求

- workflow 运行日志可追踪
- Prompt 与知识来源可审计
- 结果生成过程可解释

## 8. 与平台的接口要求

### 8.1 诊断触发接口

- 平台触发 `incident_diagnosis`
- workflow 返回 `task_id`
- 平台异步查询结果

### 8.2 结果接口要求

成功结果必须包含：

- `task_id`
- `incident_id`
- `status`
- `summary`
- `rca`
- `actions`

### 8.3 会话追问能力

建议预留人工追问接口，用于平台详情页的补充诊断，但不应影响主 workflow 结果的结构化输出约束。

## 9. 推荐的仓库内文档与资产结构

如果后续 `aiops-workflow` 独立成仓，可参考如下结构：

```text
workflow/
  AIOps_Workflow_Requirements.md
  workflows/
    incident_diagnosis/
    action_planning/
    postmortem_summary/
  prompts/
  schemas/
  evals/
  knowledge/
```

## 10. 分阶段实施建议

### Phase 1

- 打通 `incident_diagnosis`
- 接入 `query_metrics`、`query_logs`、`query_k8s`
- 固化结构化 RCA schema

### Phase 2

- 增加 `action_planning`
- 完善 RAG 引用质量
- 建立评测回归集

### Phase 3

- 增加 `postmortem_summary`
- 引入更细粒度版本治理
- 评估局部 agent 化是否有必要

## 11. 结论

`aiops-workflow` 在整个架构中的价值，不是“单独做一个 AI 服务”，
而是把诊断能力沉淀成稳定、可复用、可评测的 workflow 资产。

如果 `aiops-platform` 解决的是“事件怎么流转”，
`aiops-tools` 解决的是“数据怎么查、动作怎么控”，
那么 `aiops-workflow` 解决的就是：

- 证据如何组织
- 诊断如何形成
- 结果如何结构化输出

因此它的第一优先级不是做复杂 agent 协作，
而是先把 `incident_diagnosis -> evidence -> RCA -> actions`
这条链路做稳定、做可审计、做可回归。
