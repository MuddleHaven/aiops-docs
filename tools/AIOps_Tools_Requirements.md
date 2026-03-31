# AIOps Tools Requirements

## 1. 文档目标

本文档用于定义 `aiops-tools` 项目的职责边界、MVP 范围、核心能力、输入输出契约与非功能要求。

本文档重点回答以下问题：

1. `aiops-tools` 在整体 AIOps 架构中的定位是什么。
2. 为什么 `aiops-tools` 必须作为独立项目域存在。
3. MVP 阶段必须先做哪些查询与执行能力。
4. 如何保证工具调用和执行动作在安全、审计、幂等、超时控制下运行。

本文档与 `aiops-platform`、`aiops-workflow` 配套阅读，三者共同构成当前阶段建议的项目边界。

## 2. 项目定位

### 2.1 核心定义

`aiops-tools` 负责“如何安全地查询数据、如何安全地执行动作”。

它不是 Incident 主控层，也不是诊断编排层，而是位于 `aiops-workflow` 与真实生产系统之间的安全工具与执行网关层。

其核心职责是：

- 向 `aiops-workflow` 暴露标准化查询工具。
- 向 `aiops-platform` 暴露受控执行能力。
- 对接真实外部系统，如 Prometheus、Loki/ELK、K8s API、Ansible、变更系统。
- 提供鉴权、限流、超时、重试、幂等、审计打点、回滚入口等治理能力。

### 2.2 不负责的事情

`aiops-tools` 明确不负责：

- 持有 `Incident` 主状态
- 页面展示
- 审批流程编排
- 最终 RCA 推理逻辑
- 结构化诊断工作流编排
- 直接决定某个建议动作是否应该执行

### 2.3 为什么这一层必须独立

如果把工具查询和动作执行直接塞进 `platform` 或 `workflow`，会带来几个问题：

- `workflow` 直接接触生产执行能力，安全边界失控
- `platform` 直接耦合多种底层系统协议，后续替换和扩展困难
- 查询和执行能力缺少统一治理，审计和幂等难以标准化

因此，`aiops-tools` 的价值不只是“工具集合”，而是“统一安全网关”。

## 3. 目标调用方与使用场景

### 3.1 直接调用方

- `aiops-workflow`
- `aiops-platform`

### 3.2 典型调用场景

#### 场景一：诊断阶段查询证据

- `aiops-workflow` 调用 `query_metrics`
- `aiops-workflow` 调用 `query_logs`
- `aiops-workflow` 调用 `query_k8s`
- `aiops-workflow` 调用 `query_changes`

目标是收集指标、日志、资源状态和近期变更，形成证据链。

#### 场景二：平台执行低风险动作

- `aiops-platform` 对通过审批或命中低风险策略的动作调用 `execute_action`

目标是通过统一执行网关触发真正动作，而不是让平台自己直接操作生产环境。

#### 场景三：审计与回放

- `aiops-platform` 或内部审计任务查询工具执行记录、执行结果、失败原因、重试轨迹

### 3.3 主要使用者

- 后端工程师：对接平台与 workflow 的工具调用接口
- AI 工程师：让 workflow 稳定调用标准化工具
- 运维/平台工程师：维护外部系统连接、凭证、执行器和治理策略
- 安全/审计角色：核查调用记录、执行记录和失败原因

## 4. MVP 成功标准

MVP 阶段，`aiops-tools` 至少应达到以下结果：

1. 查询类工具可稳定供 `aiops-workflow` 调用。
2. 执行类工具可在受控条件下供 `aiops-platform` 调用。
3. 每次查询和执行都能留下结构化审计记录。
4. 超时、失败、重试、幂等等基础可靠性能力具备。
5. 不允许绕过平台审批直接执行生产动作。

建议的 MVP 指标：

| 指标 | 说明 | MVP 目标 |
| --- | --- | --- |
| Tool success rate | 工具调用成功率 | 高于 95% |
| Query latency P95 | 查询类工具 P95 时延 | 可观测且稳定 |
| Execute latency P95 | 执行类工具 P95 时延 | 可观测且稳定 |
| Audit completeness | 调用审计完整率 | 100% |
| Idempotency correctness | 幂等请求不重复执行 | 100% |
| Unauthorized execute rate | 未授权执行通过率 | 0 |

## 5. 范围边界

### 5.1 MVP 必须范围

- `query_metrics`
- `query_logs`
- `query_k8s`
- `query_changes`
- `execute_action`
- 统一工具调用协议
- 统一执行请求协议
- 鉴权
- 限流
- 超时控制
- 重试策略
- 幂等控制
- 审计打点
- 回滚入口定义

### 5.2 MVP 可选增强

- `query_topology`
- `query_cmdb`
- `query_deployments`
- 更丰富的执行器类型
- 细粒度配额控制
- 多环境多集群路由

### 5.3 明确不在 MVP

- Incident 状态管理
- 审批流引擎
- 复杂图形化工具编排界面
- 自主推理和 RCA 生成
- 完整 CMDB 平台替代
- 全量 ITSM 功能替代

## 6. 核心设计原则

- 查询与执行严格分离
- 工具调用协议统一
- 执行必须受控
- 默认最小权限
- 失败必须可追踪
- 所有动作可审计
- 高风险动作不由工具层自行放行

特别强调：

- `query_*` 允许被 `aiops-workflow` 直接调用
- `execute_action` 不允许被 `aiops-workflow` 直接绕过平台调用
- 即使是低风险自动动作，也必须由 `aiops-platform` 发起最终执行请求

## 7. 核心对象模型

建议 `aiops-tools` 至少围绕以下对象建模：

| 对象 | 说明 | 关键字段 |
| --- | --- | --- |
| `ToolDefinition` | 工具定义 | name, category, owner, enabled |
| `ToolContract` | 工具入参出参契约 | input_schema, output_schema, error_schema |
| `ToolCall` | 一次工具调用记录 | call_id, tool_name, caller, status |
| `ToolCredential` | 外部系统凭证引用 | credential_id, target_system, scope |
| `ExecutionAction` | 一次执行动作定义 | action_name, risk, executor_type |
| `ExecutionRequest` | 执行请求实例 | request_id, action_name, operator, idem_key |
| `ExecutionResult` | 执行结果 | status, output, error, started_at, finished_at |
| `RollbackPlan` | 回滚入口定义 | rollback_type, rollback_ref, rollback_hint |
| `AuditRecord` | 工具审计记录 | actor, target, operation, result |
| `RateLimitPolicy` | 限流策略 | caller_type, qps, burst |
| `RetryPolicy` | 重试策略 | max_retries, retryable_errors, backoff |

## 8. MVP 功能需求

## 8.1 Query Tools 总体要求

### 8.1.1 能力目标

查询类工具负责把底层异构系统查询能力标准化暴露给 `aiops-workflow`。

### 8.1.2 通用要求

所有查询类工具必须支持：

- 统一请求结构
- 统一响应结构
- 超时控制
- 权限校验
- 调用审计
- 错误码规范
- 调用 trace 关联

### 8.1.3 通用响应要求

查询工具返回的数据不应该只是一段原始文本，而应至少包含：

- `status`
- `summary`
- `raw_ref` 或 `query_ref`
- `duration_ms`
- `data`

## 8.2 `query_metrics`

### 8.2.1 目标

对 Prometheus 等指标系统提供统一查询能力。

### 8.2.2 输入建议

- `query`
- `start`
- `end`
- `step`
- `env`
- `cluster`

### 8.2.3 输出建议

- 序列数量
- 点位数量
- 结果摘要
- 原始查询引用

### 8.2.4 MVP 验收标准

1. 能完成基础时序查询。
2. 能返回结构化摘要。
3. 超时和空结果可区分。

## 8.3 `query_logs`

### 8.3.1 目标

对 Loki / ELK / OpenSearch 等日志系统提供统一查询能力。

### 8.3.2 输入建议

- `query`
- `start`
- `end`
- `limit`
- `service`
- `env`

### 8.3.3 输出建议

- 命中数量
- 关键错误模式摘要
- 样例日志片段
- 原始查询引用

### 8.3.4 MVP 验收标准

1. 能按时间窗口查询日志。
2. 能控制返回数量，避免输出过大。
3. 能对空结果、权限失败、查询失败做区分。

## 8.4 `query_k8s`

### 8.4.1 目标

对 K8s 集群提供资源状态查询能力。

### 8.4.2 输入建议

- `cluster`
- `namespace`
- `resource_type`
- `resource_name`
- `selector`

### 8.4.3 输出建议

- 资源摘要
- 当前状态
- 关键异常信息
- 事件摘要

### 8.4.4 MVP 验收标准

1. 可查询 Pod / Deployment / Node 等基础资源。
2. 返回结果不暴露过量底层细节给上层。
3. 能稳定提供适合诊断使用的结构化摘要。

## 8.5 `query_changes`

### 8.5.1 目标

查询近期变更记录，用于辅助根因分析。

### 8.5.2 变更来源建议

- CI/CD 发布记录
- 配置中心变更
- Git 提交或发布流水线
- 工单或变更系统

### 8.5.3 输出建议

- 变更列表
- 变更时间
- 变更类型
- 变更摘要
- 关联服务

### 8.5.4 MVP 验收标准

1. 能按时间窗口返回近期变更。
2. 能按服务或环境过滤。
3. 返回结构适合 workflow 直接消费。

## 8.6 `execute_action`

### 8.6.1 目标

提供统一受控执行网关，对接 Ansible、K8s API、脚本执行器等真实动作入口。

### 8.6.2 关键原则

- 工具层执行，不代表工具层做决策
- 工具层接收的是已通过平台策略校验的执行请求
- 工具层必须校验请求合法性、幂等性、权限和执行条件

### 8.6.3 MVP 支持的动作类型建议

- 重启 Pod
- 扩容 Deployment
- 回滚 Deployment
- 执行白名单脚本
- 调用预定义 Ansible Playbook

### 8.6.4 请求字段建议

- `request_id`
- `incident_id`
- `action_name`
- `action_params`
- `risk`
- `approval_token` 或等价审批校验信息
- `idempotency_key`
- `operator`

### 8.6.5 输出字段建议

- `status`
- `executor_type`
- `execution_id`
- `started_at`
- `finished_at`
- `result_summary`
- `rollback_hint`

### 8.6.6 MVP 验收标准

1. 执行动作必须经过显式请求。
2. 同一 `idempotency_key` 不重复执行。
3. 执行结果必须有结构化返回。
4. 失败时能返回明确原因和回滚入口提示。

## 8.7 鉴权与授权

### 8.7.1 目标

确保不是任何调用方都可以任意使用所有工具。

### 8.7.2 必须能力

- 调用方身份识别
- 工具级权限控制
- 动作级权限控制
- 环境级权限控制
- 生产环境额外保护

### 8.7.3 原则

- `aiops-workflow` 默认只能调查询类工具
- `aiops-platform` 才能发起执行类请求
- 高风险执行请求必须有平台侧审批上下文

## 8.8 限流与配额控制

### 8.8.1 必须能力

- 调用方限流
- 工具级限流
- 突发流量保护
- 重查询防护

### 8.8.2 设计价值

如果没有限流，诊断工作流在告警风暴下可能把底层 Prometheus / Loki / K8s API 一起打挂。

## 8.9 超时、重试与熔断

### 8.9.1 必须能力

- 每个工具可配置超时
- 可配置重试次数
- 区分可重试错误和不可重试错误
- 下游故障时快速失败

### 8.9.2 原则

- 查询类工具允许有限重试
- 执行类工具重试要非常谨慎，必须结合幂等键

## 8.10 幂等控制

### 8.10.1 为什么必须做

执行类动作如果没有幂等控制，可能因为重试、网络抖动或平台重复提交导致重复操作生产环境。

### 8.10.2 必须能力

- `idempotency_key`
- 幂等窗口
- 重复请求识别
- 幂等结果返回

### 8.10.3 验收标准

1. 重复提交同一执行请求不会造成重复执行。
2. 调用方可以拿到第一次执行结果或明确的幂等命中结果。

## 8.11 审计与可追踪性

### 8.11.1 必须记录的内容

- 谁调用了哪个工具
- 调用了哪个目标系统
- 请求参数摘要
- 执行或查询结果摘要
- 错误信息
- 开始时间 / 结束时间
- trace id / incident id / task id

### 8.11.2 边界说明

`aiops-tools` 记录工具层审计，`aiops-platform` 记录业务主审计。
两者需要能通过 `incident_id`、`request_id`、`trace_id` 关联。

## 8.12 回滚入口

### 8.12.1 目标

工具层至少要给平台一个明确的“如何回滚”入口，而不是执行后只返回成功或失败。

### 8.12.2 MVP 形式

- 返回回滚建议
- 返回回滚命令引用
- 返回回滚 playbook 引用

### 8.12.3 注意事项

- 工具层可以提供回滚入口定义
- 是否自动回滚、何时回滚，仍由平台或策略层决定

## 9. 统一接口与契约建议

## 9.1 Query 接口建议

建议所有查询类工具通过统一模式暴露，例如：

- `POST /api/v1/tools/query_metrics`
- `POST /api/v1/tools/query_logs`
- `POST /api/v1/tools/query_k8s`
- `POST /api/v1/tools/query_changes`

统一返回结构建议：

```json
{
  "request_id": "req-001",
  "tool_name": "query_metrics",
  "status": "ok",
  "summary": "p95 latency 从 220ms 升至 2.1s",
  "data": {},
  "duration_ms": 328
}
```

## 9.2 Execute 接口建议

建议执行类能力采用统一入口：

- `POST /api/v1/tools/execute_action`

统一返回结构建议：

```json
{
  "request_id": "exec-001",
  "execution_id": "run-001",
  "status": "accepted",
  "executor_type": "k8s",
  "result_summary": "deployment/order-service scale requested",
  "rollback_hint": "scale down deployment/order-service"
}
```

## 9.3 错误码建议

建议至少定义以下错误码：

| code | 含义 | 说明 |
| --- | --- | --- |
| `TOOL_UNAUTHORIZED` | 无权限访问工具 | 权限校验失败 |
| `TOOL_TIMEOUT` | 工具调用超时 | 查询或执行超时 |
| `TOOL_RATE_LIMITED` | 触发限流 | 调用频率过高 |
| `TOOL_DOWNSTREAM_ERROR` | 下游系统异常 | Prometheus/Loki/K8s 等异常 |
| `EXECUTION_NOT_ALLOWED` | 不允许执行 | 平台上下文或风险约束不满足 |
| `IDEMPOTENCY_CONFLICT` | 幂等冲突 | 重复执行请求 |
| `ROLLBACK_UNAVAILABLE` | 无回滚入口 | 动作未提供回滚方案 |

## 10. 适配层设计要求

## 10.1 指标适配层

- 适配 Prometheus / VictoriaMetrics 等指标系统
- 屏蔽底层差异，统一输出结构

## 10.2 日志适配层

- 适配 Loki / ELK / OpenSearch
- 屏蔽不同查询语言差异

## 10.3 K8s 适配层

- 适配多集群访问
- 对资源对象做统一摘要化输出

## 10.4 执行器适配层

- 适配 K8s API
- 适配 Ansible
- 适配预定义脚本执行器

## 10.5 设计原则

- 上层不直接接触底层协议细节
- 工具契约稳定优先于底层实现灵活性

## 11. 非功能要求

### 11.1 安全性

- 凭证统一管理，禁止硬编码
- 最小权限原则
- 生产环境与测试环境权限隔离
- 执行能力与查询能力权限隔离

### 11.2 可靠性

- 关键工具调用可观测
- 下游异常时有降级路径
- 查询失败不应拖垮整体系统

### 11.3 可维护性

- 新增一个工具适配器不应影响现有契约
- 配置与代码职责清晰
- 工具元数据和契约文档化

### 11.4 可扩展性

- 允许后续增加更多查询工具
- 允许新增执行器而不破坏统一执行入口

## 12. 部署与实现建议

### 12.1 当前实现建议

结合主架构文档，当前阶段适合将 `aiops-tools` 实现为 Python 服务。

适合的原因：

- 便于与 AI / workflow 生态结合
- 便于快速对接 Prometheus / K8s / Ansible 等基础设施
- 便于实现鉴权、重试、超时、幂等等中间件能力

### 12.2 当前部署建议

- 作为独立服务部署
- 对外暴露统一 HTTP API
- 凭证和外部系统连接配置独立管理

### 12.3 不建议的实现方式

- 让 workflow 直接调用底层系统 SDK
- 让平台直接散落调用不同外部系统
- 查询和执行接口各自为政，没有统一治理

## 13. 分阶段实施建议

### Phase 1

- 打通 `query_metrics`
- 打通 `query_logs`
- 打通 `query_k8s`
- 建立统一 tool contract
- 建立基础审计与超时控制

### Phase 2

- 打通 `query_changes`
- 打通 `execute_action`
- 建立幂等控制与重试策略
- 接入基础回滚入口

### Phase 3

- 增加更多连接器
- 增加更细粒度限流和权限策略
- 补齐多环境多集群治理

## 14. 推荐的仓库内文档与资产结构

如果后续 `aiops-tools` 独立成仓，可参考如下结构：

```text
tools/
  AIOps_Tools_Requirements.md
  contracts/
  adapters/
  executors/
  policies/
  examples/
```

## 15. 结论

`aiops-tools` 的本质不是一个简单的“工具箱”，而是整个 AIOps 架构中的安全与协议隔离层。

如果 `aiops-platform` 解决的是“事件如何流转”，
`aiops-workflow` 解决的是“诊断如何形成”，
那么 `aiops-tools` 解决的就是：

- 数据如何安全地查
- 动作如何安全地做
- 真实系统如何被统一接入

因此，`aiops-tools` 的第一优先级不是堆很多工具，而是先把以下能力做稳：

- 标准化查询接口
- 统一执行网关
- 鉴权
- 超时
- 重试
- 幂等
- 审计

只有这层稳定，平台和 workflow 才能在上层安心演进。
