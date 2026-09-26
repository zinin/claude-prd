---
name: refine-tasks
description: >
  Validate and improve tasks.json against the PRD to ensure tasks are a complete, accurate, and
  executable projection of the PRD. Finds missing coverage, contradictions with PRD, vague descriptions,
  broken dependencies, scope issues, and fixes them autonomously. Designed for iterative use: invoke
  repeatedly until tasks.json fully covers the PRD and every task is ready for autonomous agent execution.
  Use after tasks have been generated from a PRD (via TaskMaster's parse-prd, or manually). Also use
  when the user says "review tasks", "improve tasks", "validate tasks", "refine tasks", "tasks ready?",
  "check tasks against PRD", "tasks cover PRD?", or anything about making tasks.json better or ensuring
  PRD coverage. This includes requests to check task quality, fix task dependencies, or verify tasks
  are implementation-ready.
user_invocable: true
---

# Refine Tasks

Autonomous tasks.json validator and improver. The central premise: **tasks.json is the only artifact
the execution agent will see** — the PRD will not be available at execution time. Therefore tasks.json
must be a complete, self-contained, executable projection of the PRD. Every requirement, every
acceptance criterion, every architectural decision from the PRD must be captured in the tasks so
that an agent executing them blind (without the PRD) produces an implementation that satisfies the PRD.

**Critical context: the executor is an AI agent** (Claude Code, Codex, OpenCode, or similar).
Not a human developer. This fundamentally shapes how tasks must be written:
- **No manual steps.** An AI agent can't "visually inspect the UI", "ask the stakeholder",
  "deploy to staging and check", or "discuss with the team". Every step must be automatable.
- **No ambiguous quality criteria.** "Ensure good UX" means nothing to an agent. Instead:
  "Verify that the response time is under 200ms" or "Assert that the error message includes
  the field name".
- **Verification = code.** `testStrategy` must describe checks an AI agent can run: shell commands,
  test scripts, assertions, build/lint commands. Never "manually verify" or "check in browser".
- **Explicit file paths and commands.** An AI agent works best with concrete instructions:
  which files to create/edit, which commands to run, which patterns to follow from the codebase.
- **No human-only tasks.** Tasks like "get design approval", "coordinate with backend team",
  "review with PM" are not executable by an AI agent and must be reformulated or removed.

**Goal:** Each invocation brings tasks.json measurably closer to being execution-ready. After enough
iterations (often 1-2), every PRD requirement should be covered, every task should be specific enough
for autonomous execution, and the dependency graph should be valid.

## When to ask the user vs. decide yourself

Same principle as PRD refinement — autonomy first, questions only at genuine forks.

**Decide yourself (DO NOT ask) when:**
- The answer is obvious from the PRD (requirement text implies the answer)
- The answer is obvious from the project context (codebase, tech stack, conventions)
- A task is clearly too vague — just make it specific based on the PRD and codebase
- A dependency is clearly missing or wrong — just fix it
- A task is clearly too large — split it and explain in the changelog
- Coverage gap is clear — add the missing task based on the PRD requirement
- The question is about task wording or structure (cosmetic)
- testStrategy is missing or generic — write a concrete one based on the task and project test patterns

**Ask the user ONLY when ALL of these are true:**
1. There's a genuine fork — at least 2 meaningfully different ways to decompose or structure the work
2. The choice materially affects what gets implemented or in what order
3. Neither the PRD nor the codebase clearly favors one answer
4. You can't resolve it by just picking the most reasonable option and documenting it

**When you do ask:**
- Use AskUserQuestion tool
- Maximum 3 questions per session
- Present 2-4 options with the most likely first
- If you have more issues, resolve the rest yourself

## Process

### Step 1: Read tasks.json, PRD, and project context

Read `.taskmaster/tasks/tasks.json`. If it doesn't exist, stop and tell the user.
Read `.taskmaster/docs/prd.md`. If it doesn't exist, stop and tell the user — PRD is essential for
validating task coverage.

Then gather project context:
- Codebase structure (key directories, existing implementation)
- CLAUDE.md, README, package.json / build configs
- Tech stack and conventions in use
- Existing tests and test patterns
- Recent git history

This context helps you write better task details and testStrategies, and resolve ambiguities
without asking the user.

### Step 2: PRD coverage analysis (most critical)

This is the heart of the skill. Map every PRD element to tasks:

**Build a coverage matrix:**
- For each PRD requirement (REQ-NNN): which task(s) cover it? Fully or partially?
- For each PRD user story: which task(s) implement it?
- For each roadmap item: which task(s) correspond to it?
- For each acceptance criterion: is it reflected in some task's details or testStrategy?

**Flag gaps:**
- PRD requirements with no corresponding task → **missing task**
- PRD requirements partially covered (some acceptance criteria missing) → **incomplete task**
- Tasks that don't trace back to any PRD element → **orphan task** (may be valid infrastructure
  work, but verify)
- PRD acceptance criteria not reflected in any task's testStrategy → **untested requirement**

**Flag contradictions:**
- Task description contradicts PRD requirement
- Task details describe a different approach than what PRD specifies
- Task dependencies contradict PRD phase ordering
- Task scope is narrower/broader than the PRD requirement it implements

### Step 3: Task quality analysis

For each task (and subtask if present), check:

**Content quality:**
- `title` — concise, specific, action-oriented? (not "Handle stuff", but "Add JWT auth middleware")
- `description` — explains WHAT and WHY? Sufficient for an AI agent to understand the goal without PRD?
- `details` — contains concrete implementation steps? References specific files, APIs, patterns?
  An AI agent reading only this field should know exactly what to build. Includes exact file paths
  to create/modify, specific function signatures, config keys, API endpoints where relevant.
- `testStrategy` — describes how to **programmatically** verify the task is complete? Must be
  executable by an AI agent: specific test commands (`npm test`, `vitest run ...`), build commands,
  lint checks, curl/API calls with expected responses, grep assertions on output.
  **Red flags in testStrategy:** "manually verify", "visually check", "open in browser",
  "ask QA to test", "check the UI", "deploy and verify". All of these must be rewritten as
  automatable checks or removed.

**AI-agent executability:**
- Does every step in `details` describe something an AI agent can do? (edit files, run commands,
  write code, run tests — yes. Review with humans, check visually, deploy manually — no.)
- Is `testStrategy` fully automatable? An AI agent will run these checks to confirm the task is done.
  If it can't run them, it can't verify completion.
- Are there human-coordination steps? ("sync with backend team", "get PM approval", "discuss
  architecture") — these must be either removed or replaced with concrete technical actions.
- Does the task avoid subjective quality criteria? ("make it look good", "ensure great UX",
  "write clean code") — replace with measurable checks.

**Structural correctness:**
- IDs are sequential integers without gaps
- All dependencies reference existing task IDs
- No circular dependencies
- No self-dependencies
- Dependency order is logical (foundational work before dependent work)
- Priorities are consistent (critical dependencies shouldn't have low priority)
- All statuses are valid

**Scope appropriateness:**
- Is the task completable in a single agent session? If it touches too many files, systems, or
  concepts, it should be split.
- Is the task too trivial? Single-line changes or pure config tweaks can often be merged into
  a related task.
- Does the task mix unrelated concerns? "Add auth AND redesign the dashboard" should be two tasks.

**Subtask checks (if present):**
- Same quality checks as top-level tasks
- Subtask IDs are sequential 1..N within parent
- Subtask dependencies reference only sibling subtask IDs (within the same parent)
- Subtask dependencies only reference earlier IDs (no forward dependencies)
- Subtasks collectively cover the parent task's scope
- Parent task description is consistent with the sum of its subtasks

### Step 4: Apply fixes

Classify each issue as Category A (auto-fix) or Category B (ask user):

**Category A — Auto-fix:**

| Issue type | What to do |
|---|---|
| Missing task for a PRD requirement | Add task with details derived from the PRD requirement and codebase context |
| Incomplete task (missing PRD criteria) | Add missing details, acceptance criteria, testStrategy |
| Vague title/description/details | Rewrite with specifics from PRD and codebase |
| Missing or generic testStrategy | Write concrete strategy based on project test patterns |
| Broken dependency references | Fix to correct IDs |
| Missing obvious dependencies | Add them |
| Circular dependencies | Break the cycle by reordering or splitting tasks |
| Non-sequential IDs | Renumber (update all dependency references) |
| Priority inconsistencies | Align with PRD priorities and dependency structure |
| Task too broad for one session | Split into focused tasks, update dependencies |
| Contradictions with PRD | Align task with PRD (PRD is source of truth) |
| Details describe HOW without WHAT | Add the WHAT from PRD, keep useful HOW |
| Orphan tasks that are clearly needed | Keep, add note about purpose |
| Orphan tasks that duplicate PRD-covered tasks | Remove or merge |
| testStrategy contains manual/visual checks | Rewrite as automatable commands, scripts, or assertions |
| Task contains human-coordination steps | Remove or replace with concrete technical actions |
| Task uses subjective quality criteria | Replace with measurable, programmatically verifiable checks |

**Category B — Ask user (rare):**

| Issue type | Example |
|---|---|
| PRD requirement ambiguous enough that 2+ valid task decompositions exist | "REQ-005 could be one task or three — depends on whether you want incremental delivery" |
| Conflicting PRD requirements that affect task structure | "REQ-003 and REQ-007 imply different auth flows — which takes priority?" |
| Scope decision: include or exclude optional PRD items | "PRD marks REQ-012 as Could Have — include in tasks or skip?" |

### Step 5: Write changelog

Add or update a `changelog` field in tasks.json metadata (or if the project uses a separate
changelog, append there). Use this format in the commit message and report:

```
refine-tasks iteration N:
Auto-resolved:
- Added task 15 for REQ-009 (was missing from task list)
- Split task 3 into tasks 3a, 3b (too broad for one session)
- Fixed circular dependency between tasks 4 and 7
- Rewrote vague details in tasks 2, 5, 8 with specifics from PRD
- Added testStrategy to tasks 1, 4, 6 based on project vitest patterns

User decisions:
- Chose to split REQ-005 into 3 incremental tasks (option 2)

Coverage: 13/13 requirements covered, 6/6 user stories covered
Remaining: 0 gaps, 0 contradictions
```

### Step 6: Write updated tasks.json

Save the updated tasks.json to `.taskmaster/tasks/tasks.json`.
Preserve the exact format (standard or multi-tag). Update metadata fields (taskCount, lastModified).
Commit with message: `refine: tasks iteration N — <brief summary>`.

### Step 7: Report

Present a concise summary:
- **Coverage:** X/Y PRD requirements covered, X/Y user stories covered, X/Y roadmap items covered
- **Issues found and fixed:** count by category
- **Questions asked:** if any
- **Remaining gaps:** if any
- **Readiness:** Ready for execution / Almost ready / Needs more work

If tasks are ready: say so clearly. Don't invent work.
If they need more work: explain what's still missing and suggest running `/prd-flow:refine-tasks` again.

## Analysis checklist

### PRD Coverage (most important)
- [ ] Every REQ-NNN has at least one task that implements it
- [ ] Every user story has tasks covering all its acceptance criteria
- [ ] Every roadmap phase/item has corresponding tasks
- [ ] No PRD acceptance criterion is left without a task or testStrategy that verifies it
- [ ] Out-of-scope items from PRD are NOT covered by tasks (no scope creep)

### Task Completeness
- [ ] Every task has non-empty title, description, details
- [ ] Every task has a testStrategy (not just "test it" but specific scenarios)
- [ ] Details are self-contained — agent can execute without reading PRD
- [ ] Details reference specific files, modules, APIs where relevant
- [ ] Tasks reference relevant codebase patterns and conventions

### Structural Integrity
- [ ] IDs are sequential integers
- [ ] All dependency references are valid
- [ ] No circular or self-dependencies
- [ ] Dependency order matches logical build order
- [ ] Priorities are consistent with dependencies (blockers are high priority)

### Scope & Executability
- [ ] Each task is completable in one agent session
- [ ] No task mixes unrelated concerns
- [ ] No task is trivially small (merge into related task if so)
- [ ] Task ordering allows incremental progress (early tasks don't depend on late ones)

### Subtasks (if present)
- [ ] Subtask IDs sequential 1..N within parent
- [ ] Subtask dependencies reference only siblings
- [ ] No forward dependencies in subtasks
- [ ] Subtasks collectively cover the parent task scope
- [ ] Each subtask has same quality standards as top-level tasks

### AI-Agent Executability
- [ ] Every step in `details` is something an AI agent can do (edit files, run commands, write code)
- [ ] No manual/visual verification steps in any `testStrategy`
- [ ] No human-coordination tasks (approvals, reviews, discussions with people)
- [ ] No subjective quality criteria without measurable alternatives
- [ ] `testStrategy` contains only automatable checks: test commands, build commands, assertions, scripts
- [ ] Task descriptions are precise enough for an AI agent without domain intuition to execute

### Consistency
- [ ] No contradictions between tasks
- [ ] No contradictions between tasks and PRD
- [ ] Terminology matches PRD and codebase
- [ ] Priorities match PRD priorities

## Key principles

- **PRD is source of truth.** Tasks must reflect the PRD, not the other way around.
  If a task contradicts the PRD, the task is wrong.
- **Self-contained tasks.** The agent executing a task will NOT have the PRD. Everything
  needed for execution must be in the task's title + description + details + testStrategy.
- **Autonomous first.** Every question you DON'T ask is a win. Fix what you can from
  PRD and codebase context.
- **Convergent.** Each iteration strictly reduces coverage gaps and quality issues.
- **Transparent.** The changelog shows exactly what changed and why.
- **Honest.** If tasks are ready, say so. Don't invent problems.
- **AI-executable.** Every task is a prompt for an AI agent, not a ticket for a human.
  The agent can: edit files, run commands, write tests, execute builds. It cannot: open a browser
  and click around, ask a colleague, visually inspect UI, deploy to production and check manually.
  Write tasks accordingly.
