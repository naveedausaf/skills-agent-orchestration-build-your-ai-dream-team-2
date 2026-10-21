# Agent team

Mona's Project Pulse dashboard will be built by a custom four-agent team, orchestrated through GitHub Copilot CLI in a Codespace.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinates the team: obtains a plan, assigns non-overlapping file scopes, sequences dependent work, verifies the integrated result, and reports the outcome. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository and relevant documentation, identifies risks and dependencies, and produces an implementation plan with file ownership, sequencing, edge cases, and validation expectations. | `.github/agents/planner.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Defines and implements the dashboard's UI/UX, accessibility, information hierarchy, responsive behavior, and polished Project Pulse visual treatment. | `.github/agents/designer.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements the assigned application logic and support configuration with explicit errors, deterministic behavior, and validation; for Project Pulse, it can also add the assigned VS Code launch configuration. | `.github/agents/coder.agent.md` |

The Orchestrator first asks the Planner for a delivery strategy, then delegates scoped design and implementation tasks to the Designer and Coder in parallel only when their files and dependencies allow it. All agents leave staging, commits, and pushes under the learner's control through Copilot CLI prompts.
