# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate the custom agent team building Mona's Project Pulse dashboard:

- **Orchestrator** — Uses **Claude Opus 5.5 (copilot)** to coordinate the specialist agents, assign non-overlapping file scopes, manage dependencies and phases, and verify the integrated result. Definition: `.github/agents/orchestrator.agent.md`.
- **Planner** — Uses **Claude Opus 5.5 (copilot)** to research the repository and relevant documentation, identify requirements, risks, edge cases, dependencies, and validation needs, and produce an actionable implementation plan. Definition: `.github/agents/planner.agent.md`.
- **Coder** — Uses **GPT-5.5 (copilot)** to implement code, fix bugs, create required runnable-app support files, and validate deterministic, testable behavior within the assigned scope. Definition: `.github/agents/coder.agent.md`.
- **Designer** — Uses **Gemini 3.7 Flash (copilot)** to shape the dashboard's UI/UX, accessibility, information hierarchy, interaction flow, responsive behavior, and visual clarity, including Project Pulse-specific dashboard styling. Definition: `.github/agents/designer.agent.md`.
