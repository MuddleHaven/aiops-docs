# AIOps 架构图详解（实践落地路线）

本文是 `AIOps_Practical_Route_Architecture.png` 的配套说明，目标不是再讲一遍产品愿景，而是回答 3 个更实际的问题：

- 这张图是怎么推导出来的
- 图里的每一层到底在做什么
- 图中每个框和未来实现模块如何一一对应

![AIOps Practical Route Architecture](AIOps_Practical_Route_Architecture.png)

## 1. 这张图是怎么来的

这张图不是从技术栈出发画出来的，而是从 AIOps 要解决的实际链路倒推出来的。

### 1.1 从业务问题倒推

一个典型故障场景里，平台至少要回答下面几个问题：

1. 告警和事件从哪里进来，格式怎么统一。
2. 除了告警本身，还需要哪些上下文才能判断问题。
3. 多条重复或相关告警，怎么合并成一个可处理的 Incident。
4. 谁来做诊断、给出 RCA 和动作建议。
5. 建议出来以后，怎样保证执行是安全、可控、可审计的。
6. 处理完以后，经验如何沉淀，而不是下次还从头来一遍。

把这 6 个问题按顺序展开，就自然会得到 6 层：

- `Layer A`：先接数据
- `Layer B`：再补上下文
- `Layer C`：把事件变成 Incident 并推动流转
- `Layer D`：让 Agent 做诊断和建议
- `Layer E`：把动作放到可控执行框架里
- `Layer F`：做复盘、评估和知识更新

所以，这张图本质上是一个“从故障进入平台到经验回流平台”的闭环图，而不是单纯的服务清单图。

## 2. 读这张图的正确方式

### 2.1 先看主链路

最推荐的阅读顺序是自下而上：

`Observability Data` -> `NormalizedEvent` -> `IncidentContext` -> `Incident` -> `RCA Result` -> `Execution Results` -> `Knowledge Update`

这条链路对应的是“问题进入平台并被处理掉”的全过程。

### 2.2 再看两种流

- 蓝色实线：主控制流 / API 调用 / 执行 / 审计
- 红色虚线：反馈流，表示复盘之后对知识库、Prompt、工作流和指标的持续优化

### 2.3 最后看左右分工

- 左侧和中间：偏主处理链路，解决“当前这个故障怎么处理”
- 右侧：偏复盘和优化，解决“以后类似问题怎么更快处理”
- 顶部：偏执行治理，解决“能不能安全执行”

## 3. 一一对应说明

### 3.1 外部输入

### Prometheus / Loki-ELK / CMDB / Changes

- 这些不是平台内部模块，而是平台要接入的数据源。
- 它们共同回答一个问题：平台拿什么来感知故障。
- 在 MVP 阶段，不必一次性接全，优先级通常是 `Prometheus -> Logs -> CMDB -> Changes`。

### 3.2 Layer A: Perception

### 这一层解决什么问题

- 把不同来源的告警、事件、变更信号接进来。
- 把不同格式的数据统一成平台内部可消费的标准事件。

### 图中模块

- `collector-adapter`
- `alert-webhook`
- `event-normalizer`

### 输入 / 输出

- 输入：监控告警、日志事件、CMDB 信息、变更记录
- 输出：`NormalizedEvent`

### 为什么必须有这一层

- 如果没有统一事件格式，后面的去重、关联、AI 诊断都会高度依赖每个数据源自己的字段，系统很难扩展。

### 3.3 Layer B: Data Governance

### 这一层解决什么问题

- 只看告警标题通常不够，需要补齐服务归属、依赖关系、变更记录和历史相似事件。
- 这层的目标是把“告警”变成“带上下文的事件”。

### 图中模块

- `context-enricher`
- `topology-resolver`
- `change-correlator`

### 输入 / 输出

- 输入：`NormalizedEvent`
- 输出：`IncidentContext`

### 为什么必须有这一层

- AI 诊断不是只靠一句告警文案，而是要看时间窗口、依赖拓扑、近期变更和历史案例。
- 这一层是后面 RCA 质量的基础。

### 3.4 Layer C: Event Orchestration

### 这一层解决什么问题

- 多条重复告警不能让人逐条处理，需要合并成一个 Incident。
- Incident 形成后，还要有状态流转、升级通知和时间线记录。

### 图中模块

- `Incident Service`
- `Deduplication & Correlation Engine`
- `Escalation Engine`
- `Timeline Service`

### 实现映射

- `Incident Service` 对应 `incident-service`
- `Deduplication & Correlation Engine` 可以落成 `dedupe-group-engine` + 关联规则
- `Escalation Engine` 对应 `escalation-engine`
- `Timeline Service` 对应 `timeline-service`

### 输入 / 输出

- 输入：`IncidentContext`
- 输出：处于不同状态的 `Incident`

### 状态机为什么重要

- 图里的 `new -> triaged -> diagnosing -> remediating -> resolved/closed` 不是装饰，而是平台主流程。
- 没有状态机，就很难做 SLA、升级、审批和复盘闭环。

### 3.5 Layer D: Intelligent Decision

### 这一层解决什么问题

- 当 Incident 进入 `diagnosing` 状态后，需要 Agent 去做证据收集、RAG 检索、诊断判断和动作建议。

### 图中模块

- `Custom Gateway`
- `Dify Platform`
- `Agent Brain (Dify)`

### 为什么同时有 Custom Gateway 和 Dify

- `Dify Platform` 负责工作流编排、模型调用、RAG 检索和多轮对话。
- `Custom Gateway` 负责把平台和 Dify 解耦，承担结果标准化、策略封装、审计打点和兜底逻辑。
- 换句话说，Dify 负责“智能”，Custom Gateway 负责“平台接入与控制”。

### 输入 / 输出

- 输入：`Incident` + `IncidentContext`
- 输出：`RCA Result`

### RCA Result 至少要包含什么

- 结论
- 置信度
- 证据链
- 影响范围
- 建议动作

### 3.6 Layer E: Action Execution with Guardrails

### 这一层解决什么问题

- AI 给出建议不等于可以直接执行。
- 执行必须经过审批、策略校验、风险控制和审计留痕。

### 图中模块

- `action-registry`
- `approval-service`
- `Policy Engine`
- `audit-service`
- `Approval Gateway`
- `Execution Control with Policy Gateway`
- `Executors`

### 实现映射

- `Approval Gateway` 可以落成 `approval-service`
- `Execution Control with Policy Gateway` 可以落成 `execution-gateway` + `policy-engine`
- `Executors` 对接 Ansible、K8s API、脚本引擎等真实执行器

### 为什么必须单独画一层

- 因为“诊断正确”和“可以安全执行”是两个问题。
- 这一层存在的意义，就是避免 Agent 直接碰生产资源。

### 3.7 Layer F: Learning Loop

### 这一层解决什么问题

- 事件处理结束后，要把经验沉淀下来，变成下次更快的处理能力。

### 图中模块

- `postmortem-service`
- `Knowledge Feedback`
- `kpi-dashboard`
- `Assessment Metrics`
- `Optimization Outputs`

### 实现映射

- `Knowledge Feedback` 可以落成 `knowledge-feedback-worker`
- `Assessment Metrics` 由 `kpi-dashboard` 统计输出
- `Optimization Outputs` 体现为 Runbook 更新、Prompt/Workflow 优化、知识库整理

### 为什么不把它并到前面

- 前面几层解决的是“这次故障怎么处理”。
- 这一层解决的是“以后类似故障怎么处理得更快、更准、更稳”。
- 所以它是学习闭环，不是主处理链路的一部分。

## 4. 图中几个最容易困惑的点

### 4.1 为什么 Layer B 和 Layer C 都有“关联”含义

- `Layer B` 的关联更偏上下文补齐，比如拓扑、变更、历史事件。
- `Layer C` 的关联更偏事件处理，比如去重、分组、归并、Incident 归属。
- 一个是“看懂事件背景”，一个是“把事件组织成可处理对象”。

### 4.2 为什么 Dify 不直接调用执行器

- 因为这样会让 AI 和生产执行强耦合，风险很高。
- 正确做法是：Dify 只输出建议，平台统一做审批、策略和执行控制。

### 4.3 为什么学习闭环在最右侧

- 因为它不是每一步主链路都必经的同步流程，而是事件关闭后的回流与优化过程。
- 右侧单独成区，视觉上更容易看出“处理”和“学习”是两个阶段。

## 5. 用一个例子把图串起来

假设 `order-service` 延迟突然升高，这张图里的处理过程可以理解成：

1. Prometheus 触发高延迟告警，进入 `Layer A`。
2. `event-normalizer` 把告警转成统一 `NormalizedEvent`。
3. `Layer B` 补齐服务归属、依赖拓扑、最近变更和历史相似事件，形成 `IncidentContext`。
4. `Layer C` 发现同类告警很多，先去重归并，再生成一个 Incident，并把状态推进到 `diagnosing`。
5. `Layer D` 通过 `Custom Gateway` 调用 `Agent Brain (Dify)`，综合 metrics、logs、changes 和 SOP 输出 `RCA Result`。
6. 如果建议动作是低风险扩容，就进入 `Layer E` 的策略和执行控制；高风险动作则进入审批流。
7. 执行结果和关键操作写入时间线与审计。
8. Incident 关闭后，`Layer F` 做 postmortem，更新知识库，统计是否真的缩短了 MTTR、RCA 是否准确。

如果你按这个例子回头看图，基本就能理解每条箭头为什么存在。

## 6. 这张图不是部署图，而是逻辑图

这个点很重要。

- 图里的一个框，不一定等于一个独立服务。
- MVP 阶段，多个模块可以合并在一个后端服务里实现。
- 例如 `Incident Service`、`Timeline Service`、`Escalation Engine` 可以先合并。
- `Custom Gateway`、工具代理和执行网关，也可以先做成一个平台后端中的独立模块。

所以，读这张图时要把它理解成“职责分层图 + 关键能力图”，而不是“最终微服务拆分图”。

## 7. 你可以怎么对别人解释这张图

如果只用 1 分钟介绍，可以直接这么说：

> 这是一张 AIOps 实践路线的闭环架构图。底层先把 Prometheus、日志、CMDB 和变更系统的信号统一成标准事件；中间层补齐上下文并把多条告警收敛成 Incident；然后通过 Dify 做诊断、RAG 检索和 RCA 输出；再通过审批、策略和执行网关把建议动作变成受控执行；最后把复盘结果沉淀为知识库、Runbook 和优化指标，形成持续学习闭环。

如果要更简短，可以记一句话：

> 这张图讲的是“怎么把告警变成可控处置，再把处置经验变成下一次更快的处置能力”。
