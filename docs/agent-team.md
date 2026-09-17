# Project Pulse agent team

Mona's Project Pulse dashboard will be built in a Codespace with GitHub Copilot CLI coordinating the custom agents defined in `.github/agents/`.

| Agent | Model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 | Breaks the work into phases, assigns non-overlapping file scopes, coordinates dependencies, verifies the integrated dashboard, and reports the result. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 | Researches the repository and dashboard requirements, then produces an implementation plan with file assignments, dependencies, edge cases, parallel work, and validation expectations. | `.github/agents/planner.agent.md` |
| Designer | Gemini 3.1 Pro | Defines the accessible, responsive dashboard experience: information hierarchy, project cards, status and priority treatments, visual clarity, and CSS guidance. | `.github/agents/designer.agent.md` |
| Coder | GPT-5.5 | Implements the assigned dashboard files with clear, testable behavior; when assigned runnable app work, it also prepares the Project Pulse launch configuration. | `.github/agents/coder.agent.md` |

## How the team will build Project Pulse

The Orchestrator first asks the Planner for a practical plan and converts it into scoped phases. It then assigns design direction to the Designer and implementation to the Coder, running work in parallel only when the file scopes and dependencies allow it. The Orchestrator integrates and verifies the final result: a polished Project Pulse dashboard that is straightforward for Mona's contributors to open and use. Each specialist stays within its assigned files, and no agent stages, commits, or pushes changes; the learner controls Git operations through Copilot CLI prompts.
