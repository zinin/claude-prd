# prd-flow

Agent plugin: the path from a raw idea to an execution-ready task list —
one skill that authors a PRD through dialogue, and two that harden the PRD and
the generated tasks autonomously, asking you at most three questions per run.

## Features

(Skills are namespaced under `prd-flow:` — that is how Claude Code surfaces plugin skills.)

- **`prd-flow:idea-to-prd`** — turns an idea into a PRD through collaborative
  discovery: explores the project, asks questions one at a time, proposes 2-3
  approaches with trade-offs, validates the design section by section, then writes
  `.taskmaster/docs/prd.md` and commits it. Hard-gated: it writes a PRD and nothing
  else — no code, no scaffolding, no implementation plan.
- **`/prd-flow:refine-prd`** — validates an existing PRD and fixes what it finds:
  contradictions between sections, vague language ("fast", "scalable") replaced with
  measurable targets, missing acceptance criteria, priorities, dependency chains,
  broken cross-references. Answers open questions from the codebase where it can, and
  asks you only at genuine decision forks (max 3 per run). Appends a changelog to the
  PRD so every change is reviewable, and reports whether the PRD is implementation-ready.
- **`/prd-flow:refine-tasks`** — validates `.taskmaster/tasks/tasks.json` against the
  PRD. Builds a coverage matrix (every REQ-NNN, user story and roadmap item mapped to
  tasks), flags gaps, orphans and contradictions, then rewrites vague tasks into
  self-contained ones. Its premise: the executing agent will never see the PRD, so
  everything needed must live in the task — and the executor is an AI agent, so
  `testStrategy` must be automatable commands, never "check visually" or "ask QA".

Both `refine-*` skills are designed for repeated invocation: each run strictly reduces
the number of issues, and they tell you plainly when there is nothing left to fix.

## Install

```
/plugin marketplace add zinin/agent-plugins
/plugin install prd-flow@zinin
```

In Codex (`codex plugin add prd-flow@zinin`), run idea-to-prd in the interactive session: it asks
its questions one at a time, and in the smoke `codex exec` 0.157 ended at the first one.

## Dependencies

- **Required:** none for `idea-to-prd` and `refine-prd` — they only need a git repository.
- **Paths are fixed:** the skills read and write the TaskMaster layout —
  `.taskmaster/docs/prd.md` for the PRD and `.taskmaster/tasks/tasks.json` for the tasks.
  A project that keeps its PRD elsewhere needs the file moved (or the skill edited).
- **For `refine-tasks`:** an existing `tasks.json`, normally produced by
  [TaskMaster](https://github.com/eyaltoledano/claude-task-master) (`task-master parse-prd`).
  The skill validates and improves tasks; it does not generate them from scratch.
- **Git:** all three skills commit their output (PRD or tasks.json) to the current branch.

## See also

- [claude-mesh](https://github.com/zinin/claude-mesh) — multi-model code review, alt-Claude execution, session helpers
- [claude-forge](https://github.com/zinin/claude-forge) — build/test/lint delegation and dependency updates
- [claude-atlassian](https://github.com/zinin/claude-atlassian) — Jira/Confluence analysis and bug investigation
