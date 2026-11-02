# Agents

Agent definitions for AI assistants working in BDPL LabTools repositories.
Each `.md` file is a charge document: a specialist role written as a
system prompt, with a header that restricts the tools it may use. They
are instructed, not trained; the expertise is the text, versioned in git
and changed only by pull request like everything else. The pattern comes
from BDPL LabKit's `.claude/agents/`.

The standing rule: **an agent is never the decision-maker for a gated
action.** Agents read, verify, draft and report; merging, disposition and
anything irreversible belong to a person.

| Agent | Status | Role | When it runs |
|---|---|---|---|
| `docs-steward` | active | Keeps every live document current with the code: reads a PR's diff, fixes stale docs on the same branch, keeps `docs/README.md` and the catalog honest, reports in the PR body | on every pull request, before it is offered for merge (working rule 6) |

## Using an agent

In a Claude Code session in this repository, ask by task ("run the docs
steward on this branch") or by name. The assistant dispatches the
definition as a subagent with the restricted tool set. The agent's
report is its complete deliverable.

Application repositories copy this folder and add their own agents as
they need them (LabKit, for example, carries a bag auditor and a GitHub
warden).
