# AIOps Docs

这个仓库用于沉淀 AIOps 项目的产品方案、架构设计、实施路线和配套架构图，作为内部讨论、方案迭代和后续产品化的统一文档入口。

## 快速入口

### 核心架构文档

- [`AIOps_Product_Architecture_and_Commercialization.md`](AIOps_Product_Architecture_and_Commercialization.md)：产品架构与商业化主文档
- [`AIOps_Architecture_Diagram_Explanation.md`](AIOps_Architecture_Diagram_Explanation.md)：当前架构图逐层详解与推导说明
- [`AIOps_Project_Proposal.md`](AIOps_Project_Proposal.md)：项目提案文档
- [`AIOps_Practical_Route_Architecture.png`](AIOps_Practical_Route_Architecture.png)：当前统一参考架构图

### PagerDuty 参考文档

- [`pagerduty/PagerDuty_Feature_Breakdown_and_Replication_Requirements.md`](
  pagerduty/PagerDuty_Feature_Breakdown_and_Replication_Requirements.md
  )：PagerDuty 功能拆解与复刻需求
- [`pagerduty/PagerDuty_MVP_Product_Requirements.md`](
  pagerduty/PagerDuty_MVP_Product_Requirements.md
  )：PagerDuty 风格 MVP 产品需求清单
- [`pagerduty/PagerDuty_Information_Architecture_and_Page_Design.md`](
  pagerduty/PagerDuty_Information_Architecture_and_Page_Design.md
  )：PagerDuty 页面与信息架构设计

### Workflow 文档

- [`workflow/AIOps_Workflow_Requirements.md`](workflow/AIOps_Workflow_Requirements.md)
  ：`aiops-workflow` 需求清单与范围定义
- [`workflow/Dify_Lightweight_Customization_and_Upgrade.md`](workflow/Dify_Lightweight_Customization_and_Upgrade.md)
  ：基于固定 Dify tag 的轻量定制与升级策略

### Platform 文档

- [`platform/AIOps_Platform_Requirements.md`](platform/AIOps_Platform_Requirements.md)
  ：`aiops-platform` 需求清单与范围定义

### Tools 文档

- [`tools/AIOps_Tools_Requirements.md`](tools/AIOps_Tools_Requirements.md)
  ：`aiops-tools` 需求清单与范围定义

## Dify Baseline Management

当前与 Dify 相关的轻量定制和升级策略，统一参考：

- [`workflow/Dify_Lightweight_Customization_and_Upgrade.md`](workflow/Dify_Lightweight_Customization_and_Upgrade.md)

建议在后续实际落库时，持续维护以下信息：

| 项目 | 建议记录 |
| --- | --- |
| Dify baseline | 当前采用的官方 tag，例如 `1.13.3` |
| Baseline branch | 对应基线分支，例如 `baseline/dify-1.13.3` |
| Main branch role | 当前稳定定制版本 |
| Upgrade branch | 本次升级分支，例如 `upgrade/dify-1.13.4` |
| Customization scope | 当前只改哪些前端目录 |
| Backend policy | 是否允许修改 Python 后端 |

当前推荐的分支模型：

- `baseline/dify-<version>`：保存官方基线快照
- `main`：保存当前稳定定制版本
- `upgrade/dify-<version>`：处理升级适配和兼容修复

## 目录说明

- `pagerduty/`：PagerDuty 对标拆解、MVP 范围与页面设计文档
- `workflow/`：`aiops-workflow` 相关需求与设计文档
- `platform/`：`aiops-platform` 相关需求与主控流程文档
- `tools/`：`aiops-tools` 相关需求与接口治理文档

## 使用建议

- 统一围绕 `AIOps_Practical_Route_Architecture.png` 讨论和更新文档
- 架构图有较大调整时，同步更新 `AIOps_Architecture_Diagram_Explanation.md`
- 产品定位、范围、实施路径变化时，同步更新主文档
- 专题文档优先沉淀到对应目录，并保持 README 快速入口同步

## 仓库定位

- 这是一个文档仓库，不是代码仓库
- 图中的模块首先表达逻辑职责，不强制等于最终微服务拆分
- MVP 阶段允许多个模块在实现上合并
- 新建文件和目录默认使用英文名称，内容可以使用中文
