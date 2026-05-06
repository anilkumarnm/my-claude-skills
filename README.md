<p align="center">
  <img src="https://img.shields.io/badge/Claude-Skills-blueviolet?style=for-the-badge&logo=anthropic" alt="Claude Skills"/>
  <img src="https://img.shields.io/badge/Version-1.0.0-green?style=for-the-badge" alt="Version"/>
  <img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" alt="License"/>
</p>

# 🛠️ My Claude Skills

A collection of reusable Claude Code skills for **software engineering workflows** — from ideation to implementation. These skills help you maintain domain documentation, break work into vertical slices, and publish issues to your tracker of choice.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Skills](#-skills)
  - [setup-prereq-skills](#-setup-prereq-skills)
  - [ideate-with-docs](#-ideate-with-docs)
  - [to-prd](#-to-prd)
  - [to-issues](#-to-issues)
- [Workflow](#-workflow)
- [Installation](#-installation)
- [Supported Issue Trackers](#-supported-issue-trackers)
- [File Structure](#-file-structure)
- [Configuration](#-configuration)

---

## 🎯 Overview

These skills implement a **documentation-first engineering workflow**:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Setup Prereqs  │───▶│ Ideate Session  │───▶│   Create PRD    │───▶│  Break to Issues│
│                 │    │                 │    │                 │    │                 │
│ /setup-prereq-  │    │ /ideate-with-   │    │    /to-prd      │    │   /to-issues    │
│    skills       │    │    docs         │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
        │                      │                      │                      │
        ▼                      ▼                      ▼                      ▼
   docs/agents/           CONTEXT.md              docs/prd/            Issue Tracker
   issue-tracker.md       docs/adr/            0001-feature.md      (Jira/GitLab/Local)
```

### Key Principles

| Principle | Description |
|-----------|-------------|
| **Domain-First** | Sharpen terminology before writing code. `CONTEXT.md` is your glossary. |
| **Vertical Slices** | Break work into thin, end-to-end slices — not horizontal layers. |
| **ADRs Sparingly** | Only record decisions that are hard to reverse and surprising without context. |
| **Tracker Agnostic** | Works with Jira, GitLab, or local markdown files. |

---

## 🔧 Skills

### 📦 setup-prereq-skills

> **Run this first!** Configures your repo for all other skills.

```
/setup-prereq-skills
```

**What it does:**
1. Explores your repo to detect existing configuration
2. Walks you through three decisions (one at a time):
   - **Issue Tracker** — Where issues live (Jira, GitLab, or local markdown)
   - **Triage Labels** — Map the five canonical roles to your label vocabulary
   - **Domain Docs** — Single-context or multi-context layout
3. Writes configuration to `docs/agents/` and updates `CLAUDE.md`

**Output files:**
```
docs/agents/
├── issue-tracker.md   # How to create/read/update issues
├── triage-labels.md   # Label mapping for triage workflow
└── domain.md          # Where CONTEXT.md and ADRs live
```

**Triage Label Vocabulary:**

| Role | Default Label | Meaning |
|------|---------------|---------|
| Needs evaluation | `needs-triage` | Maintainer needs to look at this |
| Waiting on reporter | `needs-info` | Blocked on more information |
| AFK-ready | `ready-for-agent` | Fully specified, agent can implement |
| Human-required | `ready-for-human` | Needs human implementation |
| Won't fix | `wontfix` | Will not be actioned |

---

### 💡 ideate-with-docs

> Stress-test your plan against the domain model. Sharpen terminology. Update docs inline.

```
/ideate-with-docs
```

**What it does:**
- Interviews you relentlessly about every aspect of your plan
- Challenges fuzzy language and proposes precise terms
- Cross-references claims against actual code
- Updates `CONTEXT.md` as terms are resolved
- Offers ADRs sparingly (only when truly needed)

**When to use:**
- Before starting a new feature
- When terminology feels unclear
- When making architectural decisions

**CONTEXT.md Format:**
```markdown
# {Context Name}

{One or two sentence description.}

## Language

**Order**:
A request from a customer to purchase items.
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent after delivery.
_Avoid_: Bill, payment request

## Relationships

- An **Order** produces one or more **Invoices**
- An **Invoice** belongs to exactly one **Customer**

## Example dialogue

> **Dev:** "When does an **Invoice** get created?"
> **Domain expert:** "Only after **Fulfillment** is confirmed."

## Flagged ambiguities

- "account" was used for both **Customer** and **User** — resolved as distinct concepts.
```

**ADR Criteria — All three must be true:**
1. ⚠️ **Hard to reverse** — Cost of changing later is meaningful
2. 🤔 **Surprising** — Future reader will wonder "why?"
3. ⚖️ **Real trade-off** — Genuine alternatives existed

---

### 📄 to-prd

> Turn conversation context into a structured PRD.

```
/to-prd
```

**What it does:**
1. Explores the codebase using domain vocabulary
2. Identifies modules to build/modify (prefers deep modules)
3. Writes a comprehensive PRD to `docs/prd/`

**PRD Structure:**
```markdown
# Feature Name

**Status:** needs-triage

## Problem Statement
The problem from the user's perspective.

## Solution
The solution from the user's perspective.

## User Stories
1. As a <actor>, I want <feature>, so that <benefit>
2. ...extensive list...

## Implementation Decisions
- Modules to build/modify
- API contracts
- Schema changes
- Architectural decisions

## Testing Decisions
- What makes a good test
- Which modules to test
- Prior art in codebase

## Out of Scope
What we're NOT doing.

## Further Notes
Additional context.
```

**File location:** `docs/prd/0001-feature-name.md`

---

### 🎯 to-issues

> Break a plan into vertical slices and publish to your issue tracker.

```
/to-issues
```

**What it does:**
1. Gathers context from conversation or existing issue
2. Drafts **tracer bullet** vertical slices (not horizontal layers!)
3. Quizzes you on granularity and dependencies
4. Saves implementation plan to `docs/impl-plan/`
5. Asks for issue assignee
6. Publishes issues in dependency order

**Vertical Slice Rules:**
- ✅ Each slice cuts through ALL layers (schema → API → UI → tests)
- ✅ Each slice is demoable/verifiable on its own
- ✅ Prefer many thin slices over few thick ones
- ❌ NOT horizontal slices (e.g., "do all the database work")

**Slice Types:**

| Type | Description |
|------|-------------|
| **AFK** | Can be implemented and merged without human interaction |
| **HITL** | Requires human decision (architecture, design review) |

**Issue Template:**
```markdown
## Parent
Reference to parent issue (if applicable)

## What to build
End-to-end behavior description (not layer-by-layer)

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by
- PROJ-123: Dependency issue
Or "None - can start immediately"
```

**Output:**
- `docs/impl-plan/<feature>/0001-implementation-plan.md` — Local record with Jira links
- Issues created in your tracker with `needs-triage` label

---

## 🔄 Workflow

### Recommended Order

```
1. /setup-prereq-skills  ← Run once per repo
         │
         ▼
2. /ideate-with-docs      ← Sharpen the plan, update domain docs
         │
         ▼
3. /to-prd               ← Create formal PRD
         │
         ▼
4. /to-issues            ← Break into vertical slices, publish
```

### Example Session

```bash
# First time setup
> /setup-prereq-skills
# Answer: Jira, default labels, single-context

# New feature discussion
> I want to add metadata validation after pipeline completion
> /ideate-with-docs
# Answer questions, watch CONTEXT.md update

# Formalize the plan
> /to-prd
# Review the PRD in docs/prd/

# Create implementation tickets
> /to-issues
# Review slices, approve, assign to yourself
# Issues appear in Jira!
```

---

## 📥 Installation

### Option 1: Copy to Project

```bash
# Copy skills to your project's .claude/skills/
cp -r my-claude-skills/* /path/to/your/project/.claude/skills/
```

### Option 2: Symlink (for shared use)

```bash
# Symlink from your project
ln -s /path/to/my-claude-skills /path/to/your/project/.claude/skills
```

### Option 3: User-level Skills

```bash
# Copy to ~/.claude/skills/ for global availability
cp -r my-claude-skills/* ~/.claude/skills/
```

---

## 🎫 Supported Issue Trackers

### Jira

**Required environment variables:**
```bash
export JIRA_PAT="<your-personal-access-token>"
export JIRA_BASE_URL="https://your-instance.atlassian.net"
export JIRA_PROJECT_KEY="PROJ"
```

**Supported operations:**
- Create issue
- Read issue
- Search (JQL)
- Add comment
- Update labels
- Transition (close, etc.)

### GitLab

**Requirements:** [glab CLI](https://gitlab.com/gitlab-org/cli) installed and authenticated

**Supported operations:**
- Create issue (`glab issue create`)
- Read issue (`glab issue view`)
- List issues (`glab issue list`)
- Comment (`glab issue note`)
- Update labels (`glab issue update`)
- Close (`glab issue close`)

### Local Markdown

**No external dependencies!** Issues stored as files:

```
.scratch/
└── <feature-slug>/
    ├── PRD.md
    └── issues/
        ├── 01-first-slice.md
        ├── 02-second-slice.md
        └── 03-third-slice.md
```

---

## 📁 File Structure

```
my-claude-skills/
├── README.md                          # This file
│
├── setup-prereq-skills/               # 📦 First-run configuration
│   ├── SKILL.md                       # Skill definition
│   ├── issue-tracker-jira.md          # Jira template
│   ├── issue-tracker-gitlab.md        # GitLab template
│   ├── issue-tracker-local.md         # Local markdown template
│   ├── triage-labels.md               # Label mapping template
│   └── domain.md                      # Domain docs template
│
├── ideate-with-docs/                  # 💡 Domain ideation
│   ├── SKILL.md                       # Skill definition
│   ├── CONTEXT-FORMAT.md              # CONTEXT.md format guide
│   └── ADR-FORMAT.md                  # ADR format guide
│
├── to-prd/                            # 📄 PRD generation
│   └── SKILL.md                       # Skill definition
│
└── to-issues/                         # 🎯 Issue breakdown
    └── SKILL.md                       # Skill definition
```

---

## ⚙️ Configuration

### After Running setup-prereq-skills

Your project will have:

```
your-project/
├── CLAUDE.md                          # Updated with ## Agent Skills section
└── docs/
    └── agents/
        ├── issue-tracker.md           # Your tracker config
        ├── triage-labels.md           # Your label mapping
        └── domain.md                  # Your domain doc layout
```

### CLAUDE.md Agent Skills Section

```markdown
## Agent Skills

### Issue Tracker
Issues tracked in Jira via REST API. See `docs/agents/issue-tracker.md`.

### Triage Labels
Default vocabulary. See `docs/agents/triage-labels.md`.

### Domain Docs
Single-context with CONTEXT.md at root. See `docs/agents/domain.md`.
```

---

## 📚 Related Concepts

| Concept | Description | Reference |
|---------|-------------|-----------|
| **Vertical Slice** | End-to-end feature through all layers | [Vertical Slice Architecture](https://jimmybogard.com/vertical-slice-architecture/) |
| **Tracer Bullet** | Thin path through entire system | [Pragmatic Programmer](https://pragprog.com/titles/tpp20/) |
| **ADR** | Architecture Decision Record | [ADR GitHub](https://adr.github.io/) |
| **Deep Module** | High functionality, simple interface | [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201) |
| **Domain Language** | Ubiquitous language from DDD | [Domain-Driven Design](https://www.domainlanguage.com/) |

---

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch
3. Add or modify skills following the existing patterns
4. Update this README
5. Submit a pull request

---

## 📄 License

MIT License — Use freely, attribution appreciated.

---

<p align="center">
  <i>Built for Claude Code by engineers who believe documentation should come before implementation.</i>
</p>
