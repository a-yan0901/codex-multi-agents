# Codex GPT-5.6 Multi-Agent Pack

## Why this pack

Running everything in a single Codex session means every task pays the
cost of the full system prompt, the same long list of tools, and
general-purpose instructions. For most real work that's overkill.

This pack splits the work into six focused sub-agents — explorer,
architect, worker, tester, debugger, and reviewer — each with a short,
role-specific system prompt and a model / reasoning effort tuned to
its job. The main Codex session reads the request, decides which kind
of work it actually is (a quick search, a design question, a code
change, a failing test, a complex bug, a risky edit), and dispatches
it to the matching sub-agent.

Two things fall out of that:

- **Lower token usage.** Sub-agent prompts are tight and only mention
  the tools and policies they need. The orchestrator doesn't pay for
  the full general-purpose prompt on every turn.
- **Cleaner separation of concerns.** Exploration, design, implementation,
  verification, and review each run in their own context, so findings
  don't leak into implementation or vice versa.

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

## Sub-agent Dispatch

Add the following to `AGENTS.md`:

```markdown
## Sub-agent Dispatch

The main agent is responsible for understanding requirements, breaking
down tasks, making critical decisions, and performing final acceptance.

When a suitable sub-agent is available, prefer delegating work that
can be completed independently:

- `explorer`: search code, locate implementations, trace call chains,
  and gather context.
- `architect`: complex architecture, interface design, cross-module
  plans, and high-risk technical decisions.
- `worker`: code implementation and local edits once a plan is clear.
- `tester`: write tests, run tests, and perform regression checks.
- `debugger`: diagnose complex failures, analyze root causes, and
  verify fixes.
- `reviewer`: independent review of larger or high-risk changes.

Principles:

1. Don't invoke sub-agents for simple tasks just for the sake of using them.
2. Prefer delegating execution work (search, implementation, testing)
   to the matching execution-oriented sub-agent.
3. Prefer delegating complex architecture and high-risk judgement to
   `architect` or `reviewer`.
4. The main agent should not redo work that a sub-agent has already
   completed with trustworthy results.
5. Sub-agents should return conclusions and necessary evidence, not
   large volumes of unrelated context.
6. Independent sub-tasks may be executed in parallel.
7. Final changes are still accepted by the main agent against the
   user's requirements and verification results.
```
