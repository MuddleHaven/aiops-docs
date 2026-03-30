# Dify Lightweight Customization and Upgrade

## 1. 文档目标

本文档用于定义当前阶段基于 Dify 的最简定制与升级策略。

这份策略专门适用于下面这种现实约束：

- Dify 官方仓库在 GitHub
- 公司正式仓库必须放在内网 Gitea
- 当前没有时间做复杂的深度二开
- 主要只打算做轻量前端定制
- 一般情况下不修改 Python 后端代码

本文档的目标不是追求最完美的工程形态，而是提供一套简单、能长期执行、升级成本可控的实践方案。

本文内容结合 `langgenius/dify` 官方仓库当前公开信息整理，重点参考：

- 仓库根目录结构
- `web/` 前端目录结构
- `web/package.json`
- 官方 release 中的 upgrade guide

## 2. 核心结论

当前阶段最适合的策略是：

1. 选择一个稳定的 Dify `tag` 作为基线版本。
2. 将该版本代码放入公司内网 Gitea 仓库。
3. 只做轻量前端定制。
4. 尽量不改 Python 后端逻辑。
5. 如果后续 Dify 发布新版本，再单独开升级分支进行适配。
6. 验证通过后合并回主分支，并更新 README 中记录的 Dify 基线版本。

一句话概括：

**把 Dify 当作固定版本的上游产品来使用，只在前端做一层薄定制，升级时再重新适配这层薄定制。**

## 3. 当前推荐策略为什么成立

你的实际目标不是长期深度维护一个 Dify 二开发行版，而是：

- 借助 Dify 的现成 workflow 能力快速落地
- 补一层更适合 AIOps 场景的页面
- 保证后续还能跟上 Dify 的版本演进

在这种情况下，最重要的不是“保留最完整的上游工程关系”，而是：

- 当前基线版本清楚
- 定制范围足够小
- 升级时知道要改哪些地方

只要这三点明确，直接基于固定 tag 工作就是可行的。

## 3.1 结合 Dify 官方仓库的实际情况

当前 `langgenius/dify` 官方仓库并不是一个只有前端的仓库，而是一个完整 monorepo，根目录至少包含：

- `api/`
- `web/`
- `docker/`
- `docs/`
- `scripts/`
- `sdks/`

这意味着你当前的真实策略应该理解为：

- 你是基于整个 Dify 仓库选一个稳定 `tag`
- 但你自己的定制重点只放在 `web/`
- 后续升级时，虽然你主要改前端，但仍然要留意官方 release 对 `api/`、数据库迁移、环境变量的升级要求

换句话说：

- 你的定制范围是前端为主
- 但你的基线版本仍然是整个 Dify 项目版本

## 4. 仓库策略

## 4.1 推荐的现实做法

由于正式仓库必须放在公司内网 Gitea，当前建议采用：

- GitHub：只作为 Dify 官方代码和版本 tag 的来源
- Gitea：作为你的正式开发仓库
- 本地开发环境：用于拉取官方 tag、做定制开发、做升级适配

这意味着：

- 你的正式仓库不一定需要是 GitHub fork
- 直接把某个 Dify tag 的代码放入内网仓库也是可接受的
- 真正重要的是在文档里记录当前基线版本

## 4.2 当前基线管理方式

建议在 README 或专门说明文档中记录：

- 当前 Dify 基线版本
- 当前对应的 baseline 分支
- 当前定制范围
- 当前不做的内容

例如：

- Dify baseline: `v1.0.3`
- Baseline branch: `baseline/dify-1.0.3`
- Customization scope: `frontend only`
- Out of scope: `python backend changes`, `database schema changes`

## 4.3 推荐分支模型

结合你当前的实际情况，推荐使用三类分支：

- `baseline/dify-<version>`
- `main`
- `upgrade/dify-<version>`

这三类分支的职责分别如下。

### `baseline/dify-<version>`

用于保存某个 Dify 官方 tag 对应的纯净基线版本。

例如：

- `baseline/dify-1.13.3`
- `baseline/dify-1.13.4`

这个分支的原则是：

- 尽量保持和官方 tag 一致
- 不放你的业务定制
- 只作为“官方版本快照”存在

你可以把它理解成内网里的“官方镜像基线”。

### `main`

用于保存当前稳定可用的定制版本。

这个分支的原则是：

- 是团队默认开发和使用的主分支
- 已经包含你的轻量前端定制
- 应该始终保持可运行状态

可以理解为：

- `main = baseline + 你的轻量前端定制`

### `upgrade/dify-<version>`

用于处理新版本升级适配。

例如：

- `upgrade/dify-1.13.4`

这个分支的原则是：

- 从新的 baseline 分支拉出
- 在这个分支中把 `main` 的定制重新合进来
- 专门用于解决冲突、修页面、做验证

## 4.4 为什么这个分支模型适合当前阶段

这套模型的好处是把三件事拆开了：

- `baseline`：记录官方原始版本是什么
- `main`：记录你当前稳定定制版本是什么
- `upgrade`：记录这次升级是怎么适配的

这样做有几个现实好处：

- 你以后不会搞混“官方基线”和“你的定制版本”
- 升级时不会直接污染 `main`
- 以后回头看某次升级时，更容易知道冲突和改动来自哪里

## 4.5 分支命名建议

不要使用模糊命名，例如：

- `dify/1.2.x_baseline`

更推荐精确版本命名，例如：

- `baseline/dify-1.13.3`
- `baseline/dify-1.13.4`
- `upgrade/dify-1.13.4`

原因是：

- 模糊版本号不利于排查问题
- 不利于和 README 中记录的版本对应
- 以后很难快速判断当前分支到底对应哪个官方 tag

## 5. 定制范围边界

## 5.0 结合 Dify 实际目录的定制范围

根据 Dify 当前仓库结构，你最可能会接触的是 `web/` 下这些目录：

- `web/app/`
- `web/app/components/`
- `web/service/`
- `web/hooks/`
- `web/constants/`
- `web/context/`
- `web/i18n/`
- `web/public/`

其中：

- `web/app/` 是页面路由和页面组织核心目录
- `web/app/components/` 是页面组件和基础 UI 组件的重要入口
- `web/service/` 通常承接前端调用接口的封装
- `web/i18n/` 和 `web/i18n-config/` 关联国际化内容

如果你的目标只是“轻量前端定制”，那你应尽量把改动集中在这些目录，而不是扩散到整个仓库。

## 5.1 当前允许的轻量定制

建议只在以下范围内修改：

- 菜单入口
- 页面导航结构
- 新增少量业务页面
- workflow 相关展示页面
- 结果展示组件
- AIOps 场景下的说明文案
- 少量样式调整

结合 Dify 当前前端结构，更建议优先改这些地方：

- `web/app/` 下你新增的页面目录
- `web/app/components/` 下与你新增页面强相关的组件
- 少量导航或入口组件
- 少量 `web/service/` 下的数据读取封装

这样做的好处是：

- 你的改动会更集中
- 以后升级时更容易定位冲突文件
- 不容易被 Dify 大量公共组件改动牵连

## 5.2 当前不建议做的事情

当前阶段尽量不要做以下改动：

- 修改 Python 后端业务逻辑
- 修改数据库表结构
- 修改 Dify 核心 workflow 执行逻辑
- 修改底层鉴权、任务运行框架、队列机制
- 大范围重构前端公共基础组件

结合 Dify 当前仓库，尤其不建议你现在去碰：

- `api/` 里的运行逻辑
- `docker/` 里的部署主逻辑，除非是环境适配
- `web/package.json` 的大范围依赖调整
- `web/app/components/base/` 一类全局基础组件的大面积重构

原因很简单：

- 这些改动会显著提高未来升级成本
- 这些改动也超出了当前“轻量定制”的目标

## 5.3 最重要的判断标准

以后每做一个改动，都可以先问一句：

**如果 Dify 升级，我是不是只需要重新适配前端页面？**

如果答案是“是”，那当前改动通常是安全的。

如果答案变成：

- 还要改 Python
- 还要改数据库
- 还要改内部运行逻辑

那说明这个改动已经开始偏重，需要谨慎。

## 6. 推荐的开发流程

## 6.1 初始化阶段

建议按照以下流程开始：

1. 从 GitHub 选择一个稳定的 Dify `tag`
2. 将该版本代码放入公司内网 Gitea 仓库
3. 在 README 中记录当前 Dify 基线版本
4. 在当前基线上开始轻量前端定制

建议在 README 里明确写清楚类似信息：

- 当前 Dify baseline：例如 `1.13.3`
- 当前定制目录：例如 `web/app/aiops`、`web/app/components/aiops`
- 当前不修改：`api/`、数据库 schema、核心 workflow runtime

## 6.2 日常开发阶段

日常开发时建议：

- 以当前基线版本为基础
- 所有新增页面和改动都尽量集中在前端目录
- 保持改动点尽可能少
- 尽量避免散落式修改多个不相关页面

结合 Dify 官方前端 README，当前前端开发的基本事实也要记住：

- 前端项目在 `web/`
- 当前前端是 Next.js 项目
- `web/package.json` 使用 `pnpm`
- `web/package.json` 当前声明的 Node 版本为 `^22.22.1`
- 官方前端 README 提供了 `pnpm run dev`、`pnpm run dev:vinext`、`pnpm run dev:proxy`

这意味着你后续本地改前端页面时，应该优先围绕 `web/` 这套官方开发方式来跑，而不是自己再造一套目录结构。

## 6.3 为什么“集中改动”很重要

升级痛苦的根源通常不是“改了前端”，而是“改动散得到处都是”。

因此建议：

- 尽量集中在少数目录
- 尽量集中在少数页面
- 尽量把通用 patch 控制在小范围内

## 7. 分支策略

## 7.1 推荐分支模型

建议按下面方式使用分支：

- `baseline/dify-<version>`
  - 官方 tag 的纯净基线快照
- `main`
  - 当前稳定定制版本
- `feature/...`
  - 日常轻量前端功能开发
- `upgrade/dify-<version>`
  - 某次 Dify 升级的适配分支

如果当前主线是 Dify `1.13.3`，那么一个典型状态可能是：

- `baseline/dify-1.13.3`
- `main`
- `feature/aiops-workflow-ui`

如果准备升级到 `1.13.4`，则新增：

- `baseline/dify-1.13.4`
- `upgrade/dify-1.13.4`

## 7.2 为什么升级必须走独立分支

因为升级本质上不是普通功能开发，而是：

- 引入新的上游代码
- 重新适配已有前端改动
- 重新验证页面可用性

如果直接在 `main` 上升级，很容易把稳定版本打乱。

更准确地说，推荐做法是：

1. 先创建新的 `baseline/dify-<version>`
2. 再从这个新的 baseline 拉出 `upgrade/dify-<version>`
3. 在 upgrade 分支里把当前 `main` 的定制合进来
4. 在 upgrade 分支里解决兼容问题
5. 验证通过后再合回 `main`

## 8. 升级策略

## 8.1 什么时候升级

不要因为 Dify 发布了新版本就立刻升级。

建议只有在以下情况才升级：

- 需要某个新功能
- 需要某个关键修复
- 当前版本存在明显问题
- 团队决定统一升级基线

再补一个很现实的判断条件：

- 官方 release 明确修复了你实际会碰到的 `workflow`、`knowledge retrieval`、`web` 页面或运行问题

## 8.2 升级频率建议

建议：

- 不追 `latest`
- 尽量按明确版本升级
- 每次升级跨度不要太大

例如优先这样升级：

- `v1.0.3 -> v1.0.5`
- 再视情况评估是否升级到 `v1.1.x`

## 8.3 升级流程

推荐升级流程如下：

1. 查看 GitHub 上 Dify 新版本 tag
2. 选择目标版本
3. 创建新的 baseline 分支，例如 `baseline/dify-1.13.4`
4. 在该 baseline 分支中保存新的官方基线代码
5. 从新的 baseline 分支拉出升级分支，例如 `upgrade/dify-1.13.4`
6. 在 upgrade 分支里把当前 `main` 的定制改动合进来
7. 解决冲突并重新适配你的前端页面
8. 本地运行并验证关键页面
9. 验证通过后把 `upgrade/dify-1.13.4` 合并回 `main`
10. 更新 README 中记录的 Dify 基线版本和 baseline 分支

如果你采用的是“把官方 tag 的代码直接放进公司 Gitea”这种方式，那么第 4 步在你这里通常会表现为：

- 用新的官方 tag 覆盖当前基线代码
- 再把你自己的前端轻量改动重新适配回来

这不是最优雅的 git 方案，但对你当前阶段是可接受的。

## 8.3.1 一个完整例子

假设你当前状态如下：

- 当前官方基线：`1.13.3`
- 当前基线分支：`baseline/dify-1.13.3`
- 当前稳定定制分支：`main`

现在准备升级到 `1.13.4`，推荐流程是：

1. 新建 `baseline/dify-1.13.4`
2. 把 Dify 官方 `1.13.4` tag 的代码放到这个 baseline 分支
3. 从 `baseline/dify-1.13.4` 拉出 `upgrade/dify-1.13.4`
4. 在 `upgrade/dify-1.13.4` 上把 `main` 合进来
5. 处理前端冲突和页面兼容问题
6. 本地验证通过后，把 `upgrade/dify-1.13.4` 合并回 `main`
7. 更新 README：
   - Dify baseline = `1.13.4`
   - Baseline branch = `baseline/dify-1.13.4`

这样做的结果是：

- 旧基线仍然保留在 `baseline/dify-1.13.3`
- 新基线清晰记录在 `baseline/dify-1.13.4`
- `main` 永远代表“当前稳定可用版本`

## 8.4 升级的本质理解

升级不是“自动同步所有定制”，而是：

**换一个新的 Dify 上游版本，然后把你那层轻量前端定制重新贴上去。**

只要这层定制足够薄，升级就不会太痛苦。

## 9. 升级时重点检查什么

对当前这种轻量定制场景，不需要设计太复杂的验证体系。

升级后重点检查以下内容即可：

- 页面能否正常打开
- 菜单入口是否正常
- 新增页面是否还能访问
- 你修改过的 workflow 相关页面是否还能正常渲染
- 页面调用的数据接口是否仍然可用
- 基础流程是否正常
  - 查看 workflow
  - 查看结果
  - 查看详情页

这本质上就是一次轻量 smoke test。

## 9.1 结合 Dify release note 需要特别留意的事项

根据 Dify 最近公开 release 的 upgrade guide，后续升级时即使你“不改 Python”，也仍然需要关注这些上游变化：

- 是否有新的数据库迁移要求
  - 官方 release 中会出现 `uv run flask db upgrade`
- 是否有新的 Python 依赖同步要求
  - 官方 release 中会出现 `cd api && uv sync`
- 是否有 Sandbox 配置变更
  - 例如 release 中提到 Python 和 Node.js 默认路径变化
- 是否有队列配置要求变化
  - 例如某些版本会要求 `CELERY_QUEUES` 必须包含指定队列

这部分非常重要，因为它说明：

- 你虽然不打算改 Python 代码
- 但升级整个 Dify 基线时，不能只盯前端页面
- 仍然要读一下官方 release 的 upgrade guide

所以你最现实的升级习惯应该是：

1. 先看 release note
2. 看有没有 backend migration / env 变更
3. 再做前端适配
4. 最后一起验证

## 9.2 结合 Dify 当前前端结构的检查重点

由于 Dify 当前 `web/` 下是 App Router 风格的页面组织，升级时建议重点检查这些位置：

- `web/app/` 下你新增或修改过的页面目录
- `web/app/components/` 下你改过的业务组件
- `web/service/` 下你依赖的数据接口封装
- `web/i18n/` 下你新增的文案 key
- `web/package.json` 是否有前端主版本依赖升级

如果官方 release 里出现下面这些关键词，你要提高警惕：

- `workflow`
- `web`
- `toast`
- `base ui`
- `knowledge retrieval`
- `streaming`
- `next`
- `react`

因为这些都很可能直接影响你的页面兼容性。

## 10. README 应记录的最小信息

建议在 README 中长期保留以下最小信息：

- 当前 Dify 基线版本
- 当前 baseline 分支
- 当前定制范围
- 当前升级策略
- 当前 workflow 相关文档入口

建议格式示例：

| 项目 | 当前值 |
| --- | --- |
| Dify baseline | `v1.0.3` |
| Baseline branch | `baseline/dify-1.0.3` |
| Customization scope | `frontend pages, menu, workflow views` |
| Backend policy | `no python customization by default` |
| Upgrade policy | `upgrade in dedicated branch, merge after validation` |

## 11. 推荐保留的变更记录

为了避免以后忘记自己改过什么，建议记录两类信息：

### 11.1 当前改动范围

例如：

- 修改了哪些前端目录
- 新增了哪些页面
- 改了哪些导航入口

### 11.2 升级影响记录

例如每次升级后记录：

- 从哪个版本升级到哪个版本
- 哪些页面需要重新适配
- 是否发生接口字段变化
- 是否有已知兼容问题

这类记录不需要很长，但会非常有用。

建议你把升级记录写得更贴近 Dify 仓库实际结构，例如：

| 项目 | 记录示例 |
| --- | --- |
| From | `1.13.2` |
| To | `1.13.3` |
| Baseline branch | `baseline/dify-1.13.3` |
| Frontend touched | `web/app/...`, `web/app/components/...` |
| Upstream notes | `workflow editor fix`, `knowledge retrieval fix` |
| Backend actions | `uv sync`, `flask db upgrade` |
| Result | `passed smoke test` |

这样以后你回头看，会比抽象描述有用得多。

## 11.3 建议保留一个“自定义文件清单”

这个清单非常适合你当前模式。

建议维护一个简短列表，记录你实际改过的文件或目录，例如：

- `web/app/aiops/...`
- `web/app/components/aiops/...`
- `web/app/components/sidebar/...`
- `web/service/...`

这样以后升级时，你第一时间就知道先检查哪里。

## 12. 当前最适合的工程原则

结合当前阶段，建议坚持以下原则：

1. 先把版本固定住，不要追最新。
2. 先把前端薄定制做好，不要做后端深度二开。
3. 升级时一定走独立分支。
4. 升级完成后同步更新 README 和基线版本说明。
5. 如果未来发现前端定制越来越重，再考虑把更多页面迁到你自己的平台前端。

## 13. 什么情况下需要调整当前策略

当前这套轻量策略适合“前端轻改 + 后端基本不动”的阶段。

当出现下面任一情况时，应考虑升级策略：

- 你开始频繁修改 Python 后端
- 你需要控制 Dify 的内部权限或运行逻辑
- 你需要大量自定义 workflow 管理页面
- 你发现每次升级都要改很多核心文件

如果发生这些情况，说明你已经逐渐从“轻量定制”进入“中度二开”，那时再重新设计仓库与升级策略更合适。

## 14. 结论

当前阶段，最现实、最省时间、也最适合长期推进的做法就是：

- 用一个固定 Dify `tag` 做基线
- 把代码放进公司内网 Gitea
- 只做轻量前端定制
- 后续如果 Dify 有新版本，就开升级分支重新适配前端改动
- 验证通过后合并回 `main`
- 最后更新 README 中的基线版本说明

这不是最复杂的工程方案，但它是当前最务实、最容易执行、最符合你时间约束的方案。

## 15. 参考依据

本文主要基于 `https://github.com/langgenius/dify` 官方公开信息整理，重点参考：

- 根目录结构：`api/`、`web/`、`docker/`、`docs/`
- `web/README.md`
- `web/package.json`
- 官方 release 中的 upgrade guide 与 release notes
