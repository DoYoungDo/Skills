---
name: go-code-analysis
description: 当用户要求分析、查看或校验 Go 代码时触发，核心规则是优先使用 gopls MCP 工具获取 Go 语义上下文。
license: MIT
---

# Go Code Analysis

本 skill 用于约束 Go 代码相关任务的工具选择。核心功能只有一个：当用户要求分析、查看或校验 Go 代码时，必须优先使用 `gopls` MCP 工具，而不是先用普通文件读取、文本搜索或 shell 命令。

## 触发时机

用户请求包含以下任一意图时，`MUST` 触发本 skill：

- 分析 Go 文件、Go 代码、Go package、Go module 或 Go workspace。
- 查看 `.go` 文件的结构、依赖、符号、引用、调用关系或 package API。
- 校验 Go 文件是否存在编译、类型、诊断、接口、并发、错误处理或测试覆盖问题。
- 审查 Go 代码质量、bug 风险、重构影响、API 设计或实现正确性。
- 解释 Go 函数、类型、接口、方法、常量、变量、测试或调用链。

如果用户只是说“看一下这个文件”，但目标是 `.go` 文件，`SHOULD` 触发本 skill。

## 核心规则

- `MUST` 优先使用 `gopls` MCP 工具建立 Go 语义上下文。
- `MUST` 优先相信 `gopls` 提供的 workspace、package、symbol、reference、diagnostic 和 API 信息。
- `MUST` 在使用普通文件读取、Glob、Grep、Bash 或其他文本级工具前，先判断是否已有合适的 `gopls` MCP 工具可用。
- `MUST` 不把文本搜索结果等同于 Go 语义引用关系。
- `MUST` 不只凭文件名、函数名或搜索命中断言 Go 代码行为。
- `SHOULD` 在 `gopls` 结果不足时，再读取具体源码补充实现细节。
- `SHOULD` 对分析结论标明依据来自 `gopls`、源码阅读、测试结果还是推断。

## gopls MCP 优先级

按任务类型优先选择以下工具：

1. Workspace 级理解：优先使用 `gopls_go_workspace`。
2. 编译、类型和诊断校验：优先使用 `gopls_go_diagnostics`。
3. 单个文件上下文：优先使用 `gopls_go_file_context`。
4. package 对外 API：优先使用 `gopls_go_package_api`。
5. 符号查找：优先使用 `gopls_go_search`。
6. 引用和影响范围：优先使用 `gopls_go_symbol_references`。
7. 符号重命名：优先使用 `gopls_go_rename_symbol`。
8. 依赖漏洞检查：优先使用 `gopls_go_vulncheck`。

## 允许补充使用的工具

只有在以下情况，才使用普通文件读取、Glob、Grep、Bash 或其他工具补充：

- `gopls` 无法直接返回具体实现内容，需要读取源码确认细节。
- 需要查看非 Go 文件，例如配置、文档、生成模板、脚本或测试数据。
- 需要查找字符串字面量、注释、日志文案、配置 key 或生成代码痕迹。
- 需要运行 `go test`、`go build`、lint 或项目已有验证命令。
- `gopls` 工具不可用、失败或返回信息不足。

使用补充工具后，`MUST` 避免覆盖或忽略已有 `gopls` 语义结果；如果两者冲突，必须说明冲突并继续核实。

## 推荐流程

```text
识别 Go 任务 -> 调用 gopls MCP -> 补充读取源码 -> 必要时运行验证 -> 输出结论
```

具体要求：

- 分析前先确认目标文件、package、symbol 或问题范围。
- 查看代码时先通过 `gopls` 获取 workspace、文件上下文、API 或引用关系。
- 校验代码时先使用 `gopls_go_diagnostics`，再根据需要运行测试或构建。
- 分析影响范围时优先使用 `gopls_go_symbol_references`，再补充文本搜索。
- 涉及重命名时优先使用 `gopls_go_rename_symbol`，不要手工批量替换 Go 符号。

## 输出要求

- 结论必须简洁，优先回答用户的问题。
- 关键结论应附带文件路径和行号，或说明来自哪个 `gopls` MCP 结果。
- 如果 `gopls` 发现诊断问题，必须优先报告。
- 如果没有发现明确问题，需要说明“未发现明确问题”，并列出未覆盖的验证范围。
- 如果无法使用 `gopls` MCP 工具，需要说明原因，并标记分析可靠性降低。

## 禁止行为

- 不在 Go 代码分析、查看或校验任务中绕过可用的 `gopls` MCP 工具。
- 不仅凭 `grep`、Glob 或文件读取结果判断 Go symbol 的完整引用关系。
- 不编造 `gopls` 诊断、测试结果、调用链或 package API。
- 不为了快速回答而忽略 `gopls` 返回的错误或诊断。
- 不执行与用户请求无关的重构、格式化、提交或破坏性操作。
