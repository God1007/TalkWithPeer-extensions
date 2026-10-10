# TalkWithPeer Extensions

本机审查、按内容锁定的 Skills / Tools / Hooks。运行、审查状态和执行审计由 [TalkWithPeer](https://github.com/God1007/TalkWithPeer) 管理。

## 使用

    git clone https://github.com/God1007/TalkWithPeer-extensions.git

在 TalkWithPeer 终端中：

    /ext add /绝对路径/TalkWithPeer-extensions
    /ext list
    /ext review 1
    /ext approve 1 <当前完整哈希> 审查备注
    /ext enable 1 <当前完整哈希>

网页入口是 **设置 → 管理 Skills / Tools / Hooks**。登记本机路径后查看全部文件与上次审查内容，再批准当前哈希、启用锁定版本。批准不自动启用。

所有包默认关闭。修改任何包文件会改变内容哈希，旧批准不再适用于当前内容；正在执行的阶段会核对锁定清单，避免静默使用变化后的扩展。Git commit 记录来源，内容哈希约束实际本机文件，包含未提交变化。

## 扩展包

每个包是一个目录，包含 extension.json 和入口。平台将包内所有 Markdown / JSON 文件纳入审查和哈希，拒绝符号链接、脚本、隐藏文件（忽略 .DS_Store）和过大文件。

    skills/<id>/extension.json + SKILL.md
    tools/<id>/extension.json + definition.json
    hooks/<id>/extension.json + definition.json

extension.json 字段：schemaVersion=1、id、type、name、version、description、entry。id 使用小写字母、数字和连字符，版本为三段数字。登记后的运行 ID 还包含本机库 ID，避免不同库同名碰撞。

## 一期内容

| 包              | 类型  | 用途                                           |
| --------------- | ----- | ---------------------------------------------- |
| evidence-review | skill | 核对证据，区分原文、摘要、推断与未验证项       |
| simplify-plan   | skill | 方案讨论时核对复用依据，说明必要改动与维护成本 |
| public-read     | tool  | 本会话公共原文读取，每次 1–3 条                |
| public-search   | tool  | 公共消息关键词搜索，明确标记片段               |
| project-read    | tool  | 所选项目的只读文本快照，附带哈希               |
| evidence-note   | hook  | before_prompt 注入非权威的证据边界标注         |
| reply-audit     | hook  | after_reply 记录锁定版本执行                   |

一期工具定义只选择平台可信实现的 public_read / public_search / project_read。它们不运行任意 shell、JavaScript 或 Python。

Hook 支持 before_prompt、before_tool、after_tool、after_reply、after_compact：

- note：仅 before_prompt，静态非权威标注，计入上下文预算。
- audit：记录 hook 触发，不改写目标、候选或判断。
- deny：仅 before_tool，可通过 when.toolId 指定要拦截的工具 ID 或包名。

例如：

    {"event":"before_tool","operation":"deny","when":{"toolId":"project-read"}}

启用的 skill 作为审查后的工作流程传入平台构建的输入，不覆盖用户约束。平台 tool 与 hooks 作用于平台运行循环；原生 Agent 内部自己的工具调用不由这些 hook 接管。

## 使用方案简化 skill

在扩展库中找到“方案简化”，查看源文件、批准当前哈希后启用。讨论时可以写：

    使用 simplify-plan 讨论下面的方案，重点降低复杂度、复用现有能力；需要修改时说明必要的最小改动与验证方式。

启用的 skill 正文随平台输入提供，成员按适用条件使用；当前不是按名称动态加载。它不增加专门的讨论模式，也不要求普通任务输出方案字段。讨论中的改动是建议清单，不自动执行文件修改。

## 贡献

提交小而可核对的包。Skill 应说明适用场景、动作和完成条件；tool / hook 应使用平台支持的声明式操作。每次内容变化需重新审查。不要提交凭证、个人数据或导入的个人宠物素材。

目前仓库公开源码，尚未指定开源许可证；未提供任意脚本插件、远程安装市场或多用户权限管理。
