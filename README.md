# Codex Mastery

**A deeply researched, practical field guide to OpenAI Codex** — across CLI, IDE, app, cloud, GitHub, skills, MCP, subagents, memories, sandboxing, and production workflows.

> 🎨 **Styled version with collapsible cards, search, and dark theme:** <https://felixkruger.github.io/Codex-Mastery/>
>
> The version below is the same content, formatted for GitHub.

`April 2026` · `OpenAI Codex` · `CLI · IDE · App · Cloud` · `Plan → edit → run → review`

> **Default stance:** trust slowly, verify aggressively.

---

## Table of contents

1. [Mental model](#1-mental-model)
2. [Models & reasoning](#2-models--reasoning)
3. [AGENTS.md](#3-agentsmd)
4. [Configuration](#4-configuration)
5. [Security, sandboxing & approvals](#5-security-sandboxing--approvals)
6. [Context, memory & compaction](#6-context-memory--compaction)
7. [Workflows](#7-workflows)
8. [Codex app](#8-codex-app)
9. [Cloud / Web Codex](#9-cloud--web-codex)
10. [Skills & plugins](#10-skills--plugins)
11. [MCP & external tools](#11-mcp--external-tools)
12. [Subagents](#12-subagents)
13. [Integrations & automation](#13-integrations--automation)
14. [Patterns that compound](#14-patterns-that-compound)
15. [Troubleshooting](#15-troubleshooting)
16. [Sources & research base](#16-sources--research-base)

---

## 1. Mental model

> Codex is not only "a model that writes code." It is an agent runtime around a model: instructions, repo context, tool calls, file edits, commands, approvals, and a verification loop.

### 01 · The useful definition
*Think "model + harness + permissions + context + feedback," not chat.*

> ✅ **Core idea:** Codex becomes good when the environment is set up well. Bad outputs often trace back to weak instructions, missing tests, wrong working directory, too much irrelevant context, or permissions that block the verification loop.

**Operating loop:**

1. **Prompt** — You give the goal, constraints, files, and done criteria.
2. **Context** — Codex layers system instructions, AGENTS.md, config, memories, skills, MCP resources, and repo files.
3. **Plan** — The model decides what to inspect, edit, run, or ask permission for.
4. **Act** — It reads files, applies patches, runs commands, uses browser/tools if enabled, and tracks diffs.
5. **Verify** — Tests, linters, typechecks, reviews, PRs, or follow-up prompts close the loop.

| Layer | What it contributes | Typical failure mode | Mastery move |
|---|---|---|---|
| Model | Reasoning, code synthesis, debugging, review judgment. | Overkill model for simple tasks, weak model for ambiguous architecture work. | Pick by task shape: small model for bounded/simple; frontier/high reasoning for long-horizon or risky changes. |
| Client/runtime | CLI, IDE, app, or cloud execution surface with file/command/browser capabilities. | Using cloud for local-only context or CLI for long parallel async work. | Match surface to workflow: local pairing, IDE focus, app orchestration, cloud delegation. |
| Instructions | Global and repo rules via AGENTS.md, config, skills, prompts. | Rules buried in chat history or scattered across docs Codex will not reliably read. | Put stable, required behavior in AGENTS.md; put repeatable procedures in skills. |
| Permissions | Sandbox and approval policy determine what can be done without asking. | Either too strict to run tests or too permissive for untrusted code. | Start narrow; allow workspace edits and local tests; escalate deliberately. |
| Verification | Tests, lint, typecheck, code review, diff review, PR review. | Accepting plausible code without running the project's real checks. | Make "done when checks pass and diff is reviewed" part of every serious prompt. |

### 02 · Where Codex runs
*Same agent family, different ergonomics and trust boundaries.*

| Surface | Best use | Strength | Watch out |
|---|---|---|---|
| CLI | Terminal pairing in a repo. | Fast local loop, scriptable, transparent commands. | Context must be pointed at files/paths unless discovered. |
| IDE extension | Focused code changes while editing. | Open files, selections, and diagnostics can be natural context. | Can over-focus on visible files; mention hidden dependencies. |
| Codex app | Multiple threads, worktrees, automations, Git review, browser/computer use. | Best command center for ongoing projects. | Background automations need sandbox discipline. |
| Cloud / Web | Async delegated tasks in isolated containers. | Runs in parallel and can produce a diff/PR while you do other work. | Needs environment setup and cannot infer local uncommitted state. |
| GitHub / Slack / SDK | Team workflows and automation. | Meet developers where work arrives: issues, PRs, CI, chat, pipelines. | Requires precise scoping and review gates. |

### 03 · The mastery stack
*What to set up before blaming the model.*

1. **Repository instructions:** one clean root `AGENTS.md`, nested files only where conventions differ.
2. **Checks:** document exact install/build/test/lint/typecheck commands, including known failures.
3. **Config:** choose model, reasoning effort, sandbox, approvals, MCP servers, memory settings.
4. **Skills:** encode repeatable workflows such as "ship a Next.js page," "triage flaky test," or "prepare release notes."
5. **Tools:** add MCP only when it supplies high-signal context or deterministic actions.
6. **Review:** make Codex review its own diff and then use a separate review pass for meaningful changes.

> ⚠️ **Hard truth:** "Make it better" is a weak prompt. "Refactor `src/payments` to remove duplicate retry logic, preserve public API, add tests for timeout and idempotency, run `pnpm test payments`, then summarize diff risks" is a usable work order.

---

## 2. Models & reasoning

> The model is a dial, not a status symbol. Match depth to uncertainty, risk, and context length.

### 04 · Model selection map
*Use frontier models where errors are expensive; use smaller models for cheap exploration and subagents.*

| Model | Use it for | Avoid it for | Notes |
|---|---|---|---|
| `gpt-5.5` | Complex coding, architecture, debugging, computer use, research-heavy workflows. | Trivial edits where speed matters more than judgment. | OpenAI docs recommend starting here when it is available in Codex. |
| `gpt-5.4` | Default fallback when 5.5 is not available; long-horizon application work. | Ultra-fast low-cost scans. | Still a strong general coding model. |
| `gpt-5.4-mini` | Fast exploration, large-file review, documentation, lower-reasoning subagents. | High-risk migrations or ambiguous product/architecture calls. | Useful when you want breadth before depth. |
| `gpt-5.3-codex` | Older Codex-optimized workflows where configured or preserved. | New frontier tasks when newer models are available. | Relevant historically and in compatibility contexts. |
| `spark` | Very fast text-only exploration on supported plans. | Tasks requiring maximum reasoning, images, or computer use. | Research-preview style workflow; treat as speed tool, not final authority. |

> 💡 **Rule of thumb:** expensive ambiguity deserves expensive reasoning. Cheap certainty deserves cheap execution.

### 05 · Reasoning effort
*Reasoning depth should change with the phase of work.*

| Effort | Use | Prompt style |
|---|---|---|
| `minimal` | Simple commands, formatting, tiny edits. | Direct imperative with exact target file. |
| `low` | Known fix, small refactor, documentation pass. | Give goal and constraints; ask for concise diff summary. |
| `medium` | Default for normal implementation. | Goal + context + constraints + done criteria. |
| `high` | Debugging, migrations, security-sensitive changes, unfamiliar code. | Ask for plan first, identify assumptions, run checks. |
| `xhigh` | Deep architecture, root-cause analysis, multi-system failures. | Split into investigation → plan → implementation → review. |

```bash
# CLI one-off examples
codex -m gpt-5.5 -c model_reasoning_effort=high "Find the root cause of the flaky checkout test. Do not patch until you explain the cause."

codex -m gpt-5.4-mini -c model_reasoning_effort=low "Summarize the API surface of src/billing in 20 bullets."
```

### 06 · Model switching pattern
*Use smaller models for map-making; switch up for judgment and patching.*

1. **Explore cheap:** ask a fast model to inventory files, modules, risks, and unknowns.
2. **Plan deep:** use a frontier model to decide strategy and failure modes.
3. **Patch focused:** keep the implementation scope narrow and testable.
4. **Review separate:** run `/review` or a fresh reviewer thread after the patch.

> ⚠️ **Avoid:** leaving a high-reasoning frontier model on while asking for repetitive formatting or doc cleanup. That burns budget without improving quality.

---

## 3. AGENTS.md

> AGENTS.md is the repo's operating manual for coding agents. It is the single highest leverage setup file.

### 07 · Discovery and precedence
*Codex combines global and project instructions, then closer project files override earlier ones by appearing later.*

| Layer | Typical location | Role | Precedence behavior |
|---|---|---|---|
| Global override | `~/.codex/AGENTS.override.md` | Personal hard rules. | Preferred over the normal global file when present. |
| Global | `~/.codex/AGENTS.md` | Your stable preferences across projects. | Loaded before project docs. |
| Project root | `<repo>/AGENTS.md` | Repository-wide conventions. | Loaded from project/Git root downward. |
| Nested scope | `frontend/AGENTS.md`, `packages/api/AGENTS.md` | Subtree-specific rules. | Closer files are appended later, so they can override or specialize root rules. |

> 💡 **Important:** keep the file short. Codex stops adding project instruction content once the combined budget reaches the configured cap, which defaults to 32 KiB.

```
repo/
  AGENTS.md                 # global repo rules
  web/
    AGENTS.md               # frontend-specific rules
    src/...
  api/
    AGENTS.md               # backend-specific rules
    src/...

# Starting Codex from repo/web/src loads:
# global ~/.codex file → repo/AGENTS.md → repo/web/AGENTS.md
```

### 08 · A usable AGENTS.md template
*Codex does not need prose. It needs operating constraints.*

```markdown
# AGENTS.md

## Project map
- Web app: `apps/web` (Next.js, App Router)
- API: `apps/api` (FastAPI)
- Shared types: `packages/contracts`

## Setup
- Install: `pnpm install`
- Dev server: `pnpm dev`
- Typecheck: `pnpm typecheck`
- Test all: `pnpm test`
- Test web: `pnpm --filter web test`

## Coding rules
- Do not introduce new runtime dependencies without explaining why.
- Prefer small functions and explicit domain names over generic helpers.
- Keep public API compatibility unless the task asks for a breaking change.
- Never remove error handling to make tests pass.

## Validation
Before reporting done, run the narrowest relevant check. If a check cannot run,
state the exact command attempted and the reason.

## Git and review
- Do not commit unless explicitly asked.
- Summarize touched files and risk areas.
- For non-trivial changes, run a review pass on the diff.
```

> ⚠️ **Do not stuff everything here:** detailed procedures belong in skills; temporary task context belongs in the prompt; secret values belong nowhere near agent-readable instructions.

### 09 · What goes where
*Use the right memory surface for the job.*

| Need | Put it in |
|---|---|
| Permanent repo rules | `AGENTS.md` |
| Personal style defaults | `~/.codex/AGENTS.md` |
| Repeatable method | Skill |
| Tool / API access | MCP server or plugin |
| One-off work order | Prompt |
| Emergent user preference | Memory, where available |
| Hard security constraint | Config / sandbox / policy, not just prose |

---

## 4. Configuration

> Config turns Codex from a generic assistant into a controlled local engineering system.

### 10 · Config layers and precedence
*User, project, profiles, CLI overrides, and trust gates.*

| Layer | Example | Use | Risk |
|---|---|---|---|
| System | `/etc/codex/config.toml` | Org baseline or managed machine defaults. | Can surprise local users if undocumented. |
| User | `~/.codex/config.toml` | Personal defaults: model, reasoning, sandbox, MCP. | Too broad if it assumes every repo is trusted. |
| Project | `.codex/config.toml` | Repo-specific model, roots, hooks, commands, instructions. | Ignored until the project is trusted. |
| Profile | `[profiles.deep-review]` | Quickly switch between modes. | Experimental/CLI-focused; avoid relying on it for team-critical behavior. |
| CLI override | `-c key=value` | One-off precision. | Easy to forget what you changed for that run. |

```toml
# ~/.codex/config.toml — pragmatic local baseline
model = "gpt-5.5"
model_reasoning_effort = "medium"
model_reasoning_summary = "concise"
model_verbosity = "medium"

approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false
writable_roots = ["/tmp/codex-scratch"]

[features]
memories = false          # enable only where available and useful
codex_hooks = false       # turn on when you have deterministic hooks ready

[profiles.deep_review]
model = "gpt-5.5"
model_reasoning_effort = "high"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[profiles.fast_scan]
model = "gpt-5.4-mini"
model_reasoning_effort = "low"
sandbox_mode = "read-only"
```

### 11 · Command-line controls
*Use flags for one-off changes; config for stable defaults.*

```bash
# Start interactive Codex in a repo
codex

# Start with a prompt
codex "Explain this codebase and identify the test commands."

# Add another readable directory
codex --add-dir ../shared-libs

# Set approvals and sandbox for one invocation
codex -a on-request -c sandbox_mode=workspace-write "Fix the failing auth tests."

# Non-interactive scripting
codex exec "Run the minimal check for the current diff and summarize failures."
```

> 🚨 **Do not normalize YOLO mode.** `danger-full-access` plus no approvals is convenient only until untrusted code, a prompt injection, or a mistaken command touches something expensive.

### 12 · Project config strategy
*Make the project reproducible before asking Codex to be smart.*

1. Create a root `.codex/config.toml` only for repo-specific settings.
2. Keep personal preferences in `~/.codex/config.toml`.
3. Keep project setup and conventions in `AGENTS.md`.
4. Use project config for safe writable roots, hooks, or project-specific model defaults.
5. Trust the project deliberately; untrusted project config is skipped by design.

```toml
# .codex/config.toml inside a trusted repo
model_reasoning_effort = "medium"

[sandbox_workspace_write]
writable_roots = ["./tmp", "./.cache"]
network_access = false
```

---

## 5. Security, sandboxing & approvals

> The Codex safety model has two separable parts: what the runtime can technically do, and when it must ask you.

### 13 · Sandbox mode × approval policy
*Permissions are engineering controls. Treat them like production deploy permissions.*

| Control | Options | Meaning | Use |
|---|---|---|---|
| Sandbox mode | `read-only` | Codex can inspect but not modify. | Audits, codebase understanding, threat modeling. |
| Sandbox mode | `workspace-write` | Codex can edit within workspace/writable roots. | Default for normal local implementation. |
| Sandbox mode | `danger-full-access` | Codex can access broadly. | Rare; trusted repos only; short-lived; understand blast radius. |
| Approval policy | `untrusted` | More prompts; cautious mode. | New repos or code you do not trust. |
| Approval policy | `on-request` | Codex asks when it needs powers outside the baseline. | Good default for productive local work. |
| Approval policy | `never` | No human confirmation prompts. | Only with tight sandboxing and known-safe automation. |

> ⚠️ **Network is off by default in many local flows.** That is good. Install dependencies in a controlled setup phase, or grant network only for the commands and environment that need it.

### 14 · Cloud security model
*Cloud tasks run in isolated containers with a two-phase lifecycle.*

1. **Container:** Codex creates an isolated environment and checks out the repo at the selected branch or commit.
2. **Setup phase:** dependencies and setup scripts run; network and secrets can be used depending on environment configuration.
3. **Agent phase:** Codex edits/runs checks; internet is off by default unless enabled for that environment.
4. **Finish:** it returns answer, diff, logs, and can create a PR or continue follow-up work.

> 💡 **Secret handling:** secrets are intended for setup, not agent reasoning. Design setup scripts so the agent can run checks without seeing secrets.

### 15 · Prompt injection posture
*Every external page, README, issue, log, and dependency script can try to steer the agent.*

- Treat browser/page content as untrusted instructions.
- Do not paste secrets into prompts, browser pages, or screenshots.
- Use read-only mode for unknown repos and security review.
- Prefer allowlisted commands for automation.
- Ask Codex to quote evidence paths and command output for risky claims.
- Manually review and validate agent-generated code before integration.

```
# Safer prompt for untrusted code
Audit this repo in read-only mode. Do not run install scripts or network commands.
Find risky scripts, secrets handling, and suspicious postinstall behavior.
Report evidence with file paths and exact lines where possible.
```

---

## 6. Context, memory & compaction

> Context is a scarce working set. Feed Codex what matters and keep mandatory rules out of volatile chat history.

### 16 · Context control
*Good agents are context engineers. Codex will search, but you should still point.*

| Input | Use | Practice |
|---|---|---|
| File/path mentions | Direct Codex to relevant code. | Use `@` file autocomplete or explicit paths. |
| Open IDE files | Fast context for local edits. | Mention hidden callers/tests not open in editor. |
| Images/screenshots | UI bugs, diagrams, error screens. | Pair image with expected behavior and target files. |
| AGENTS.md | Stable project expectations. | Short, scoped, concrete. |
| MCP resources | External docs, databases, design systems, issue trackers. | Add only high-signal servers; too many tools dilute context. |
| Skills | Repeatable procedures. | Use progressive disclosure: short description first, full file only when needed. |

### 17 · Memories
*Useful for preferences; wrong place for hard requirements.*

Memories are opt-in where available, stored under Codex home, and are generated from eligible prior threads. They are useful for recurring preferences and project habits, but not for rules that must be followed every time.

> ⚠️ **Availability caveat:** OpenAI docs note regional/platform limitations for memories and related screen-context features. In the EEA/UK/Switzerland, treat memory availability as something to verify before designing a workflow around it.

```toml
# Config shape
[features]
memories = true

# Thread controls
/memories
```

### 18 · When threads get long
*Long context improves continuity until it starts hiding the current task.*

**Use one thread when…** the task is a coherent unit, previous decisions matter, and the diff is still manageable.

**Fork or start fresh when…** the old thread contains abandoned plans, stale assumptions, or unrelated exploration.

```
# Context reset prompt
We are starting a fresh implementation pass.
Use only these decisions from the prior thread:
1. ...
2. ...
Ignore abandoned approaches A and B.
Now implement the plan in @docs/PLAN.md and run the checks listed there.
```

---

## 7. Workflows

> Codex performs best when you treat it like an engineering teammate with a crisp ticket and a review gate.

### 19 · The prompt formula
*Goal, context, constraints, done criteria. Use it until it becomes muscle memory.*

```
Goal:
  Fix the checkout retry bug where duplicate payment attempts are created after timeout.

Context:
  Relevant files: @apps/api/src/payments, @packages/contracts/src/payment.ts
  Failing test/log: paste or attach here.

Constraints:
  Preserve public API. Do not add dependencies. Do not change database schema.
  Prefer a minimal patch over architectural cleanup.

Done when:
  - Root cause explained.
  - Regression test added.
  - `pnpm --filter api test payments` passes.
  - Diff summary includes risk areas and rollback notes.
```

> ✅ **For hard tasks:** first ask for a plan. Then let Codex implement only after you accept the plan. This prevents the common "plausible but sprawling patch" failure.

### 20 · Implementation loop
*Use Codex as a patch generator plus test runner, not as a magic final authority.*

1. **Inspect:** ask it to find relevant files, tests, and current behavior.
2. **Plan:** require a short plan and risk list before edits.
3. **Patch:** keep edits narrow and visible in the diff.
4. **Test:** run the smallest relevant command, then broader checks.
5. **Review:** run `/review` or a new reviewer pass.
6. **Summarize:** changed files, commands run, failures, risks, next steps.

### 21 · Review workflow
*Separate "builder" and "critic" roles.*

```
# Builder prompt
Implement the accepted plan. Keep the diff minimal. Run the specified checks.

# Reviewer prompt or /review custom instructions
Review this diff as a senior maintainer. Focus on:
- correctness and edge cases
- changed public behavior
- test coverage gaps
- security and data integrity risks
- unnecessary complexity
Do not suggest style churn unless it hides a real issue.
```

> 💡 **Practical test:** if Codex cannot explain what changed and why the checks are sufficient, the task is not done.

### 22 · Workflow recipes
*Reusable task shapes that work well with Codex.*

| Task | Prompt shape | Surface | Extra check |
|---|---|---|---|
| Explain codebase | "Map the architecture, entry points, data flow, and test commands. No edits." | CLI/IDE/app | Ask for evidence paths. |
| Fix CI failure | "Read failing logs, reproduce locally if possible, patch minimal cause, rerun exact check." | CLI/cloud | Compare before/after failure output. |
| Refactor module | "Plan first, preserve API, add characterization tests, then refactor." | CLI/app | Run tests before and after. |
| UI bug | "Use screenshot/browser, inspect component, fix visual behavior, verify in browser." | App/IDE | Attach screenshot or browser observation. |
| Security review | "Read-only audit; no install scripts; cite files and exploit path." | CLI/cloud read-only | Separate likelihood from impact. |
| Migration | "Inventory usage, write migration plan, patch in phases, test each phase." | App/worktrees | Use subagents for inventory; main agent owns final diff. |
| Release notes | "Summarize commits since tag, group by user impact, flag breaking changes." | Automation/cloud | Human approve before publishing. |

---

## 8. Codex app

> The desktop app is the orchestration layer: parallel threads, worktrees, automations, Git review, local environments, browser use, and computer use.

### 23 · Worktrees
*Parallelism without wrecking your main checkout.*

Worktrees are separate checkouts backed by the same Git repository metadata. They let Codex work on separate branches/tasks while your local checkout stays clean.

| Mode | Use | Risk |
|---|---|---|
| Local | Pairing on the code you are actively editing. | Codex can modify files you have open. |
| Worktree | Parallel task, experiment, or automation. | Merge conflicts and stale branch context. |
| Handoff | Move a thread between local and worktree. | Review Git state before and after. |

### 24 · Automations
*Schedule repeatable checks; do not automate chaos.*

- Good: summarize recent commits every morning.
- Good: scan for likely bugs after dependency updates.
- Good: draft weekly release notes from merged PRs.
- Good: check CI failures and propose likely fixes.
- Bad: background full-access agent on untrusted repos.
- Bad: vague recurring prompt with no acceptance criteria.

> 💡 **Useful distinction:** skills define the method; automations define the schedule.

### 25 · Browser and computer use
*Powerful for UI workflows, but treat external content as hostile.*

| Feature | Use | Limitations / cautions |
|---|---|---|
| In-app browser | Preview local dev servers, file-backed pages, public pages, UI bugs. | No normal authenticated browser profile; do not paste secrets; external content is untrusted. |
| Browser use | Let Codex interact with pages after permission. | Ask for scoped tasks and verify actions. |
| Computer use | Operate GUI apps where APIs are absent. | Platform and regional availability varies; permission prompts matter; review before destructive actions. |
| Windows app | Native PowerShell sandbox or WSL2 workflows. | Make sure sandbox mode matches the execution environment. |

---

## 9. Cloud / Web Codex

> Cloud Codex is best for well-scoped delegated work where the repository can be built inside an OpenAI-managed container.

### 26 · Environment lifecycle
*Cloud quality is mostly environment quality.*

1. Connect/select repository and branch/SHA.
2. Container starts from a default or custom image.
3. Setup script installs dependencies and prepares caches.
4. Secrets are available to setup if configured, then removed before agent work.
5. Agent reads instructions, edits files, runs checks, and produces a diff.
6. You review the result, ask follow-ups, or create/continue a PR.

```bash
# Cloud CLI examples
codex cloud "Investigate failing CI on the payments branch and propose a fix."

codex cloud exec --env production-like "Run the backend tests and explain failures."

# Best-of-N cloud attempts for hard tasks
codex cloud exec --attempts 3 "Find three independent fixes for the flaky onboarding test."
```

### 27 · Use cloud when…
*Async, isolated, repeatable, reviewable.*

- The task can be defined from repo state, issue, PR, or branch.
- Setup can run without your local machine.
- Checks are documented and can be run in the container.
- You want multiple attempts or parallel approaches.
- The result can be reviewed as a diff/PR.

> 🚨 **Do not use cloud as a mind reader:** it cannot know your local uncommitted edits, private terminal state, or undocumented setup hacks unless you explicitly provide or encode them.

### 28 · Cloud environment checklist
*Make the container boring before expecting Codex to be brilliant.*

| Item | Question | Fix |
|---|---|---|
| Dependencies | Can setup install exact versions? | Pin runtime versions; use lockfiles; document package manager. |
| Secrets | Can checks run without exposing secrets to the agent? | Use setup-only secrets; create mock/test credentials where possible. |
| Network | Does the agent phase need internet? | Prefer offline agent phase; enable network only when truly needed. |
| Tests | Does AGENTS.md list the right commands? | Add narrow and broad checks with expected runtime. |
| Cache | Is setup slow or stale? | Use cached setup and reset cache after dependency/setup changes. |
| Image | Does default image miss system packages? | Use custom image or setup script to install required tools. |

---

## 10. Skills & plugins

> Skills are reusable agent procedures. Plugins are a distribution mechanism. The goal is repeatability, not more text in every prompt.

### 29 · Skill anatomy
*Codex sees a small description first, then loads the full skill only when it is relevant.*

```
my-release-notes-skill/
  SKILL.md
  scripts/
    collect_commits.py
  references/
    changelog_style.md
  assets/
    release-template.md
  agents/
    openai.yaml       # optional Codex app metadata / invocation policy
```

```markdown
---
name: release-notes
description: Use when preparing customer-facing release notes from commits, PRs, or changelog drafts.
---

# Release notes skill

## When to use
Use this for weekly/monthly release notes, launch notes, or PR-to-release summaries.

## Procedure
1. Collect commits and PRs for the requested range.
2. Group by customer-visible impact.
3. Separate fixes, improvements, breaking changes, and internal-only changes.
4. Draft notes in the house style from `references/changelog_style.md`.
5. Flag claims that need product/legal review.
```

> 💡 **Trigger design:** the description is the skill's front door. If it is vague, Codex will not reliably invoke it. Name the exact situations where the skill should fire.

### 30 · Personal vs team skills
*Keep private habits private; share procedures that raise team quality.*

| Scope | Path | Use |
|---|---|---|
| Personal | `$HOME/.agents/skills` | Your own workflows and preferences. |
| Team/repo | `.agents/skills` | Shared procedures for the repository or organization. |
| Packaged | Plugin | Installable, versioned distribution across projects. |

Start local, iterate, then package or check into the repo when the skill is stable and genuinely useful.

### 31 · Good skill candidates
*If you have prompted it three times, it may be a skill.*

- "Prepare a safe database migration."
- "Debug flaky Playwright test."
- "Implement design system component from Figma and verify states."
- "Write incident postmortem from timeline and logs."
- "Review PR for security and data integrity risks."
- "Modernize package to new framework version."

> ⚠️ **Not a skill:** project facts that never change. Put those in AGENTS.md or docs.

---

## 11. MCP & external tools

> MCP lets Codex reach structured external context and actions. Add it when the tool improves precision; do not add it for novelty.

### 32 · MCP mental model
*Servers expose tools, resources, and prompts; Codex is the client.*

| MCP part | Meaning | Examples |
|---|---|---|
| Tool | Action Codex can call. | Create ticket, query DB, fetch docs, run custom analysis. |
| Resource | Read-only context. | Design system docs, API schema, issue metadata. |
| Prompt | Reusable prompt/template exposed by server. | Company review checklist, support triage procedure. |
| Transport | How client talks to server. | STDIO, Streamable HTTP, bearer auth, OAuth. |

```bash
# Example pattern; adapt to the exact server docs
codex mcp add context7 -- npx -y @upstash/context7-mcp

# In Codex TUI
/mcp
```

### 33 · MCP selection test
*Ask these before connecting another server.*

1. Does this server provide context Codex cannot cheaply read from the repo?
2. Are tool actions deterministic enough to trust?
3. Can access be scoped and audited?
4. Will prompts become shorter and more reliable?
5. Does it increase prompt-injection exposure?

> 🚨 **Security move:** do not grant broad write-capable MCP tools to background automations without separate policy checks.

---

## 12. Subagents

> Subagents are parallel specialists. They are useful for bounded research and review; dangerous when each tries to edit the same surface.

### 34 · When to use subagents
*Use them for independent workstreams, then merge judgment in the main thread.*

| Good subagent task | Why it works | Bad subagent task |
|---|---|---|
| Inventory all call sites of an API. | Read-heavy, parallelizable, bounded. | Each subagent edits call sites independently with no merge plan. |
| Compare three implementation strategies. | Produces options and tradeoffs. | Let three agents ship three incompatible architectures. |
| Audit tests for coverage gaps. | Separate critic role avoids builder bias. | Subagent modifies tests while main agent modifies production code blindly. |
| Explore unfamiliar modules. | Keeps noisy exploration out of main context. | Delegate final decision to a worker without human or main-agent synthesis. |

```
Use subagents for read-only research. Spawn:
1. explorer: map payment retry flow and cite files.
2. reviewer: find data-integrity risks in current implementation.
3. tester: identify existing tests and missing regression cases.

Return their findings, then synthesize one implementation plan. Do not edit yet.
```

> 💡 **Cost reality:** subagents use more tokens than a single-agent pass. Spend them where parallel independent cognition beats linear chat.

---

## 13. Integrations & automation

> Codex becomes more valuable when it is wired into existing developer workflows, not when it becomes a side chat.

### 35 · GitHub workflows
*Issues, PRs, review comments, and CI are natural work queues.*

- Tag Codex on PRs/issues where supported to request review or changes.
- Use GitHub-connected cloud tasks for clean branch-based work.
- Ask for suggested fixes, but require human review before merge.
- For PR review, ask Codex to separate blocking correctness issues from style preferences.

```
@codex review this PR for data integrity and auth regressions.
Focus on changed behavior, missing tests, and security risk.
Ignore formatting unless it hides a bug.
```

### 36 · Scriptable Codex
*Use non-interactive execution for deterministic checks and reports.*

```bash
# Report-only local check
codex exec -c sandbox_mode=read-only \
  "Summarize changed files against main and identify risky areas. No edits."

# Controlled implementation script
codex exec -a on-request -c sandbox_mode=workspace-write \
  "Fix the failing unit test shown in logs/test-failure.txt. Run only pnpm test unit."
```

> ⚠️ **CI caution:** agentic edits in CI need explicit branch, artifact, and review handling. Do not let a bot silently rewrite production code without a PR gate.

### 37 · Operating model for teams
*Decide what humans own, what Codex drafts, and what automation monitors.*

| Function | Codex role | Human role | Control |
|---|---|---|---|
| Feature implementation | Draft patch, tests, summary. | Scope, review, product judgment. | PR review and test gate. |
| Code review | Find likely defects and missing tests. | Prioritize and decide merge. | Separate reviewer prompt/model. |
| Maintenance | Scheduled scans, release notes, dependency triage. | Approve actions and exceptions. | Automations + worktrees. |
| Security | Read-only audits and fix drafts. | Threat modeling, acceptance, disclosure. | Sandbox, no secrets, evidence requirements. |
| Docs | Draft based on code and changes. | Tone, accuracy, customer claims. | Source-path citations and review. |

---

## 14. Patterns that compound

> These are the boring moves that make Codex consistently useful instead of occasionally impressive.

### 38 · Plan mode first
*For anything ambiguous, make Codex explain before it edits.*

Use `/plan` or the app/IDE equivalent before greenfield features, migrations, security fixes, or large refactors. Plans reveal mistaken assumptions before they become code.

### 39 · Narrow checks
*Run the smallest relevant check, then broader checks.*

A 90-second targeted test gives faster feedback than a 25-minute full suite. Ask for both when warranted, but sequence them intentionally.

### 40 · Diff budgets
*Limit blast radius explicitly.*

Tell Codex which files are in scope, which APIs cannot change, and when to stop and ask. Small diffs are easier to review and safer to merge.

### 41 · Evidence prompts
*Make claims traceable.*

```
For every claim about existing behavior, cite the file path and function/test/log that supports it.
```

### 42 · Retrospective loop
*Turn repeated failures into setup.*

When Codex repeats a mistake, do not just correct it in chat. Decide whether the fix belongs in AGENTS.md, config, a skill, a test, or a hook.

### 43 · Separate exploration from execution
*Read-only first for unfamiliar systems.*

Let Codex map the territory before it edits. For complex systems, "investigate and propose" is often the highest-value first turn.

---

## 15. Troubleshooting

> Most Codex failures are not mysterious. Diagnose the layer: context, instructions, permissions, environment, or task shape.

### 44 · Symptom → likely cause → fix
*A fast field diagnostic table.*

| Symptom | Likely cause | Fix |
|---|---|---|
| Codex ignores project conventions. | AGENTS.md not found, too large, wrong CWD, or nested override missing. | Run from repo/subdir intentionally; ask Codex to summarize active instructions; trim AGENTS.md. |
| Project config not applied. | Project is untrusted or precedence overridden. | Trust the project; inspect user/project config; use one-off `-c` to test. |
| Cannot edit or write temp files. | Sandbox blocks path. | Add a writable root or move output inside workspace; avoid full access unless necessary. |
| Cannot install/fetch dependencies. | Network disabled. | Use setup phase, allowlist network temporarily, or vendor required dependencies. |
| Cloud task fails before agent starts. | Bad setup script, missing runtime, stale cache, secret mismatch. | Reproduce setup commands; pin versions; reset cache after changes. |
| Memory seems absent. | Feature disabled, unavailable, not enough eligible idle thread history, or region/platform limit. | Verify settings and availability; keep required rules in AGENTS.md. |
| Skills do not trigger. | Description too vague, skill too broad, or invocation not explicit. | Improve description; invoke with `$skill-name` or `/skills`; keep scope sharp. |
| Patch sprawls beyond request. | Prompt lacked constraints or Codex optimized architecture instead of minimal fix. | Set file/diff/API limits; ask for plan first; stop and restart with narrower scope. |
| Tests pass but change is risky. | Tests insufficient; review missing edge cases. | Run reviewer pass; ask for missing regression tests and behavioral risks. |
| Automation creates clutter. | Too many worktrees/runs retained. | Archive old runs; avoid pinning unless you need the worktree. |

### 45 · Recovery prompts
*Use these when a thread drifts.*

```
# Recover from drift
Stop editing. Summarize the current diff, the original goal, and every way the diff exceeds scope.
Then propose a minimal rollback or narrow patch plan. Do not modify files yet.

# Recover from test failure
Do not guess. Read the failing output, identify the exact failing assertion or stack frame,
trace it to code, and explain the root cause before patching.

# Recover from config confusion
List the model, reasoning effort, sandbox mode, approval policy, working directory,
active AGENTS.md files, and relevant config layers you can infer.
```

### 46 · When to stop Codex
*Agentic persistence is useful until it becomes thrashing.*

- It patches the symptom repeatedly without explaining the cause.
- It proposes broad rewrites to avoid a small bug.
- It cannot run or cite the relevant check.
- It starts changing unrelated files.
- It treats web/page text as authoritative instructions.

> 🚨 **Intervene early:** cancel, ask for a diff/scope summary, then restart with a tighter prompt or higher reasoning.

---

## 16. Sources & research base

This guide prioritizes official OpenAI Codex docs, the open-source Codex repository, AGENTS.md public guidance, and the Codex changelog.

### Primary sources used

- [OpenAI Codex CLI](https://developers.openai.com/codex/cli)
- [Codex CLI features](https://developers.openai.com/codex/cli/features)
- [Codex CLI command reference](https://developers.openai.com/codex/cli/reference)
- [Codex best practices](https://developers.openai.com/codex/learn/best-practices)
- [Custom instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md)
- [AGENTS.md public format](https://agents.md/)
- [Config basics](https://developers.openai.com/codex/config-basic)
- [Advanced configuration](https://developers.openai.com/codex/config-advanced)
- [Configuration reference](https://developers.openai.com/codex/config-reference)
- [Agent approvals & security](https://developers.openai.com/codex/agent-approvals-security)
- [Sandboxing](https://developers.openai.com/codex/concepts/sandboxing)
- [Codex app](https://developers.openai.com/codex/app)
- [Codex app worktrees](https://developers.openai.com/codex/app/worktrees)
- [Codex app automations](https://developers.openai.com/codex/app/automations)
- [Codex Cloud / Web docs](https://developers.openai.com/codex/cloud)
- [Agent skills](https://developers.openai.com/codex/skills)
- [MCP in Codex](https://developers.openai.com/codex/mcp)
- [Subagents](https://developers.openai.com/codex/subagents)
- [Codex memories](https://developers.openai.com/codex/memories)
- [Codex changelog](https://developers.openai.com/codex/changelog)
- [openai/codex GitHub repository](https://github.com/openai/codex)
- [Introducing Codex](https://openai.com/index/introducing-codex/)

> ⚠️ **Scope note:** this is a practical guide, not a reverse-engineering claim. Where OpenAI documents behavior, the guide states it. Where internals are not public, the guide uses safe operational mental models rather than pretending to know hidden implementation details.

### Acknowledgments

- **[OpenAI](https://github.com/openai)** — Creators of Codex. Official docs at [developers.openai.com/codex](https://developers.openai.com/codex), open-source CLI at [github.com/openai/codex](https://github.com/openai/codex).
- **[AGENTS.md format](https://agents.md/)** — Public specification for the agent-instruction file format.
- Inspired in structure by the [Claude Code Mastery](https://github.com/FelixKruger/claude-code-mastery) guide; written from independent research with no copied content or styling.

---

*Codex Mastery · static single-page field guide · researched and generated 26 April 2026. Compiled by [@FelixKruger](https://github.com/FelixKruger). Share freely. Review official docs for pricing, plan availability, regional restrictions, and version-specific behavior before making procurement or security decisions. If something here is wrong, misattributed, or you'd like credit adjusted — open an issue or PR on [GitHub](https://github.com/FelixKruger/Codex-Mastery).*
