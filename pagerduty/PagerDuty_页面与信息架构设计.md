# PagerDuty 页面与信息架构设计

## 1. 文档目标

本文档在 `PagerDuty_MVP_产品需求清单.md` 基础上，定义 PagerDuty 风格 Incident Management MVP 的页面结构、导航方式、核心信息组织方式和关键页面布局。

目标是帮助产品、设计和研发统一以下内容：

1. 整个产品有哪些主要页面。
2. 这些页面如何通过导航组织起来。
3. 每个页面最重要的信息模块是什么。
4. 用户完成关键任务时会经过哪些页面。

## 2. 信息架构设计原则

### 2.1 以 Incident 为中心

系统中的大部分操作最终都应回到 `Incident`：

- 事件进来是为了产生或更新 Incident
- 值班和升级是为了找到 Incident 的处理人
- 状态更新和订阅是围绕 Incident 协同
- 复盘是 Incident 关闭后的延续

因此导航和页面设计都应让用户快速进入 Incident。

### 2.2 以任务流而不是配置流组织页面

用户最常见的任务顺序不是“先配规则，再看用户，再看服务”，而是：

1. 看见问题
2. 确认谁在处理
3. 找到上下文
4. 继续协作
5. 最后回看规则或排班是否合理

因此首页和主导航应优先暴露运行态页面，而不是后台配置页。

### 2.3 运行态与配置态分离

建议把产品分成两种视角：

- 运行态
  - Incidents
  - Services
  - On-call
  - Reports
  - Reviews
- 配置态
  - Event Orchestration
  - Escalation Policies
  - Schedules
  - Users / Teams

这样能避免管理配置和处理故障混在一起。

## 3. 顶层导航建议

MVP 推荐左侧一级导航如下：

1. `Incidents`
2. `Services`
3. `On-call`
4. `Reviews`
5. `Analytics`
6. `Settings`

其中 `Settings` 下再承载配置型模块。

### 3.1 一级导航含义

| 一级导航 | 目标用户 | 核心价值 |
| --- | --- | --- |
| `Incidents` | 一线响应人、团队负责人 | 看当前故障、处理故障 |
| `Services` | 服务负责人、管理员 | 看服务与关联 Incident |
| `On-call` | 值班人、排班管理员 | 看当前值班与排班 |
| `Reviews` | 负责人、管理者 | 管理复盘与行动项 |
| `Analytics` | 管理者、平台运营 | 查看运营指标 |
| `Settings` | 管理员 | 管理编排、升级、人员等配置 |

## 4. 页面地图

## 4.1 一级页面结构

```text
Incidents
  Incident List
  Incident Detail
  Create Incident

Services
  Service List
  Service Detail
  Create/Edit Service

On-call
  On-call Overview
  Schedule List
  Schedule Detail
  Escalation Policy List
  Escalation Policy Detail

Reviews
  Review List
  Review Detail

Analytics
  Dashboard

Settings
  Event Orchestration
  Users
  Teams
  Notification Settings
  Audit Log
```

## 4.2 关键跳转关系

- Incident List -> Incident Detail
- Incident Detail -> Service Detail
- Incident Detail -> Review Detail
- Service Detail -> Escalation Policy Detail
- Escalation Policy Detail -> Schedule Detail
- On-call Overview -> Schedule Detail
- Review Detail -> Incident Detail

## 5. 页面详细设计

## 5.1 `Incidents` 列表页

### 5.1.1 页面目标

这是最重要的运行态入口，用户要能在这里快速回答：

- 现在有哪些未关闭 Incident
- 哪些最紧急
- 哪些无人接手
- 哪些已经在升级

### 5.1.2 页面模块

建议包含以下区域：

1. 顶部筛选栏
2. 统计摘要卡片
3. Incident 列表表格
4. 快速操作区

### 5.1.3 筛选条件

- 状态
- 优先级
- 服务
- 团队
- 是否已确认
- 是否升级中
- 时间范围

### 5.1.4 列表字段

- Incident 标题
- 服务
- 状态
- 优先级
- 当前 assignee
- 当前 on-call
- 触发时间
- 最近更新时间
- 升级层级/升级状态

### 5.1.5 快速操作

- 创建 Incident
- Acknowledge
- Resolve
- Add responder
- Add subscriber

## 5.2 `Incident Detail` 详情页

### 5.2.1 页面目标

这是平台最核心的页面，用户要在一个页面内完成大部分响应动作。

### 5.2.2 页面布局建议

建议采用三栏或二栏布局。

方案 A：三栏布局

- 左栏：基础信息和状态卡
- 中栏：Timeline 与状态更新
- 右栏：响应人、订阅人、关联对象

方案 B：二栏布局

- 主栏：Timeline
- 侧栏：基础信息 + 人员 + 快速操作

MVP 建议先做二栏布局，复杂度更低。

### 5.2.3 顶部头部信息

- Incident 标题
- 状态标签
- 优先级
- 所属服务
- 创建时间
- 当前负责人

### 5.2.4 侧栏模块

- 基础信息卡片
- Responders
- Subscribers
- Escalation 状态
- 关联链接
  - Service
  - Schedule
  - Escalation Policy
  - Review

### 5.2.5 主栏模块

- Status Updates
- Unified Timeline
- Notes

### 5.2.6 页面动作

- Acknowledge
- Resolve
- Reopen
- Add responder
- Add subscriber
- Publish status update
- Add note
- Create review

## 5.3 `Create Incident` 页面/弹窗

### 5.3.1 页面目标

允许用户手工声明事故，不依赖外部事件接入。

### 5.3.2 表单字段

- 标题
- 服务
- 优先级
- 描述
- 指派对象
  - 用户或升级策略
- 附加响应人
- 订阅人

### 5.3.3 交互建议

MVP 可采用右侧抽屉或模态框，不必单独做复杂页面。

## 5.4 `Services` 列表页

### 5.4.1 页面目标

用于查看和维护服务对象，回答：

- 系统里有哪些服务
- 每个服务归谁负责
- 当前是否有打开的 Incident

### 5.4.2 列表字段

- 服务名
- 所属团队
- 默认升级策略
- 当前 on-call
- 打开 Incident 数

### 5.4.3 页面动作

- 创建服务
- 编辑服务
- 查看服务详情

## 5.5 `Service Detail` 页面

### 5.5.1 页面目标

展示一个服务的运行态概况和配置入口。

### 5.5.2 页面模块

- 服务基础信息
- 关联升级策略
- 当前 on-call
- 最近 Incident
- 服务维度指标卡

### 5.5.3 关键跳转

- 跳转到 `Escalation Policy Detail`
- 跳转到 `Schedule Detail`
- 跳转到该服务的 Incident 列表

## 5.6 `On-call Overview` 页面

### 5.6.1 页面目标

快速回答：

- 现在谁在值班
- 哪些服务的 on-call 正常
- 有没有排班空洞

### 5.6.2 页面模块

- 当前 on-call 总览
- 即将交班提醒
- 关键服务 on-call 列表
- 异常排班提示

### 5.6.3 适合用户

- 值班经理
- 团队负责人
- 平台管理员

## 5.7 `Schedule List` 列表页

### 5.7.1 列表字段

- 值班表名称
- 时区
- 当前 on-call
- 下一位 on-call
- 最近更新时间

### 5.7.2 页面动作

- 创建值班表
- 编辑值班表
- 查看排班详情

## 5.8 `Schedule Detail` 详情页

### 5.8.1 页面目标

帮助管理员理解和维护排班结构。

### 5.8.2 页面模块

- 基础信息卡片
- 当前 on-call 信息
- 排班时间视图
- 临时替班管理
- 关联升级策略列表

### 5.8.3 页面布局建议

- 顶部摘要卡
- 中部日历/时间轴
- 下部参与人配置区

## 5.9 `Escalation Policy List` 列表页

### 5.9.1 页面目标

用于维护升级规则链。

### 5.9.2 列表字段

- 策略名称
- 关联服务数
- 规则级数
- 更新时间

## 5.10 `Escalation Policy Detail` 详情页

### 5.10.1 页面目标

清晰展示升级链结构，让管理员一眼知道超时后会通知谁。

### 5.10.2 页面模块

- 基础信息
- 升级层级可视化
- 关联服务
- 关联值班表

### 5.10.3 可视化建议

建议采用纵向步骤流：

```text
Level 1 -> wait 5m -> Level 2 -> wait 10m -> Level 3
```

比表格更适合理解升级链。

## 5.11 `Event Orchestration` 页面

### 5.11.1 页面目标

让管理员维护事件路由与编排规则。

### 5.11.2 页面结构建议

- 顶部规则说明区
- 规则列表
- 规则详情编辑抽屉
- 测试事件模拟区

### 5.11.3 列表字段

- 规则名称
- 状态
- 优先级
- 条件摘要
- 动作摘要
- 更新时间

### 5.11.4 MVP 特别建议

即使第一版不做复杂可视化规则编辑器，也最好提供：

- 条件摘要展示
- 命中结果预览
- 测试样例事件

否则规则很难维护。

## 5.12 `Users` 页面

### 5.12.1 列表字段

- 用户名
- 邮箱
- 团队
- 联系方式状态
- 当前是否 on-call

### 5.12.2 详情能力

- 联系方式配置
- 通知方式偏好
- 所属团队
- 关联值班表

## 5.13 `Teams` 页面

### 5.13.1 页面模块

- 团队列表
- 成员列表
- 关联服务
- 关联升级策略

## 5.14 `Reviews` 列表页

### 5.14.1 页面目标

帮助团队查看哪些 Incident 已复盘，哪些还未完成。

### 5.14.2 筛选条件

- 复盘状态
- 所属团队
- 服务
- 时间范围

### 5.14.3 列表字段

- 复盘标题
- 关联 Incident
- 负责人
- 状态
- 最近更新时间

## 5.15 `Review Detail` 页面

### 5.15.1 页面目标

承载 MVP 版复盘记录与行动项。

### 5.15.2 页面模块

- 基础信息
- Incident 摘要
- 时间与影响
- Takeaways
- Action Items
- 状态流转

### 5.15.3 第一版建议

MVP 不做复杂多标签页，直接一个长页即可，降低实现复杂度。

## 5.16 `Analytics Dashboard` 页面

### 5.16.1 页面目标

面向负责人和管理者展示运行指标。

### 5.16.2 页面模块

- 指标卡
  - Incident 总量
  - MTTA
  - MTTR
  - 升级次数
- 趋势图
- 服务排行
- 响应人负载分布

## 5.17 `Audit Log` 页面

### 5.17.1 页面目标

给管理员查看核心配置变更与关键运行事件。

### 5.17.2 页面范围

- 规则配置变更
- 值班表变更
- 升级策略变更
- Incident 关键状态变更

## 6. 核心用户任务流

## 6.1 任务流一：处理新 Incident

```text
Incident List
  -> Incident Detail
  -> Acknowledge
  -> Add responder / Publish status update
  -> Resolve
  -> Create review
```

## 6.2 任务流二：排查通知为什么没命中

```text
Incident Detail
  -> Service Detail
  -> Escalation Policy Detail
  -> Schedule Detail
```

## 6.3 任务流三：新增一个服务接入

```text
Services
  -> Create Service
  -> Bind Escalation Policy
  -> Configure Event Orchestration Rule
  -> Send test event
```

## 6.4 任务流四：进行复盘

```text
Incident Detail
  -> Create Review
  -> Review Detail
  -> Add Takeaways
  -> Add Action Items
  -> Mark Completed
```

## 7. 信息层级建议

### 7.1 Incident 详情页的信息优先级

P0 信息：

- 当前状态
- 谁在处理
- 服务
- 最近状态更新
- 时间线

P1 信息：

- 升级情况
- 订阅人
- 关联复盘

P2 信息：

- 详细配置关系
- 历史关联对象

### 7.2 配置页面的信息优先级

P0 信息：

- 当前配置是什么
- 影响哪些对象
- 是否启用

P1 信息：

- 编辑历史
- 测试验证结果

## 8. 设计与前端实现建议

### 8.1 MVP 优先用表格 + 详情页

对这类运维产品，第一版不需要追求过度炫技的卡片化布局。对 Incident、Service、Schedule、Policy 这类对象，`列表 + 详情` 是最稳的结构。

### 8.2 时间线组件应尽早统一

Incident Timeline 是全系统最关键的基础展示组件，建议尽早沉淀统一样式，复用到：

- Incident 详情
- 复盘详情
- 审计视图

### 8.3 关系跳转要强

这类产品的用户经常在对象之间来回跳转，因此页面内应加强：

- Service 链接
- Schedule 链接
- Escalation Policy 链接
- Review 链接

不要让用户只能靠返回列表找对象。

## 9. MVP 页面清单总结

推荐首批交付页面如下：

1. `Incident List`
2. `Incident Detail`
3. `Create Incident`
4. `Service List`
5. `Service Detail`
6. `On-call Overview`
7. `Schedule List`
8. `Schedule Detail`
9. `Escalation Policy List`
10. `Escalation Policy Detail`
11. `Event Orchestration`
12. `Users`
13. `Teams`
14. `Review List`
15. `Review Detail`
16. `Analytics Dashboard`
17. `Audit Log`

## 10. 结论

如果从信息架构角度总结 PagerDuty 风格产品，最关键的不是页面多，而是“是否让用户在最短路径上找到故障、找到责任人、找到上下文、找到下一步动作”。

因此 MVP 页面设计应坚持三个原则：

1. 运行态优先于配置态。
2. Incident 详情页优先于花哨首页。
3. 对象之间必须强链接跳转。

只要这三点做对，后续再扩展 Status Page、Narrative Builder、AIOps 面板都会比较自然。
