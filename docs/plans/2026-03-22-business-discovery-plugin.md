# Business Discovery Plugin Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a Claude Code plugin that guides users through business idea discovery and generates comprehensive project documentation via `/discover` slash command.

**Architecture:** Single monolithic skill in a marketplace-style plugin repo. A `/discover` command invokes the `business-discovery` skill which runs a two-phase guided conversation (business discovery, then technical planning), generating 8 documents total.

**Tech Stack:** Claude Code plugin system (markdown skills, JSON config, slash commands)

---

### Task 1: Create marketplace scaffolding

**Files:**
- Create: `.claude-plugin/marketplace.json`

**Step 1: Create the marketplace directory and file**

```json
{
  "name": "claude-plugins",
  "owner": {
    "name": "borys"
  },
  "metadata": {
    "description": "Personal Claude Code plugin collection",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "business-discovery",
      "source": "./plugins/business-discovery",
      "skills": "./",
      "description": "Guided business idea discovery and project documentation generator"
    }
  ]
}
```

**Step 2: Verify file exists**

Run: `cat .claude-plugin/marketplace.json`
Expected: Valid JSON with plugins array containing business-discovery entry

**Step 3: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: add marketplace scaffolding for plugin collection"
```

---

### Task 2: Create plugin metadata

**Files:**
- Create: `plugins/business-discovery/.claude-plugin/plugin.json`

**Step 1: Create the plugin directory structure**

Run: `mkdir -p plugins/business-discovery/.claude-plugin plugins/business-discovery/commands plugins/business-discovery/skills/business-discovery plugins/business-discovery/hooks`

**Step 2: Create plugin.json**

```json
{
  "name": "business-discovery",
  "description": "Discover business ideas and generate comprehensive project documentation",
  "version": "1.0.0",
  "author": {
    "name": "borys"
  }
}
```

**Step 3: Verify structure**

Run: `find plugins/ -type f`
Expected: `plugins/business-discovery/.claude-plugin/plugin.json`

**Step 4: Commit**

```bash
git add plugins/business-discovery/.claude-plugin/plugin.json
git commit -m "feat: add business-discovery plugin metadata"
```

---

### Task 3: Create the /discover slash command

**Files:**
- Create: `plugins/business-discovery/commands/discover.md`

**Step 1: Write the command file**

The command file has YAML frontmatter and a body that invokes the skill. Use `disable-model-invocation: true` so only users can trigger it. Reference format from the `revise-claude-md.md` example.

```markdown
---
description: "Discover a business idea and generate full project documentation"
disable-model-invocation: true
argument-hint: "[app-name]"
---

Invoke the business-discovery:business-discovery skill and follow it exactly as presented to you.

If the user provided an app name as an argument, use it as the project name throughout the workflow. If no argument was provided, ask for the project name as the first question.

The argument provided was: $ARGUMENTS
```

**Step 2: Verify file**

Run: `cat plugins/business-discovery/commands/discover.md`
Expected: YAML frontmatter with description, disable-model-invocation, argument-hint, followed by skill invocation body

**Step 3: Commit**

```bash
git add plugins/business-discovery/commands/discover.md
git commit -m "feat: add /discover slash command"
```

---

### Task 4: Create the SKILL.md — Frontmatter and Overview

**Files:**
- Create: `plugins/business-discovery/skills/business-discovery/SKILL.md`

**Step 1: Write the frontmatter and overview section**

This is the first part of the skill file. The full SKILL.md will be built across Tasks 4-7.

```markdown
---
name: business-discovery
description: "Guided business idea discovery and project documentation generator. Walks users through two phases — business discovery and technical planning — asking questions one at a time, then generating comprehensive documentation sufficient for project implementation."
tools: Read, Write, Glob, Bash, Edit
disable-model-invocation: true
---

# Business Discovery

Guide the user through discovering and documenting a business idea. Generate comprehensive project documentation that is sufficient for a developer to implement the project later.

**You are a business analyst and technical architect.** Your job is to ask smart questions, synthesize answers into professional documents, and never write implementation code.

## Hard Rules

1. **Documentation only** — never generate implementation code, boilerplate, or scaffolding
2. **One question at a time** — never ask multiple questions in a single message
3. **Multiple choice preferred** — use AskUserQuestion with options when possible; open-ended only when the answer space is too large
4. **Phase gates are mandatory** — summarize findings, get explicit approval before generating documents
5. **All documents written to user-specified directory** — ask for the output path before generating anything
```

**Step 2: Verify frontmatter parses correctly**

Run: `head -8 plugins/business-discovery/skills/business-discovery/SKILL.md`
Expected: Valid YAML frontmatter with name, description, tools, disable-model-invocation

**Step 3: Commit**

```bash
git add plugins/business-discovery/skills/business-discovery/SKILL.md
git commit -m "feat: add SKILL.md frontmatter and overview"
```

---

### Task 5: Write SKILL.md — Workflow Structure and Phase 1

**Files:**
- Modify: `plugins/business-discovery/skills/business-discovery/SKILL.md`

**Step 1: Append the workflow overview and Phase 1 section**

Add the following after the Hard Rules section:

```markdown

## Workflow Overview

```
Phase 0: Setup → Phase 1: Business Discovery → Phase 1 Gate → Phase 1 Docs
→ Phase 2: Technical Planning → Phase 2 Gate → Phase 2 Docs → Summary
```

### Phase 0: Setup

1. If the user provided an app name argument, use it. Otherwise ask:
   - "What's the name of your project/app?" (open-ended)
2. Ask: "Where should I save the project documentation?"
   - Use AskUserQuestion with options:
     - `~/Documents/Apps/[app-name]` (suggested default)
     - `Current directory ./[app-name]`
     - "I'll specify a custom path"
3. Create the output directory if it doesn't exist: `mkdir -p [chosen-path]`

### Phase 1: Business Discovery

Ask these questions **one at a time**. Wait for each answer before asking the next. Adapt follow-up questions based on answers.

**Question 1 — The Problem:**
"What problem are you solving? Describe the pain point your target users experience."
(Open-ended — the answer space is too large for options)

**Question 2 — Target Audience:**
Use AskUserQuestion:
- "Who is your primary target audience?"
- Options: B2C Consumers, B2B Small/Medium Businesses, B2B Enterprise, Developers/Engineers, Creators/Freelancers

**Question 3 — Monetization Model:**
Use AskUserQuestion:
- "What's your preferred monetization model?"
- Options: SaaS Subscription (recurring monthly/annual), Freemium (free tier + paid upgrades), Marketplace/Commission (take a cut of transactions), One-Time Purchase, Usage-Based Pricing (pay per use/API call)

**Question 4 — Competitive Advantage:**
"What makes your solution better than existing alternatives? What's your unfair advantage?"
(Open-ended)

**Question 5 — Revenue Target:**
Use AskUserQuestion:
- "What's your target Monthly Recurring Revenue (MRR) within 12 months?"
- Options: ~$1K MRR (lifestyle/side project), ~$5K MRR (serious side business), ~$10K MRR (full-time viable), $50K+ MRR (scale-up)

**Question 6 — Follow-ups:**
Based on answers, ask 1-3 additional questions to clarify gaps. Examples:
- If B2B: "What's your expected deal size and sales cycle?"
- If marketplace: "Who are the two sides of your marketplace?"
- If freemium: "What feature would make free users upgrade?"

### Phase 1 Gate

Present a structured summary of all business discovery findings:

```
## Business Discovery Summary

**Project:** [name]
**Problem:** [one paragraph]
**Target Audience:** [segment]
**Monetization:** [model]
**Competitive Edge:** [summary]
**Revenue Target:** [MRR goal]
**Key Insights:** [from follow-ups]
```

Ask: "Does this summary accurately capture your business idea? Should I adjust anything before generating the business documents?"

**Wait for explicit approval before proceeding.**

### Phase 1 Document Generation

After approval, generate these 4 files in the output directory. Each document should be thorough, specific to the user's answers, and professionally written.

**1. project-charter.md**
- Project name, vision statement, mission
- Business objectives (SMART goals)
- Success criteria and KPIs
- Target audience profile
- Key stakeholders and roles
- Constraints and assumptions
- High-level timeline (discovery → MVP → launch)

**2. business-model-canvas.md**
- All 9 BMC blocks filled with specifics from the conversation:
  - Customer Segments, Value Propositions, Channels
  - Customer Relationships, Revenue Streams, Key Resources
  - Key Activities, Key Partnerships, Cost Structure
- Each block should have 3-5 bullet points minimum

**3. monetization-strategy.md**
- Pricing model details (tiers, pricing points)
- Revenue projections (month 1, 3, 6, 12)
- Customer acquisition cost estimates
- Lifetime value analysis
- Competitive pricing comparison
- Pricing psychology rationale

**4. go-to-market-plan.md**
- Pre-launch activities (audience building, waitlist)
- Launch strategy (channels, messaging, timing)
- Growth channels ranked by expected ROI
- Content/marketing strategy
- Key metrics to track
- First 90 days action plan

After writing all 4 files, list them and ask: "Phase 1 documents are ready. Would you like to review them before we move to technical planning?"
```

**Step 2: Verify the file is well-formed**

Run: `wc -l plugins/business-discovery/skills/business-discovery/SKILL.md`
Expected: Line count significantly increased

**Step 3: Commit**

```bash
git add plugins/business-discovery/skills/business-discovery/SKILL.md
git commit -m "feat: add Phase 1 business discovery workflow to SKILL.md"
```

---

### Task 6: Write SKILL.md — Phase 2 Technical Planning

**Files:**
- Modify: `plugins/business-discovery/skills/business-discovery/SKILL.md`

**Step 1: Append Phase 2 section**

Add after Phase 1 Document Generation:

```markdown

### Phase 2: Technical Planning

Ask these questions **one at a time**. Adapt based on the business context from Phase 1.

**Question 1 — Tech Stack Preference:**
Use AskUserQuestion:
- "What's your preferred tech stack?"
- Options:
  - Next.js + Node.js (full JavaScript stack, fastest to MVP)
  - Next.js frontend + C# / .NET backend (enterprise-grade, strong typing)
  - React + Node.js (flexible, decoupled frontend/backend)
  - Let me recommend based on your requirements

If "Let me recommend" is chosen, analyze the business requirements and recommend with rationale.

**Question 2 — Key Integrations:**
Use AskUserQuestion with multiSelect:
- "Which integrations does your MVP need?"
- Options: Authentication (OAuth/SSO), Payment Processing (Stripe/etc.), Email/Notifications, Analytics/Tracking, Database, File Storage/CDN, AI/ML Features

**Question 3 — Deployment:**
Use AskUserQuestion:
- "Where do you want to deploy?"
- Options: Vercel (easiest for Next.js), AWS (most flexible), Azure (best for C#/.NET), Self-hosted / VPS

**Question 4 — Scale Expectations:**
Use AskUserQuestion:
- "What's your expected scale at launch?"
- Options: <100 users (early beta), 100-1000 users (public beta), 1000-10000 users (growth phase), 10000+ users (scale)

**Question 5 — Follow-ups:**
Based on answers, ask 1-2 targeted questions. Examples:
- If payments selected: "Subscription billing, one-time payments, or both?"
- If AI/ML selected: "What AI capabilities? (e.g., content generation, recommendations, search)"
- If enterprise audience: "Do you need multi-tenancy or SSO?"

### Phase 2 Gate

Present a structured summary:

```
## Technical Planning Summary

**Tech Stack:** [chosen stack with rationale]
**Integrations:** [list]
**Deployment:** [platform]
**Scale Target:** [expected users]
**Key Technical Decisions:** [from follow-ups]
```

Ask: "Does this technical direction look right? Should I adjust anything before generating the technical documents?"

**Wait for explicit approval.**

### Phase 2 Document Generation

Generate these 4 files in the output directory:

**5. mvp-scope.md**
- SLC (Simple, Lovable, Complete) definition
- MoSCoW prioritization:
  - **Must Have:** Core features for launch (3-5 items)
  - **Should Have:** Important but not blocking (3-5 items)
  - **Could Have:** Nice-to-have (2-3 items)
  - **Won't Have (yet):** Explicitly deferred (2-3 items)
- MVP success criteria
- Estimated development timeline (in weeks, not days)
- Risk assessment for MVP scope

**6. user-stories.md**
- Organized by Epics (3-5 epics for MVP)
- Each epic contains 3-7 user stories
- Format per story:
  - "As a [role], I want [capability] so that [benefit]"
  - Acceptance criteria (Given/When/Then)
  - Priority (Must/Should/Could)
  - Estimated complexity (S/M/L)

**7. tech-stack-decision.md**
- Chosen technologies with version recommendations
- Rationale for each choice
- Alternatives considered and why rejected
- Key libraries and frameworks
- Development environment setup requirements
- CI/CD pipeline recommendations

**8. architecture-overview.md**
- System architecture diagram (described in text/ASCII)
- Component breakdown with responsibilities
- Data flow description
- API design (key endpoints, REST/GraphQL decision)
- Database schema (entities, relationships, key fields)
- Authentication and authorization approach
- Third-party service integrations
- Infrastructure and deployment architecture
```

**Step 2: Verify**

Run: `wc -l plugins/business-discovery/skills/business-discovery/SKILL.md`
Expected: Further increased line count

**Step 3: Commit**

```bash
git add plugins/business-discovery/skills/business-discovery/SKILL.md
git commit -m "feat: add Phase 2 technical planning workflow to SKILL.md"
```

---

### Task 7: Write SKILL.md — Completion and Summary Section

**Files:**
- Modify: `plugins/business-discovery/skills/business-discovery/SKILL.md`

**Step 1: Append the completion section**

Add after Phase 2 Document Generation:

```markdown

## Completion

After all 8 documents are generated, present a final summary:

```
## Project Documentation Complete

**Project:** [name]
**Location:** [output directory]

### Documents Generated:

**Business Documents:**
1. project-charter.md — Vision, goals, success criteria
2. business-model-canvas.md — 9-block business model
3. monetization-strategy.md — Pricing and revenue projections
4. go-to-market-plan.md — Launch and growth strategy

**Technical Documents:**
5. mvp-scope.md — SLC scope with MoSCoW prioritization
6. user-stories.md — Epics and stories with acceptance criteria
7. tech-stack-decision.md — Technology choices with rationale
8. architecture-overview.md — System design and data flow

These documents are designed to be sufficient for a developer to implement
the project. No implementation code has been generated.
```

Ask: "Your project documentation is complete. Would you like to revise any document, or is everything good?"

If the user wants revisions, make targeted edits to the specific document. Do not regenerate from scratch unless asked.

## Tips for High-Quality Output

- **Be specific, not generic.** Every document should reference the user's actual answers, not boilerplate.
- **Revenue projections should be realistic** based on the target MRR, audience size, and pricing model.
- **Tech stack recommendations should match the business context.** A $1K MRR side project doesn't need enterprise infrastructure.
- **MVP scope should be ruthlessly small.** The SLC should be buildable by one developer in 4-8 weeks.
- **User stories should be testable.** Every acceptance criterion should be verifiable.
- **Architecture should match scale expectations.** Don't over-architect for <100 users.
```

**Step 2: Verify complete SKILL.md**

Run: `wc -l plugins/business-discovery/skills/business-discovery/SKILL.md`
Expected: Complete file, ~250-350 lines

**Step 3: Commit**

```bash
git add plugins/business-discovery/skills/business-discovery/SKILL.md
git commit -m "feat: add completion section and quality tips to SKILL.md"
```

---

### Task 8: Add README.md

**Files:**
- Create: `README.md`

**Step 1: Write the README**

```markdown
# Claude Plugins

Personal Claude Code plugin collection.

## Plugins

### business-discovery

Guided business idea discovery and project documentation generator.

**Command:** `/discover [app-name]`

Walks you through two phases:
1. **Business Discovery** — problem, audience, monetization, competitive advantage
2. **Technical Planning** — tech stack, integrations, deployment, architecture

Generates 8 comprehensive documents:
- Project Charter, Business Model Canvas, Monetization Strategy, Go-to-Market Plan
- MVP Scope, User Stories, Tech Stack Decision, Architecture Overview

## Installation

Add this repo as a Claude Code plugin source.

## License

Apache 2.0
```

**Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with plugin overview"
```

---

### Task 9: Final verification

**Step 1: Verify complete directory structure**

Run: `find . -not -path './.git/*' -not -path './.git' | sort`

Expected:
```
.
./.claude-plugin
./.claude-plugin/marketplace.json
./LICENSE
./README.md
./docs
./docs/plans
./docs/plans/2026-03-22-business-discovery-plugin-design.md
./docs/plans/2026-03-22-business-discovery-plugin.md
./plugins
./plugins/business-discovery
./plugins/business-discovery/.claude-plugin
./plugins/business-discovery/.claude-plugin/plugin.json
./plugins/business-discovery/commands
./plugins/business-discovery/commands/discover.md
./plugins/business-discovery/hooks
./plugins/business-discovery/skills
./plugins/business-discovery/skills/business-discovery
./plugins/business-discovery/skills/business-discovery/SKILL.md
```

**Step 2: Verify JSON files are valid**

Run: `python -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "marketplace.json OK"`
Run: `python -m json.tool plugins/business-discovery/.claude-plugin/plugin.json > /dev/null && echo "plugin.json OK"`

**Step 3: Verify SKILL.md frontmatter**

Run: `head -6 plugins/business-discovery/skills/business-discovery/SKILL.md`
Expected: Valid YAML frontmatter starting and ending with `---`

**Step 4: Commit any remaining changes**

```bash
git status
# If any unstaged changes remain, add and commit
```
