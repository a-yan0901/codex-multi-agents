# Codex GPT-5.6 Multi-Agent Pack

This pack defines seven standalone Codex custom agents:

- architect    -> gpt-5.6-terra / high
- worker       -> gpt-5.6-luna / medium
- tester       -> gpt-5.6-luna / medium
- debugger     -> gpt-5.6-luna / high
- explorer     -> gpt-5.6-luna / low
- reviewer     -> gpt-5.6-terra / high

## Install on Windows PowerShell

```powershell
New-Item -ItemType Directory -Force "$HOME\.codex\agents" | Out-Null
Copy-Item ".\agents\*.toml" "$HOME\.codex\agents\" -Force
```

## Install on macOS / Linux

```bash
mkdir -p ~/.codex/agents
cp ./agents/*.toml ~/.codex/agents/
```

## Optional global multi-agent defaults

Add this to `~/.codex/config.toml` if desired:

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 6
default_subagent_model = "gpt-5.6-luna"
default_subagent_reasoning_effort = "medium"
```

The standalone agent files explicitly pin their own models/effort, so those values take precedence for these named agents.

## Suggested parent-agent instruction

You can tell the main Codex session:

> Coordinate this task as the orchestrator. Keep architecture and final decisions in the main thread. Delegate codebase discovery to explorer, design questions to architect, implementation to worker, validation to tester, difficult failures to debugger, and final review to reviewer. Avoid duplicating delegated work and prefer Luna workers for routine execution.

## Notes

- Custom agent files require `name`, `description`, and `developer_instructions`.
- The agent `model` and `model_reasoning_effort` override inherited defaults.
- Read-heavy roles are set to `read-only`; implementation/testing/debugging roles use `workspace-write`.
- Parent runtime permission/approval overrides may still apply to spawned agents.

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
