# AIOps Docs

这个仓库用于沉淀 AIOps 项目的产品方案、架构设计、实施路线和配套架构图，作为内部讨论、方案迭代和后续产品化的统一文档入口。

## 当前文档

- `AIOps_Product_Architecture_and_Commercialization.md`：产品架构与商业化主文档
- `AIOps_Architecture_Diagram_Explanation.md`：当前架构图逐层详解与推导说明
- `AIOps_Project_Proposal.md`：项目提案文档
- `AIOps_Practical_Route_Architecture.png`：当前统一参考架构图

## 使用建议

- 统一围绕 `AIOps_Practical_Route_Architecture.png` 讨论和更新文档
- 架构图有较大调整时，同步更新 `AIOps_Architecture_Diagram_Explanation.md`
- 产品定位、范围、实施路径变化时，同步更新主文档

## 仓库定位

- 这是一个文档仓库，不是代码仓库
- 图中的模块首先表达逻辑职责，不强制等于最终微服务拆分
- MVP 阶段允许多个模块在实现上合并
