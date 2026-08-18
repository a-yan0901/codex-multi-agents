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
