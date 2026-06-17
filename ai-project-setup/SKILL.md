# AI Project Setup & Agent Optimization Skill

## What this skill does

When invoked, this skill instructs you on how to structure any new project or multi-package repository so that AI coding agents (Claude, Cursor, Grok, or similar) remain sharp, self-improving, and context-efficient across every session — for the lifetime of the project.

Apply this skill whenever:
- Setting up a new project or monorepo from scratch
- Adding a new package or sub-directory to an existing project
- Noticing agent performance degrading over long sessions
- Onboarding a new agent into an existing codebase

---

## Core Principles

### Principle 1 — Granular, Directory-Level SKILL.md Files

Every directory in every project gets its own `SKILL.md`. Not one global file. One per directory.

**Why:** An agent working inside `apps/web/` should not have to parse instructions for `packages/api/`. Directory-scoped files keep context tight, relevant, and fast to load.

**What each SKILL.md must contain:**

```markdown
# [Directory Name] — Agent Instructions

## Purpose
One sentence: what this directory does and why it exists.

## Run
Commands to start, build, test, and lint this package.
Example:
  npm run dev        # dev server
  npm run build      # production build
  npm run test       # unit tests
  npm run lint       # lint check

## Debug
Known errors + their fixes. Grows over time as the agent discovers new issues.
Example:
  Error: "Module not found: @repo/ui"
  Fix: Run `npm install` from repo root, not from this directory.

## Deploy
How to ship this package (CI command, platform, required env vars).

## Conventions
Project-specific rules the agent must follow in this directory only.
Examples:
  - All components use named exports, never default.
  - Tailwind only — no inline styles.
  - API calls go through /lib/api.ts, never fetch() directly.

## Findings Log
Agent appends here automatically when it discovers something non-obvious.
Format: [DATE] — [finding] — [how it was resolved]
```

**Single-package project layout:**
```
project-root/
├── SKILL.md          ← root: package manager, test suite, branch/PR rules, deploy
├── CLAUDE.md         ← entry point: @AGENTS.md @HANDOFF.md
├── AGENTS.md         ← agent rules + skill trigger map
└── HANDOFF.md        ← current project state (updated each session)
```

**Multi-package / monorepo layout:**
```
project-root/
├── SKILL.md                        ← root: workspace commands, shared conventions, CI/CD
├── CLAUDE.md
├── AGENTS.md
├── HANDOFF.md
├── apps/
│   ├── web/
│   │   └── SKILL.md               ← Next.js conventions, Tailwind/shadcn configs, component rules, testing
│   └── mobile/
│       └── SKILL.md               ← React Native: navigation patterns, styling, simulator flags
├── packages/
│   ├── ui/
│   │   └── SKILL.md               ← Shared component library: export rules, Storybook, versioning
│   ├── api/
│   │   └── SKILL.md               ← API layer: auth pattern, error handling, env vars
│   └── config/
│       └── SKILL.md               ← Shared config: tsconfig, eslint, tailwind base
└── scripts/
    └── SKILL.md                   ← Build scripts: what each does, when to run, failure modes
```

**Rules for writing SKILL.md files:**
- Write for the agent, not for humans — be explicit, not polite
- Use exact commands, not descriptions of commands
- Errors section is the most valuable section — fill it aggressively
- Keep each file under 200 lines; if it grows beyond that, split the directory

---

### Principle 2 — Self-Improving Memory Framework

The agent must update SKILL.md files as it works. This is non-negotiable.

**The problem it solves:** AI agents degrade in performance as context length grows. A session that started sharp becomes slow and error-prone after dozens of tool calls. The standard fix — starting a new chat — loses all accumulated knowledge from the session.

**The solution:** Every finding gets written to the relevant SKILL.md immediately. New sessions start fresh but inherit all past discoveries. Performance stays high. Knowledge compounds.

**How to implement — add this block to your root AGENTS.md:**

```markdown
## Self-Improving Memory Rule

After every session in which you:
- Fix a bug that wasn't obvious from reading the code
- Discover a project convention not yet documented
- Find that a documented command is wrong or outdated
- Learn how a specific error is resolved in this codebase

You MUST append a finding to the Findings Log of the relevant SKILL.md before the session ends.

Format:
[YYYY-MM-DD] — [what you found] — [how it was resolved or why it matters]

Example:
[2026-06-17] — `npm run build` fails if .env.local is missing NEXT_PUBLIC_API_URL — copy from .env.example and restart dev server.

Do not ask for permission. Do not summarize. Just write it.
```

**When to write a finding (trigger conditions):**
- You hit an error not documented in the current SKILL.md
- You had to try more than one approach to solve something
- A command from the SKILL.md didn't work and you found the correct one
- You discovered a dependency, ordering requirement, or environment constraint
- A pattern in this codebase differs from the framework's defaults

**What NOT to write:**
- Things already in the SKILL.md
- General programming knowledge (not project-specific)
- Temporary state or in-progress work
- Anything that will be obvious from reading the code

---

### Principle 3 — HANDOFF.md as the Session Bridge

HANDOFF.md is the living document that carries state between sessions without growing the context window.

```markdown
# HANDOFF — [Project Name]

> Read this instead of the full chat history. ~5 min to be productive.

## Current state
What is built, what works, what is broken right now.

## Next step
The single most important thing to do next. One thing only.

## Decisions made
Non-obvious choices and why they were made (so the agent doesn't re-litigate them).

## Blockers
What is stuck and why. What's needed to unblock.
```

**Rules:**
- Update HANDOFF.md at the end of every session
- "Current state" is a fact, not a summary — list page/feature names with ✅/❌/🔄
- "Next step" is singular — if there are three next steps, the agent picks the most important one
- Never delete old decisions — they prevent circular reasoning across sessions

---

## Implementation Checklist

When setting up a new project, complete these in order:

**Session 1 — Scaffolding**
- [ ] Create `CLAUDE.md` → `@AGENTS.md` `@HANDOFF.md`
- [ ] Create `AGENTS.md` → stack rules + skill trigger map + self-improving memory rule
- [ ] Create root `SKILL.md` → package manager, test command, deploy command, conventions
- [ ] Create `HANDOFF.md` → initial state (empty is fine)
- [ ] For each sub-directory: create its own `SKILL.md` with run/debug/deploy/conventions

**Each subsequent session**
- [ ] Read HANDOFF.md before touching any code
- [ ] Check the relevant directory's SKILL.md before running any command
- [ ] Append any new finding to the correct SKILL.md during the session
- [ ] Update HANDOFF.md before ending the session

**When adding a new package or directory**
- [ ] Create `SKILL.md` in the new directory immediately — don't wait until there's something to put in it
- [ ] Add the package to the root SKILL.md's workspace commands section
- [ ] Add the package's skill trigger to AGENTS.md if relevant

---

## Anti-Patterns to Avoid

| Anti-pattern | Problem | Fix |
|---|---|---|
| One global SKILL.md for the whole repo | Agent loads irrelevant context; files bloat fast | One SKILL.md per directory |
| Never writing findings | Every session re-discovers the same bugs | Mandatory findings log in AGENTS.md |
| Updating HANDOFF.md only when things are clean | Misrepresents actual state | Update it honestly, including broken things |
| Writing SKILL.md as prose | Agents scan, not read — prose is slow | Use headers, code blocks, and bullet points only |
| Skipping SKILL.md for "simple" directories | Simple directories become complex; no docs = no memory | Create the file even if it's 5 lines |

---

## Skill Trigger for AGENTS.md

Add this to any project's AGENTS.md skill trigger map:

```markdown
| Setting up a new project or monorepo | `ai-project-setup` |
| Adding a new package or sub-directory | `ai-project-setup` |
| Agent seems slow, confused, or re-making past mistakes | `ai-project-setup` Principle 2 |
| Onboarding into an unfamiliar codebase | `ai-project-setup` |
```
