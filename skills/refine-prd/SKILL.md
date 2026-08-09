---
name: refine-prd
description: >
  Validate and improve an existing PRD (.taskmaster/docs/prd.md) by finding contradictions, gaps,
  vague language, and missing details — then fixing them autonomously. Only asks the user about
  genuine decision forks where context doesn't clearly favor one answer. Designed for iterative use:
  invoke repeatedly across sessions until the PRD is implementation-ready with zero open questions.
  Use this skill after claude-prd:idea-to-prd (or any other PRD generator) has produced a PRD and
  you want to harden it before implementation. Also use when the user says "review PRD", "improve PRD", "validate PRD",
  "refine PRD", "PRD quality", "PRD ready?", or anything about making an existing PRD better.
user_invocable: true
---

# Refine PRD

Autonomous PRD validator and improver. Reads the PRD, understands the project context, finds every
issue it can, fixes what's fixable, and only bothers the user when there's a genuine decision that
the context can't resolve.

**Goal:** Each invocation brings the PRD measurably closer to being implementation-ready. After
enough iterations (often just 1-2), the PRD should have zero contradictions, zero vague requirements,
and zero open questions that block implementation.

## When to ask the user vs. decide yourself

This is the most important part of this skill. Getting this wrong (asking too many questions, or
asking obvious ones) makes the skill annoying and useless.

**Decide yourself (DO NOT ask) when:**
- The answer is obvious from the project context (codebase, README, tech stack, configs)
- The answer is obvious from the PRD itself (one section implies the answer)
- The answer follows from common sense or industry standards (e.g., "password hashing" → bcrypt/argon2)
- One option is clearly more appropriate given the overall project direction
- The question is about implementation details (PRD says WHAT, not HOW)
- The question is cosmetic or stylistic (wording, formatting, section order)
- You're asking "have you considered X?" — instead, just add X if it's important, or skip it

**Ask the user ONLY when ALL of these are true:**
1. There's a genuine fork — at least 2 meaningfully different paths
2. The choice materially affects scope, architecture, or user experience
3. The context doesn't clearly favor one answer over the other
4. You can't resolve it by making the decision explicit in the PRD (sometimes just documenting
   "we chose X because Y" is enough, and the user can change it next iteration if they disagree)

**When you do ask:**
- Use AskUserQuestion tool
- Present 2-4 options as a numbered list
- Put the most likely answer first
- Briefly explain what each option implies for the project
- Maximum 3 questions per session — if you have more, pick the 3 most impactful and resolve
  the rest yourself by choosing the most reasonable option and documenting the rationale

**When in doubt: decide yourself, document your reasoning in the PRD, and move on.** The user
can always override in the next iteration. This is dramatically better than asking 15 questions
that waste the user's time and attention.

## Process

### Step 1: Read PRD and project context

Read `.taskmaster/docs/prd.md`. If it doesn't exist, stop and tell the user.

Then gather context to inform your analysis:
- Existing codebase structure (key directories, files)
- README, CLAUDE.md, package.json / pom.xml / build.gradle / pyproject.toml
- Recent git history (what's been built, what's the trajectory)
- Any existing implementation related to the PRD
- Tech stack and conventions already in use

This context is essential — it lets you answer questions that would otherwise require user input.

### Step 2: Systematic analysis

Analyze the PRD across these dimensions. For each issue found, classify it:

**Category A — Auto-fix (you fix it, no questions asked):**

| Issue type | What to do |
|---|---|
| Contradictions between sections | Resolve based on context, note in changelog |
| Vague language ("fast", "scalable", "secure") | Replace with specific targets based on project type and industry norms |
| Missing required PRD sections | Add section with reasonable content from context |
| Inconsistent terminology | Pick one term, use consistently, note the choice |
| Missing acceptance criteria | Add criteria based on requirement description |
| Missing priorities (Must/Should/Could) | Assign based on dependency analysis and project goals |
| Requirements without REQ-NNN numbering | Add numbering |
| Broken cross-references | Fix references |
| Duplicate or overlapping requirements | Merge, keep the more detailed one |
| Missing dependency chains | Add dependencies based on logical analysis |
| Unrealistic NFR targets | Adjust to industry standards, note the change |
| "Open Questions" that you can answer from context | Answer them and move to resolved |
| Incomplete user stories | Fill in missing acceptance criteria |
| Tasks that describe HOW instead of WHAT | Rewrite to describe outcomes |
| Missing "Out of Scope" items that are clearly out | Add them |
| Phase ordering issues | Reorder based on dependencies |

**Category B — Genuine decision forks (ask only if criteria above are met):**

| Issue type | Example |
|---|---|
| Scope ambiguity with real trade-offs | "Support mobile? Doubles scope but 60% of users are mobile" |
| Architecture fork with no clear winner | "SSR vs SPA — depends on SEO importance vs interactivity needs" |
| Business priority conflict | "Features A and B both Must Have but can't fit in Phase 1 together" |
| Target audience ambiguity | "B2B vs B2C — completely different auth and pricing models" |

### Step 3: Apply fixes and ask questions

1. **First**, apply all Category A fixes to the PRD in memory
2. **Then**, if there are Category B questions (max 3), ask them one at a time
3. Apply user's answers to the PRD
4. If user's answer reveals new Category A issues, fix those too

### Step 4: Write the changelog

Add or update a `## Changelog` section at the end of the PRD:

```markdown
## Changelog

### YYYY-MM-DD — Refinement iteration N
**Auto-resolved:**
- Fixed contradiction: section 3 said "3 user roles" but section 4 only defined 2 → added "Moderator" role
- Replaced vague "fast response time" with "API response < 200ms p95"
- Added missing acceptance criteria to REQ-004, REQ-007
- Resolved open question about auth provider → using existing OAuth2 setup (found in codebase)

**User decisions:**
- Scope: chose to include mobile web support (option 1) → affects REQ-012 through REQ-015
- Priority: moved REQ-008 from Must Have to Should Have for Phase 1

**Remaining:**
- No open questions remain / N open questions remain (listed in section 10)
```

This changelog serves two purposes:
- Transparency — the user sees exactly what was changed and why
- Convergence tracking — each iteration should have fewer items

### Step 5: Write updated PRD

Save the updated PRD to `.taskmaster/docs/prd.md`.
Commit with message: `refine: PRD iteration N — <brief summary>`.

### Step 6: Report

Present a concise summary:
- How many issues found and fixed
- What questions were asked (if any)
- How many open questions remain
- Overall PRD readiness assessment (Ready / Almost ready / Needs more work)

If the PRD is ready: say so clearly. Don't invent work.
If it needs more work: explain what's still missing and suggest running `/claude-prd:refine-prd` again.

## Analysis checklist

Use this as a systematic checklist — go through every item:

### Structure & Completeness
- [ ] All 10 required sections present
- [ ] Executive summary is 2-3 sentences, not a paragraph
- [ ] Problem statement covers current situation, user impact, business impact, why now
- [ ] Every goal has a SMART metric with baseline → target

### Requirements Quality
- [ ] Every requirement has REQ-NNN numbering
- [ ] Every requirement has a priority (Must/Should/Could)
- [ ] No vague words: "fast", "easy", "good", "scalable", "secure", "robust", "flexible", "intuitive", "modern", "efficient"
  — each replaced with a specific, measurable target
- [ ] Every requirement has acceptance criteria (testable, not just "works correctly")
- [ ] No implementation details in requirements (describe WHAT, not HOW)
- [ ] No duplicate or overlapping requirements

### User Stories
- [ ] Each story follows "As a [role], I want [action], so that [benefit]"
- [ ] Each story has ≥3 acceptance criteria
- [ ] All user roles from the system are represented
- [ ] Edge cases and error scenarios covered

### Non-Functional Requirements
- [ ] Performance targets: specific numbers (ms, req/s, MB)
- [ ] Security: specific measures, not "must be secure"
- [ ] Scalability: specific capacity targets
- [ ] Reliability: uptime %, RTO, RPO
- [ ] All NFRs are realistic for the project type

### Internal Consistency
- [ ] User story roles match roles defined elsewhere in PRD
- [ ] Requirements referenced in roadmap tasks actually exist
- [ ] Dependencies form a valid DAG (no circular dependencies)
- [ ] Phase ordering respects dependency chains
- [ ] Task complexity estimates are consistent (similar tasks = similar estimates)
- [ ] "Out of Scope" items don't contradict requirements
- [ ] Success metrics align with stated goals

### Implementation Roadmap
- [ ] Every requirement is covered by at least one task
- [ ] Tasks describe WHAT, not HOW
- [ ] Each task is one focused session worth of work
- [ ] Dependencies are explicit and correct
- [ ] Phases have clear goals

### Risks & Open Questions
- [ ] Key risks identified with mitigations
- [ ] Open questions have owners and deadlines
- [ ] Questions that can be answered from context ARE answered (not left open)

## Key principles

- **Autonomous first.** Every question you DON'T ask is a win. The user's time is precious.
- **Convergent.** Each iteration strictly reduces the number of issues. Never introduce new
  ambiguity or vague language.
- **Transparent.** The changelog shows exactly what changed and why, so the user can disagree
  and override in the next iteration.
- **Context-aware.** Read the codebase. Read the configs. Read the git history. The answers to
  most questions are already in the project.
- **Honest.** If the PRD is ready, say so. Don't make up problems to seem thorough. Don't ask
  questions to seem engaged. Be direct.
