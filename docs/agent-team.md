# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate the work of building Mona's Project Pulse dashboard with a team of four custom agents.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks down requests, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repo and docs, then returns an implementation plan with steps, file assignments, dependencies, edge cases and validation. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code, fixes bugs, and creates support files such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opens `index.html`) within its assigned scope. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility and visual design: polished dashboard with project cards, status badges, priority treatment, responsive layout, and `.dashboard` / `.project-card` hooks. | `.github/agents/designer.agent.md` |

## Workflow

1. Orchestrator asks Planner for a plan.
1. Orchestrator splits the plan into phases with file scopes.
1. Coder and Designer work in parallel on non-overlapping files, otherwise sequentially.
1. Orchestrator verifies the result and reports back.

The agents never stage, commit, or push; I control git through Copilot CLI prompts.
