# Codex GPT-5.6 多 Agent 包

本包提供六个独立的 Codex 自定义 Agent：

- architect    -> gpt-5.6-terra / high
- worker       -> gpt-5.6-luna / medium
- tester       -> gpt-5.6-luna / medium
- debugger     -> gpt-5.6-luna / high
- explorer     -> gpt-5.6-luna / low
- reviewer     -> gpt-5.6-terra / high

## Windows PowerShell 安装

```powershell
New-Item -ItemType Directory -Force "$HOME\.codex\agents" | Out-Null
Copy-Item ".\agents\*.toml" "$HOME\.codex\agents\" -Force
```

## macOS / Linux 安装

```bash
mkdir -p ~/.codex/agents
cp ./agents/*.toml ~/.codex/agents/
```

## 可选的全局多 Agent 默认配置

如果需要启用，可以把以下内容追加到 `~/.codex/config.toml`：

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 6
default_subagent_model = "gpt-5.6-luna"
default_subagent_reasoning_effort = "medium"
```

独立的 Agent 文件中已经显式指定了 `model` 和 `model_reasoning_effort`，所以这些命名 Agent 会优先使用自身配置。

## 建议的主 Agent 提示词

可以让主 Codex 会话这样理解任务：

> 由当前会话担任 orchestrator，负责协调本任务。架构和最终决策保留在主线程；代码库探索委派给 explorer，方案设计委派给 architect，实现委派给 worker，验证委派给 tester，复杂故障委派给 debugger，最终 review 委派给 reviewer。避免重复委派，并优先使用 Luna worker 处理常规执行任务。

## 子 Agent 调度

在 `AGENTS.md` 中新增如下内容：

```markdown
## 子 Agent 调度

主 Agent 负责需求理解、任务拆解、关键决策和最终验收。

存在合适的子 Agent 时，优先委派可独立完成的工作：

- `explorer`：搜索代码、定位实现、调查调用链和收集上下文。
- `architect`：复杂架构、接口设计、跨模块方案和高风险技术决策。
- `worker`：明确方案后的代码实现和局部修改。
- `tester`：测试编写、测试执行和回归验证。
- `debugger`：定位复杂故障、分析失败原因和验证修复。
- `reviewer`：对较大或高风险修改进行独立 review。

原则：

1. 简单任务不要为了使用子 Agent 而使用子 Agent。
2. 搜索、实现、测试等执行型任务优先委派给对应的执行型子 Agent。
3. 复杂架构和高风险判断优先委派给 `architect` 或 `reviewer`。
4. 主 Agent 不重复执行已经由子 Agent 完成且结果可信的工作。
5. 子 Agent 返回结论和必要证据，不要返回大量无关上下文。
6. 可以并行执行互不依赖的子任务。
7. 最终修改仍由主 Agent 对照用户需求和验证结果进行验收。
```

## 注意事项

- 自定义 Agent 文件必须包含 `name`、`description` 和 `developer_instructions`。
- Agent 的 `model` 和 `model_reasoning_effort` 会覆盖继承的默认配置。
- 偏读取的角色设置为 `read-only`，实现/测试/调试类角色使用 `workspace-write`。
- 父会话的运行时权限/审批设置仍可能作用于派生的 Agent。

## English Version

See [README.md](README.md) for the English version.
