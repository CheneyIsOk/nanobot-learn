# nanobot Agent 体系学习计划

> 学习分支：`nanobot-learn`  
> 代码基线：`93571149`  
> 制定日期：2026-07-21  
> 当前阶段：阶段 0（学习基线与路线规划）

## 1. 学习目标

本学习计划以“一条用户消息如何被 Agent 接收、理解、执行并返回”为主线，
逐步掌握 nanobot 的核心设计与工程实现。完成全部阶段后，应具备以下能力：

1. 能独立说明一次 Agent Turn 的完整调用链。
2. 能区分 `AgentLoop`、`AgentRunner`、Provider、Tool、Session 和 Memory 的职责。
3. 能从用户可见现象快速定位到对应源码和测试。
4. 能使用 Mock 或本地环境验证 Agent 的关键机制，而不依赖真实付费 API。
5. 能识别工具执行、文件访问、网络请求和渠道接入中的安全边界。
6. 能遵循 nanobot 现有扩展机制，实现并验证一个小型 Agent 扩展。

## 2. 学习策略

采用“主流程驱动 + 子系统扩展”的方式，先建立完整心智模型，再深入各模块。

统一学习循环如下：

```text
提出问题 → 阅读官方文档 → 定位源码 → 绘制流程 → 运行最小实验
        → 对照测试 → 记录结论 → 完成阶段验收
```

学习过程中遵循以下原则：

- **围绕问题阅读**：每次阅读源码前先明确需要回答的问题。
- **以运行证据为准**：源码结论尽量通过现有测试或最小实验验证。
- **先主干后分支**：先理解 CLI、Bus、Loop、Runner，再研究外围能力。
- **区分事实与推测**：笔记中明确标记源码事实、实验结果和个人理解。
- **控制实验范围**：优先使用 Mock、临时目录和本地 Provider。
- **及时沉淀**：每完成一个阶段，形成独立文档和可复查的验收记录。

## 3. 文档组织

所有学习资料统一存放在 `learn-docs/`：

```text
learn-docs/
├── overview-guide.md
├── 00-learning-plan.md
├── 01-cli-and-message-bus.md
├── 02-agent-loop-and-context.md
├── 03-agent-runner-and-tools.md
├── 04-session-and-memory.md
├── 05-provider-and-config.md
├── 06-channel-gateway-webui.md
├── 07-security-and-extension.md
├── 08-architecture-and-design-philosophy.md
├── 09-capstone-review.md
└── experiments/
```

每份阶段笔记使用统一结构：

1. 本阶段目标。
2. 需要回答的问题。
3. 核心概念。
4. 源码地图。
5. 调用流程或时序图。
6. 最小实验。
7. 测试证据。
8. 结论与遗留问题。
9. 阶段验收。

## 4. 阶段路线

### 阶段 0：学习基线与路线规划

**目标**

建立独立学习分支、固定代码基线、整理已有笔记，并明确后续学习顺序。

**主要内容**

- 确认 `nanobot-learn` 分支与代码基线。
- 将学习资料统一放入 `learn-docs/`。
- 保留并校正已有的 `overview-guide.md`。
- 确认 Python、pytest、ruff 和 nanobot CLI 的运行环境。

**产物**

- `00-learning-plan.md`
- 更新后的 `overview-guide.md`
- 环境与测试基线记录

**验收条件**

- [x] 当前分支为 `nanobot-learn`。
- [x] 已记录代码基线提交。
- [x] 已明确文档目录和阶段路线。
- [x] 修正 `overview-guide.md` 中的旧目录说明和失效链接。
- [x] 验证 `nanobot-learn-dev` 环境、Python、pytest 和 ruff 版本。
- [x] CLI 入口验证：`nanobot-learn-dev` 中 CLI 可正常加载（退出码 0）。
- [x] Bus 出站事件定向测试通过（9 passed）。

### 阶段 1：CLI 入口与 MessageBus

**核心问题**

- `python -m nanobot` 如何进入 Typer 命令？
- 单次模式、交互模式和 Gateway 模式如何产生消息？
- `InboundMessage` 与 `OutboundMessage` 分别承载什么信息？
- MessageBus 为什么只需要两条 `asyncio.Queue`？

**重点源码**

- `nanobot/__main__.py`
- `nanobot/cli/commands.py`
- `nanobot/bus/events.py`
- `nanobot/bus/queue.py`
- `nanobot/bus/outbound_events.py`

**最小实验**

- 检查 CLI 帮助和命令注册结果。
- 在独立事件循环中发布并消费一条入站、出站消息。
- 运行 `tests/bus/` 中与事件结构相关的测试。

**产物与验收**

- 文档：`01-cli-and-message-bus.md`。
- 能画出 CLI → Bus → AgentLoop → Bus → CLI 的基础时序。
- 能解释 `session_key` 的来源和消息路由发生的位置。

### 阶段 2：AgentLoop 与上下文构建

**核心问题**

- `AgentLoop` 如何接收、分发并串行化同一会话的消息？
- Turn 状态机包含哪些状态，状态之间如何转换？
- 系统提示词、历史消息、技能和当前输入如何组合？
- 命令短路、消息注入、checkpoint 和取消机制如何工作？

**重点源码**

- `nanobot/agent/loop.py`
- `nanobot/agent/context.py`
- `nanobot/agent/turn_hooks.py`
- `nanobot/command/router.py`
- `nanobot/command/builtin.py`

**最小实验**

- 使用测试中的 Fake 或 Mock Runner 驱动一次 Agent Turn。
- 观察构建完成的消息列表和系统提示词组成。
- 运行 `tests/agent/test_context_builder.py` 和选定的 Loop 测试。

**产物与验收**

- 文档：`02-agent-loop-and-context.md`。
- 能画出 Turn 状态机并说明每个状态的输入、输出和副作用。
- 能说明 `AgentLoop` 与 `AgentRunner` 的职责边界。

### 阶段 3：AgentRunner 与工具循环

**核心问题**

- Runner 如何向 Provider 发起请求并处理流式响应？
- 模型返回 tool call 后，工具如何被查找、校验和执行？
- 工具结果如何回填上下文并触发下一次模型请求？
- 最大迭代次数、取消、超时和工具异常如何终止循环？

**重点源码**

- `nanobot/agent/runner.py`
- `nanobot/agent/tools/base.py`
- `nanobot/agent/tools/schema.py`
- `nanobot/agent/tools/registry.py`
- `nanobot/agent/tools/loader.py`

**最小实验**

- 使用 Mock Provider 依次返回工具调用和最终文本。
- 注册一个无副作用工具，观察参数校验和结果回填。
- 运行 `tests/tools/test_tool_registry.py`、
  `tests/tools/test_tool_validation.py` 和 Runner 集成测试。

**产物与验收**

- 文档：`03-agent-runner-and-tools.md`。
- 能完整解释“模型 → 工具 → 模型”的多轮迭代。
- 能指出工具名称、JSON Schema 和错误信息为何属于模型契约。

### 阶段 4：Session、Context 与 Memory

**核心问题**

- Session 与长期 Memory 分别保存什么？
- 会话历史如何写入、加载、压缩和恢复？
- Context 超限时如何治理？
- Dream 如何把历史记录整理为长期记忆？

**重点源码**

- `nanobot/session/manager.py`
- `nanobot/session/`
- `nanobot/agent/memory.py`
- `nanobot/templates/memory/`
- `docs/memory.md`

**最小实验**

- 使用临时 workspace 创建、保存并重新加载一个 Session。
- 查看 JSONL、MEMORY.md 和 history.jsonl 的职责差异。
- 运行 `tests/session/` 以及选定的 Dream、压缩测试。

**产物与验收**

- 文档：`04-session-and-memory.md`。
- 能区分短期上下文、持久化会话和长期记忆。
- 能说明一次自动压缩或 Dream 整理的触发与结果。

### 阶段 5：Provider、配置与流式响应

**核心问题**

- 配置文件如何被加载、校验和转换？
- Provider 如何根据 preset、模型名和 API Base 被选择？
- 通用 OpenAI 兼容实现与专用 Provider 有何区别？
- 重试、限流、流式增量和错误元数据如何统一处理？

**重点源码**

- `nanobot/config/schema.py`
- `nanobot/config/loader.py`
- `nanobot/providers/base.py`
- `nanobot/providers/registry.py`
- `nanobot/providers/factory.py`
- `nanobot/providers/openai_compat_provider.py`

**最小实验**

- 在临时目录加载一份最小配置并检查解析结果。
- 使用 Mock Provider 验证模型选择、重试和流式响应。
- 运行 `tests/providers/` 中与当前问题对应的定向测试。

**产物与验收**

- 文档：`05-provider-and-config.md`。
- 能从配置项追踪到最终 Provider 实例和模型参数。
- 不在笔记、日志或测试数据中记录真实密钥。

### 阶段 6：Channel、Gateway 与 WebUI

**核心问题**

- Channel 如何把外部消息转换为 Bus 事件？
- ChannelManager 如何发现、启动和停止渠道？
- Gateway 启动了哪些长期服务？
- WebUI 如何通过 WebSocket 与后端通信？

**重点源码**

- `nanobot/channels/base.py`
- `nanobot/channels/manager.py`
- `nanobot/channels/websocket/`
- `nanobot/gateway/service.py`
- `nanobot/gateway/runtime.py`
- `nanobot/webui/`

**最小实验**

- 使用 Mock Channel 验证消息收发和路由。
- 启动最小 Gateway，验证生命周期和健康检查。
- 运行 `tests/gateway/` 和相关 WebUI smoke 测试。

**产物与验收**

- 文档：`06-channel-gateway-webui.md`。
- 能对比 CLI 单次、CLI 交互、Gateway 和 WebUI 四种入口。
- 能指出消息协议边界和运行时生命周期的归属。

### 阶段 7：安全边界与扩展机制

**核心问题**

- Workspace 文件访问如何限制在允许范围内？
- Shell、Web Fetch 和网络请求有哪些安全检查？
- MCP、Skill、Tool 和 Subagent 分别如何扩展 Agent？
- 新增能力时需要保持哪些安全与兼容性约束？

**重点源码**

- `nanobot/security/`
- `nanobot/agent/tools/filesystem.py`
- `nanobot/agent/tools/shell.py`
- `nanobot/agent/tools/web.py`
- `nanobot/agent/tools/mcp.py`
- `nanobot/skills/`

**最小实验**

- 在临时 workspace 验证允许路径和越界路径。
- 观察危险命令、SSRF 地址和非法工具参数的拒绝结果。
- 运行 `tests/security/` 和相关 Tools 安全测试。

**产物与验收**

- 文档：`07-security-and-extension.md`。
- 能列出主要信任边界、威胁入口和对应防护。
- 能为一个扩展需求选择合适的 Tool、Skill、MCP 或 Subagent 机制。

### 阶段 8：架构、设计模式与设计哲学

**目标**

基于阶段 1～7 的源码、测试和运行证据，归纳 nanobot 的整体架构、设计模式、核心取舍与工程哲学。

**核心问题**

- nanobot 如何划分渠道、消息总线、Agent 核心、Provider、Tool 和持久化边界？
- 哪些设计模式真实存在于源码中，它们解决了什么问题？
- 哪些结论是源码事实，哪些是基于实现的设计推断？
- nanobot 如何在轻量化、可扩展性、安全性和可观测性之间取舍？

**重点分析**

- MessageBus：事件驱动、生产者/消费者和解耦。
- AgentLoop：状态机、会话隔离和依赖注入。
- AgentRunner：模型—工具迭代循环、流式处理和运行时治理。
- Provider：策略、适配器、注册表和工厂组合。
- Tool、Channel、Skill、MCP：发现机制、插件化和扩展边界。
- Session、Memory 和 Workspace：持久化、恢复、压缩和长期记忆。
- Security：工作区、Shell、网络和渠道访问控制边界。

**分析方法**

1. 先引用源码位置和测试证据，再给出模式名称。
2. 对每个模式记录适用场景、收益、代价和潜在限制。
3. 明确区分“实现事实”和“个人推断”。
4. 不使用外部文章替代对 nanobot 源码的理解。

**产物与验收**

- 文档：`08-architecture-and-design-philosophy.md`。
- 形成一张 nanobot 总体架构图和一张模块职责表。
- 形成“源码证据 → 设计模式 → 收益/代价”的分析表。
- 能解释至少五个核心设计取舍，并指出对应源码和测试。

### 阶段 9：综合复盘与小型实践

**目标**

把前面分散的知识重新组织为一个可验证的端到端 Agent 心智模型。

**综合任务**

1. 从一个用户问题开始，完整追踪消息、上下文、模型、工具和持久化过程。
2. 对比“无工具回答”和“包含一次工具调用”的两条执行路径。
3. 选择一个低风险扩展点，完成设计、实现、测试和复盘。
4. 总结 nanobot 的核心设计取舍、适用场景和当前限制。

**产物与验收**

- 文档：`09-capstone-review.md`。
- 形成一张端到端调用图和一份源码导航表。
- 小型实践具备设计说明、完整代码和对应测试。
- 能脱离笔记口述 nanobot Agent 的主要运行机制。

## 5. 阶段完成标准

一个阶段只有同时满足以下条件才视为完成：

- 目标问题均有明确答案，并标注对应源码位置。
- 至少完成一个可重复的最小实验或定向测试。
- 关键流程已用时序图、流程图或结构图表达。
- 文档中的命令在当前环境中实际执行过。
- 已记录未解决问题，并明确是否影响进入下一阶段。
- 已检查文档链接、命令准确性和 Git 变更范围。

## 6. 版本与变更策略

- `main` 仅用于跟踪上游项目，不直接存放学习改动。
- 所有学习文档和实验在 `nanobot-learn` 分支完成。
- 每个阶段独立提交，便于回顾和比较学习进度。
- 上游更新时，先记录新旧基线，再同步 `main` 并合入学习分支。
- 不在学习阶段顺手重构 nanobot 源码。
- 涉及源码或重要配置修改时，先编写设计说明并确认风险。

建议的提交粒度示例：

```text
docs: 制定 nanobot Agent 体系学习计划
docs: 完成 CLI 与消息总线学习笔记
test: 添加 AgentRunner 工具循环学习实验
```

## 7. 当前推进项

阶段 0 已完成，进入阶段 1：

1. 已完成：校正 `overview-guide.md` 的目录说明和相对链接。
2. 已完成：记录 `nanobot-learn-dev` 环境的 Python、Pydantic、pytest 和 ruff 版本。
3. 已完成：CLI 帮助、`pip check` 和 Bus 定向测试均通过。
4. 当前学习主题：CLI 入口与 MessageBus。

## 8. 关联资料

- [已有项目概览](./overview-guide.md)
- [官方核心概念](../docs/concepts.md)
- [官方架构说明](../docs/architecture.md)
- [官方开发说明](../docs/development.md)
- [项目贡献指南](../CONTRIBUTING.md)


## 9. 阶段 0 验证记录

验证日期：2026-07-21。

| 检查项 | 结果 |
|---|---|
| 验证环境 | `nanobot-learn-dev` |
| 当前分支 | `nanobot-learn` |
| 代码基线 | `93571149` |
| Python | `3.12.13` |
| Pydantic | `2.13.4` |
| pytest | `9.1.1` |
| ruff | `0.15.22` |
| pip check | 通过 |
| nanobot CLI | 通过，退出码 0 |
| Bus 定向测试 | 9 passed |

阶段 0 验收已完成，后续进入阶段 1。