# PagerDuty 功能拆解与复刻需求

## 1. 文档目标

本文档用于基于 PagerDuty 公开产品资料，系统梳理其在以下三个方向的核心能力：

- 事件编排（Event Orchestration）
- 值班排班与升级（On-call / Escalation）
- 事件复盘与改进闭环（Post-Incident Review / Postmortem）

目标不是逐字复述官网，而是把 PagerDuty 的能力整理成一份适合产品设计和功能复刻的需求文档，便于后续回答两个问题：

1. 如果先复刻一个 PagerDuty 风格的 Incident Management 平台，MVP 应该包含哪些功能。
2. 在完成基础 Incident Management 后，AIOps 应该再叠加哪些智能化能力。

说明：

- 本文基于 PagerDuty 官方公开页面与帮助文档整理，不代表其内部 PRD。
- 文中“建议复刻”表示适合纳入你的产品规划，不代表必须完全 1:1 实现。

## 2. PagerDuty 的产品定位

PagerDuty 本质上不是一个单纯“发告警”的工具，而是一套面向生产故障响应的运行管理平台。它解决的问题链路是完整的：

1. 从监控系统、日志系统、CI/CD、云平台接收事件。
2. 对事件进行路由、降噪、抑制、富化和分派。
3. 将事件转化为 `Incident`，并自动通知当前值班人。
4. 在响应过程中组织协同人员、角色、任务和沟通。
5. 在恢复后形成复盘、行动项、分析报表和改进闭环。

因此，PagerDuty 的核心价值不是“消息通知”，而是“面向 Incident 生命周期的编排系统”。

## 3. 复刻 PagerDuty 之前先明确的对象模型

如果要复刻 PagerDuty，首先需要定义稳定的领域对象。否则后续编排、升级、复盘都会缺少统一模型。

建议优先抽象以下核心对象：

| 对象 | 含义 | 说明 |
| --- | --- | --- |
| `Event` | 外部系统送入的原始事件 | 来自监控、日志、变更、邮件、Webhook 等 |
| `Alert` | 平台内部标准化后的告警 | 可被抑制、分组、去重、转 Incident |
| `Incident` | 需要协同处理的问题实体 | 有状态、负责人、升级链路、时间线 |
| `Service` | 技术服务或受监控对象 | Incident 通常挂在某个 service 上 |
| `Business Service` | 业务服务 | 用于对外状态同步、业务影响展示 |
| `EscalationPolicy` | 升级策略 | 定义未响应时如何升级 |
| `Schedule` | 值班表 | 定义谁在什么时间值班 |
| `Responder` | 响应人 | 当前参与某个 Incident 的人 |
| `Stakeholder` | 干系人/订阅人 | 接收状态更新，但不一定参与处理 |
| `IncidentRole` | 事件角色 | 如 Incident Commander、沟通负责人 |
| `IncidentTask` | 事件任务 | 响应过程中需要被跟踪的具体事项 |
| `PostIncidentReview` | 复盘实体 | 用于复盘、分析、行动项沉淀 |
| `ActionItem` | 改进行动项 | 复盘后的工程任务、流程任务、会议任务 |

PagerDuty 的很多功能之所以可扩展，关键就在于这些对象之间关系稳定，例如：

- `Service -> EscalationPolicy -> Schedule -> On-call User`
- `Event -> Alert -> Incident`
- `Incident -> Responders / Subscribers / Roles / Tasks / Timeline`
- `Incident -> PostIncidentReview -> ActionItems`

## 4. 功能全景

从复刻角度看，PagerDuty 可以拆成 6 个能力域：

1. 事件接入与标准化
2. 事件编排与路由
3. 值班排班与升级通知
4. Incident 协同处理
5. 复盘与改进闭环
6. 分析报表与运营洞察

下面逐块展开。

## 5. 事件接入与标准化能力

### 5.1 多入口事件接入

PagerDuty 支持从多种来源触发 Incident 或导入事件，典型入口包括：

- 第三方监控集成
- Events API
- REST API
- Email Integration
- Slack / Mobile / Web 手工声明 Incident

对应你的产品，MVP 至少应支持：

- Webhook / Events API 接入
- 手工创建 Incident
- 邮件或 IM 触发可以后置

### 5.2 统一事件模型

PagerDuty 在编排前会把事件转为统一字段模型，可理解为平台内部标准事件格式。这样后续规则就不需要适配每个监控源的异构字段。

建议你的平台在事件接入层完成：

- 原始事件落库
- 标准字段抽取
- 原始 payload 保留
- 常用字段标准化，如：
  - 标题
  - 来源
  - 服务名
  - 主机/实例
  - 严重级别
  - 环境
  - 标签
  - 去重键 `dedup_key`

### 5.3 告警与 Incident 的区分

PagerDuty 明确区分 `Alert` 与 `Incident`：

- `Alert` 是事件处理后的告警记录。
- `Incident` 是需要人介入的处理单元。

这个设计非常关键，因为并不是所有事件都应该立刻通知人。很多事件只需要被记录、抑制、合并或等待进一步判断。

建议你的系统也采用二层模型：

- `Event -> Alert -> Incident`

这样后续做降噪、关联和 AIOps 会更自然。

## 6. 事件编排能力

事件编排是 PagerDuty 最值得优先复刻的部分之一。它本质上是“事件进入平台后，如何按规则被处理”的能力中心。

### 6.1 单入口统一接入

PagerDuty 的 Event Orchestration 支持将事件统一打到一个入口，再由平台按规则决定：

- 路由到哪个 `Service`
- 是否触发 Incident
- 是否仅保留为 suppressed alert
- 是否变更优先级、字段或升级策略

这个能力的价值在于：

- 上游系统不用知道最终具体通知谁
- 规则集中管理，不散落在各个服务里
- 后续可以逐步演化更复杂的自动化逻辑

### 6.2 编排层级

PagerDuty 的编排规则可分成三层理解：

- 全局编排规则：进入平台后的统一预处理
- 路由规则：决定流向哪个服务
- 服务级编排规则：到达服务后再做细分处理

你的产品复刻时建议采用类似分层：

1. `global_orchestration`
2. `routing_rules`
3. `service_orchestration`

### 6.3 路由规则能力

PagerDuty 的路由规则支持基于事件内容进行服务分发。需要复刻的关键点包括：

- 条件匹配路由
- 无条件默认路由
- catch-all 兜底规则
- 动态路由
  - 从事件字段提取服务名或服务 ID
  - 用正则提取目标值
- 时间条件路由
- 日期范围路由
- 频次条件路由

这意味着规则引擎至少需要支持以下条件类型：

- 字段等于/不等于
- 包含/不包含
- 正则匹配
- 数值比较
- 时间窗口
- 频率阈值

### 6.4 编排动作能力

PagerDuty 在规则命中后可以执行多个动作。这里是最核心的复刻清单。

#### 6.4.1 事件与 Incident 控制动作

- 触发 Incident
- 暂停通知一段时间后再触发
- 抑制告警，不创建 Incident
- 丢弃事件并停止处理
- 设置 Incident Priority
- 动态覆盖 Escalation Policy
- 向 Incident 自动写入说明 note

这是 PagerDuty 从“收事件”升级为“事件处理编排平台”的关键。

#### 6.4.2 字段富化与转换

PagerDuty 支持把原始事件字段转为统一字段，或覆盖已有字段。复刻时应支持：

- 字段映射
- 模板赋值
- 正则提取
- 自定义变量
- 构造 `dedup_key`
- 写入 `custom_details`
- 补充上下文信息

这类能力对后续 AIOps 非常重要，因为它直接决定事件数据质量。

#### 6.4.3 Incident 自定义字段填充

PagerDuty 允许把事件字段或变量写入 Incident Custom Fields。这个能力非常适合你的产品复刻，因为它能把结构化上下文贯穿到整个响应链路。

建议支持：

- 静态值填充
- 动态字段映射
- 模板变量填充
- 多种字段类型
  - 文本
  - 单选
  - 多选
  - 标签
  - 时间
  - 整数/小数
  - 布尔值

### 6.5 去重与聚合前置能力

PagerDuty 的编排可以直接设置 `dedup_key`。这意味着：

- 相同问题不会不断创建新的 Incident
- 后续事件可以进入同一个 Alert / Incident 生命周期

这部分建议作为 MVP 必须能力，因为没有去重，值班系统会非常吵。

### 6.6 Catch-all 与未命中处理

PagerDuty 对未命中规则的事件提供兜底行为：

- 不进入服务，也不创建 Incident
- 或统一路由到一个默认服务

这看似是小功能，实际上对线上运行非常重要。你的平台也需要支持：

- 明确的兜底规则
- 兜底规则审计
- 未命中事件可观测

### 6.7 自动化触发点

PagerDuty 的编排规则可以进一步触发：

- Webhook
- Automation Actions
- Incident Workflows

这说明其编排层不仅能“判定”，还能“驱动动作”。

你的产品可以先做两级：

- MVP：规则命中后调用内部工作流/外部 Webhook
- 后续：对接 Runbook、审批流、自动修复动作

## 7. 值班、排班与升级能力

这是 PagerDuty 的第二根主线，也是最容易形成产品闭环的模块。

### 7.1 值班排班 `Schedule`

PagerDuty 的值班排班不是简单名单，而是一个完整的时间覆盖系统。核心能力包括：

- 创建排班表
- 设置时区
- 设置轮转方式
  - 日轮转
  - 周轮转
  - 自定义轮转
- 设置交接时间
- 调整人员顺序
- 多层排班 `layer`
- 限定值班时间窗口
  - 按天固定时间段
  - 按周不同时间段
- 支持 follow-the-sun 场景
- 预览最终排班结果

建议你的产品在设计时把 `Schedule` 视为一等对象，而不是附属配置。

### 7.2 排班层 `Schedule Layer`

PagerDuty 支持一个排班中有多个 layer，后创建的 layer 优先级更高。这个设计可以支持很多复杂场景：

- 平日/周末双层排班
- 全球时区覆盖
- 白班/夜班覆盖
- 节假日特殊层覆盖正常层

如果你想做企业级值班系统，`layer` 是很值得复刻的设计。

### 7.3 覆盖缺口与约束

PagerDuty 明确告诉用户：如果值班表限制了时间，就可能出现 coverage gap。若无人值班，Incident 甚至不会被创建。

这给你的产品一个重要启示：

- 值班系统必须可计算“当前是否有人值班”
- 覆盖缺口必须可见
- 触发失败要给明确原因

建议必须支持：

- 当前 on-call 查询
- 未来值班预览
- 覆盖缺口检测
- 无值班人告警

### 7.4 升级策略 `Escalation Policy`

PagerDuty 的升级策略连接 `Service`、`Schedule` 和 `User`。核心功能包括：

- 多级升级规则
- 每级指定用户或值班表
- 每级超时时间 `escalation timeout`
- 未响应时自动升级到下一级
- 可重复整条升级策略多次
- 多人并行通知
- 支持 round robin
- 可关联到 team

这个模块建议你按“规则链”建模，不要只做“主值班人 + 备份值班人”两层固定结构。

### 7.5 升级行为细节

PagerDuty 在升级行为上有几个关键设计，非常值得复刻：

- 一旦有人 `acknowledge`，升级停止
- 如果当前层无人值班，则跳到下一层
- Incident 使用的是触发当时的升级策略快照
- 修改升级策略不会影响已经打开的 Incident

这几点会极大减少线上歧义。建议你的系统也保留：

- 升级快照
- 升级轨迹
- 每次通知记录
- 每次升级原因

### 7.6 通知方式与个人通知规则

PagerDuty 通知不只由升级策略决定，还受用户个人通知规则影响。支持渠道包括：

- Push
- Phone
- SMS
- Email
- Slack 等集成渠道

你的产品至少需要区分两层：

- 平台层决定“应该通知谁”
- 用户层决定“怎么通知他”

### 7.7 On-call Handoff

PagerDuty 提供值班交接通知，支持在上岗前最多 48 小时提醒。这类功能虽然不是 MVP 必需，但对真实值班体验非常重要。

建议纳入第二阶段：

- 值班前提醒
- 值班交接摘要
- 当前/下一班值班人可视化

## 8. Incident 协同响应能力

PagerDuty 并不止于“谁被通知”，它还覆盖了响应过程中的组织与协同。

### 8.1 Incident 生命周期

PagerDuty 的 Incident 具有明确状态：

- `triggered`
- `acknowledged`
- `resolved`

并支持：

- 手工声明 Incident
- API 触发 Incident
- 重新打开 Incident
- 取消 acknowledge
- redaction（敏感信息擦除）

建议你的产品至少实现：

- 新建
- 确认接手
- 解决
- 重开
- 状态时间线

### 8.2 Incident Timeline

PagerDuty 的 Timeline 是非常关键的基础能力，它记录：

- 状态变更
- 通知发送
- 升级动作
- 响应人加入/拒绝
- 备注
- 工作流执行结果

这不只是审计日志，也是复盘的数据源。

建议你的产品从第一版就做统一 Timeline，而不是把日志散落在多个页面。

### 8.3 手工声明与覆盖分派

PagerDuty 支持手工创建 Incident 时：

- 指定受影响服务
- 指定优先级与紧急度
- 直接覆盖默认升级策略
- 指定用户或升级策略作为 assignee
- 添加额外响应人
- 添加会议桥接信息

这意味着手工声明不是“简单建单”，而是“一次完整的事件动员动作”。

### 8.4 添加响应人 `Add Responders`

PagerDuty 支持在 Incident 已经打开后继续加入响应人，包括：

- 单个用户
- 整个升级策略
- 手工加入
- 通过 Incident Workflow 自动加入

响应人可接受、拒绝或待处理，平台会记录状态。

建议你的系统复刻：

- 主负责人
- 额外响应人
- 响应邀请状态
- 升级策略作为 responder group

### 8.5 Incident Roles

PagerDuty 有专门的 Incident Roles，而不是只靠备注描述角色。能力包括：

- 预定义角色
- 自定义角色
- 给响应人分配角色
- 一个用户可承担多个角色
- 同一角色同一时刻只指向一个负责人
- Slack / Teams / Workflow 内都可操作

这是非常值得借鉴的企业级设计。

建议 MVP 支持至少两个默认角色：

- `Incident Commander`
- `Communications Lead`

### 8.6 Incident Tasks

PagerDuty 允许在响应过程中创建任务，既可以 ad-hoc，也可以由工作流预定义。任务可包括：

- 任务名称
- 描述
- 状态
- 指派人

这类能力有助于把“协同处理”从聊天工具中结构化出来。

建议在你的产品第二阶段加入：

- Incident 下挂任务列表
- 任务指派
- 任务状态流转
- 与工作流联动

### 8.7 会议与协作入口

PagerDuty 支持会议桥接信息，例如：

- Conference number
- Meeting URL

这说明 Incident 页不仅是状态页，也是协作入口页。

你的产品建议预留：

- 会议链接
- Chat channel 链接
- Runbook 链接
- Dashboard 链接

## 9. 干系人沟通与状态更新能力

PagerDuty 另一个很强的点，是把“故障处理”和“故障通报”明确分开。

### 9.1 订阅人 `Subscribers`

PagerDuty 支持给 Incident 增加订阅人：

- 用户
- 团队
- 业务服务订阅者
- 工作流自动订阅

而且订阅本身是 silent 的，不会立即打扰订阅者，只有后续状态更新才会通知。这是很成熟的设计。

### 9.2 状态更新 `Status Updates`

PagerDuty 支持响应人主动发布状态更新，典型内容包括：

- 当前影响面
- 当前处理进展
- 预计恢复时间
- 下一次更新时间

更新可通过：

- Web
- Mobile
- API
- Incident Workflow

### 9.3 多渠道发布

PagerDuty 的状态更新可发到：

- Push
- Email
- SMS
- Internal Status Page

这意味着平台要区分：

- 响应者通知
- 干系人状态通知

这两个不是一回事。

### 9.4 模板化沟通

PagerDuty 提供 `Status Update Templates`，支持：

- 模板名称和描述
- 标准消息模板
- Email 标题和正文模板
- 变量字段引用
- Liquid 模板语言
- 条件渲染
- 自定义字段带入

这类能力非常有价值，因为很多企业故障通报的难点不在“能发消息”，而在“内容是否规范一致”。

建议你的产品至少支持：

- 状态更新模板
- 变量占位符
- 富文本邮件模板可以后置

## 10. 复盘与事件学习能力

PagerDuty 通过 Jeli 把复盘做成了独立能力域，而不是只留一段文字备注。

### 10.1 复盘入口

PagerDuty 支持从已解决 Incident 创建 Post-Incident Review。也支持传统 Postmortem 报告模式。

这意味着：

- 复盘对象应与 Incident 绑定
- 一个 Incident 可以触发一个正式复盘流程

### 10.2 复盘状态流转

Jeli 提供明确的复盘阶段：

1. `Unassigned`
2. `Assigned`
3. `In Progress`
4. `In Review`
5. `Completed`

这个设计非常适合你复刻，因为它让“复盘有没有做”变成可管理对象，而不是靠口头推动。

### 10.3 复盘报告结构

PagerDuty 的 Post-Incident Review Report 不是单一表单，而是一组结构化标签页，核心包括：

- Overview
- Summary
- Key Roles
- Imported Data Sources
- Tags
- Related Reviews
- Narrative
- People
- Takeaways
- Action Items
- Attached Resources
- Export Report

对于你的产品，这说明复盘需要覆盖三类内容：

- 事实数据
- 叙事分析
- 改进行动

### 10.4 Narrative Builder

这是 Jeli 很有特色的能力。它不是简单时间线，而是“基于证据构建事故叙事”。核心点包括：

- 导入 Slack 等原始数据
- 通过 Event Marker 组织叙事
- Marker 类型包括：
  - Detection
  - Diagnosis
  - Repair
  - Key Moment
- 每个 Marker 可包含：
  - 摘要
  - Supporting Evidence
  - Supplemental Evidence
  - Notes
  - 时间点或时间范围
- 支持按参与者、来源、标签检索证据
- 支持复盘会议时直接展示和讲解

这个能力对于 AIOps 非常重要，因为它让“故障经过”变成结构化知识，而不是散落在聊天记录里。

### 10.5 Key Roles 与参与人分析

Jeli 在复盘里单独强调：

- 关键角色是谁
- 谁在什么阶段参与了 Incident
- 谁承担了调查者/指挥者/沟通者职责

这对组织学习和流程优化很有价值。

### 10.6 Takeaways 与 Action Items

PagerDuty 把复盘结论分成两个层面：

- `Takeaways`：经验、发现、主题
- `Action Items`：后续必须落实的动作

Action Items 既可以是：

- 纯文本 quick items
- 也可以对接 Jira 创建或绑定工单

这一点非常值得直接复刻。很多平台停留在“写复盘文档”，但没有把行动项拉通到工程系统。

### 10.7 复盘附件与导出

PagerDuty 支持：

- 上传 PDF、TXT、VTT 等附件
- 导出 PDF 报告
- 自定义导出章节顺序

这类能力不是 MVP 必需，但对企业交付和审计非常重要。

## 11. 分析报表与管理洞察

PagerDuty 提供 `Insights`，主要面向管理和运营改进，包含：

- Incident Activity
- Service Performance
- Responder
- Team
- Escalation Policy
- Business Impact

这意味着系统不仅服务一线值班人，还服务管理者。

建议你的产品在基础版本至少统计：

- Incident 数量
- MTTA / MTTR
- 升级次数
- 值班命中率
- 服务级故障分布
- 响应人工作量

后续可以扩展到：

- 团队维度表现
- 升级策略合理性分析
- 业务服务影响分析

## 12. 按复刻优先级给出的产品范围建议

下面是更适合你当前阶段的拆分方式。

### 12.1 第一阶段：先复刻 PagerDuty 的基础闭环

这是最值得先做的范围。

#### 必须有

- 事件接入 API
- 统一事件模型
- `Event -> Alert -> Incident` 基础链路
- 路由规则
- 去重 `dedup_key`
- 抑制/暂停/默认兜底策略
- `Service`
- `Schedule`
- `EscalationPolicy`
- 当前 on-call 计算
- 多渠道通知基础能力
- Incident 状态流转
- Incident Timeline
- 手工声明 Incident
- 添加响应人
- 订阅人与状态更新
- 基础复盘记录
- 基础统计报表

#### 第一阶段最好有

- 自定义字段
- 模板化状态更新
- 角色分配
- 工作流触发点
- 复盘行动项

### 12.2 第二阶段：补齐 PagerDuty 的企业级体验

- 多层排班 `layer`
- 覆盖缺口检测
- round robin
- handoff 通知
- Incident Roles 全量能力
- Incident Tasks
- Internal Status Page
- Postmortem 模板
- 导出 PDF 报告
- Jira / ITSM 双向集成

### 12.3 第三阶段：进入 AIOps 增强

当基础 Incident Management 稳定后，再叠加 AIOps 更合理。建议的 AIOps 增强点包括：

- 智能告警聚合
- 相似 Incident 检索
- Probable Origin / 可能根因推荐
- 关联近期变更
- 告警降噪与异常检测
- 自动填充 Incident 摘要和状态更新
- 自动生成复盘初稿
- 自动提取行动项建议
- 自动推荐 responder / escalation policy
- 自动运行诊断工作流

换句话说，AIOps 应建立在以下基础能力之上：

- 结构化事件
- 结构化 Incident
- 可靠时间线
- 历史复盘数据
- 服务拓扑与变更数据

如果没有这些底座，AIOps 只会变成“会说话的告警页面”。

## 13. 建议你优先复刻的功能清单

如果你现在要做一个“先像 PagerDuty，再逐步进化到 AIOps”的产品，我建议优先级如下：

### P0

- 事件接入
- 编排路由
- 去重与抑制
- 服务管理
- 值班排班
- 升级策略
- 多渠道通知
- Incident 生命周期
- Timeline

### P1

- 添加响应人
- Incident 角色
- 干系人订阅
- 状态更新
- 模板化通报
- 基础复盘
- 行动项跟踪

### P2

- 复杂排班 layer
- 任务系统
- 复盘叙事构建
- 复盘阶段流转
- 报表洞察
- Slack / Teams / Jira 深度联动

### P3

- 智能聚合
- 智能根因建议
- 智能状态更新生成
- 智能复盘初稿
- 自动化诊断与修复编排

## 14. 对 AIOps 设计的直接启发

PagerDuty 给你的最大启发，不是某个单点功能，而是产品层次顺序：

1. 先把 Incident Management 做扎实。
2. 再把 Automation 接上。
3. 最后再把 AIOps 做成增强层，而不是替代底层模型。

因此你的 AIOps 平台设计更合理的路径应是：

- 第一层：事件接入与标准化
- 第二层：编排、值班、升级、协同
- 第三层：复盘、知识沉淀、行动项闭环
- 第四层：AIOps 智能增强

这比一上来就强调“AI Agent 自动处理故障”更容易真正落地。

## 15. 结论

如果把 PagerDuty 的核心功能提炼成一句话，它不是“告警平台”，而是“围绕 Incident 生命周期的编排与协作平台”。

对于你的产品路线，最合理的做法是：

1. 先复刻它的基础闭环：事件编排、值班升级、Incident 协同、复盘沉淀。
2. 再在这些结构化数据之上叠加 AIOps：聚合、关联、根因、推荐、自动化。

如果继续往下做，我建议下一份文档直接接这个主题，写成：

- `PagerDuty_MVP_产品需求清单.md`

里面把本文件再进一步收敛成：

- MVP 必做功能
- 页面清单
- 核心对象模型
- 后端服务拆分建议
- AIOps 二期能力边界

## 16. 参考来源

以下内容主要参考 PagerDuty 官方公开资料：

- Event Orchestration
- Incidents
- Schedule Basics
- Escalation Policy Basics
- Add Responders
- Communicate with Stakeholders
- Status Update Templates
- Incident Roles
- Incident Tasks
- Jeli Post-Incident Reviews and Postmortems
- Post-Incident Review Report
- Post-Incident Review Stages
- Collect Action Items
- Narrative Builder
- Insights
