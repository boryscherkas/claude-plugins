# Business Discovery Plugin Design

## Overview

A Claude Code plugin that guides users through business idea discovery and generates comprehensive project documentation — without initiating any implementation. The plugin lives in a marketplace-style repo that can host multiple plugins.

## Repository Structure

```
claude-plugins/                          # Marketplace root
├── .claude-plugin/
│   └── marketplace.json                 # Lists all plugins
├── plugins/
│   └── business-discovery/              # This plugin
│       ├── .claude-plugin/
│       │   └── plugin.json              # Plugin metadata
│       ├── commands/
│       │   └── discover.md              # /discover slash command
│       ├── skills/
│       │   └── business-discovery/
│       │       └── SKILL.md             # Full guided workflow
│       └── hooks/                       # Reserved for future use
├── LICENSE
└── README.md
```

## Invocation

- **Slash command only:** `/discover [app-name]`
- `discover.md` invokes the `business-discovery` skill
- `disable-model-invocation: true` — user must explicitly start it

## Workflow: Two-Phase Guided Conversation

### Phase 1: Business Discovery

**Questions (asked one at a time, multiple choice preferred):**

1. "What problem are you solving?" (open-ended)
2. "Who is your target audience?" (B2C consumers, B2B SMBs, B2B Enterprise, Developers, Creators/Freelancers)
3. "What's your monetization model?" (SaaS subscription, Freemium, Marketplace/commission, One-time purchase, Usage-based)
4. "What's your competitive advantage?" (open-ended)
5. "What's your MRR target?" ($1K, $5K, $10K, $50K+)
6. Follow-up questions based on answers

**Documents generated after user approval:**

| File | Contents |
|------|----------|
| `project-charter.md` | Vision, goals, success criteria, stakeholders, constraints |
| `business-model-canvas.md` | All 9 BMC blocks filled based on answers |
| `monetization-strategy.md` | Pricing tiers, revenue projections, competitive analysis |
| `go-to-market-plan.md` | Launch strategy, marketing channels, growth metrics |

### Phase 2: Technical Planning

**Questions:**

1. "Where should I create the project files?" (open-ended, user specifies path)
2. Tech stack preference (Next.js + Node.js, Next.js + C#, React + Node.js, Let Claude decide)
3. "What integrations do you need?" (multi-select: Auth, Payments, Email, Analytics, Database, Storage, AI/ML)
4. "What's your deployment preference?" (Vercel, AWS, Azure, Self-hosted)
5. Follow-up questions based on answers

**Documents generated after user approval:**

| File | Contents |
|------|----------|
| `mvp-scope.md` | SLC definition, MoSCoW prioritization, in/out of MVP |
| `user-stories.md` | Epics and stories with acceptance criteria |
| `tech-stack-decision.md` | Chosen technologies with rationale, alternatives considered |
| `architecture-overview.md` | System components, data flow, API design, DB schema |

### Phase Gates

- Each phase ends with a summary presented to the user
- User must approve before documents are generated
- User reviews generated docs before proceeding to next phase
- No implementation code is ever generated

## File Formats

### marketplace.json

```json
{
  "name": "claude-plugins",
  "owner": { "name": "borys" },
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

### plugin.json

```json
{
  "name": "business-discovery",
  "description": "Discover business ideas and generate comprehensive project documentation",
  "version": "1.0.0",
  "author": { "name": "borys" }
}
```

### discover.md

Frontmatter with `disable-model-invocation: true` and `argument-hint: "[app-name]"`. Body invokes the business-discovery skill.

### SKILL.md

Single monolithic skill containing:
- Frontmatter: name, description, tools list, disable-model-invocation
- Workflow instructions with phase structure
- Question templates for each phase
- Document generation instructions with content guidelines
- Phase gate rules (approve before generating, review before next phase)

## Architecture Decision: Single Monolithic Skill

Chosen over multi-skill orchestrator and template-based approaches because:
- The workflow is inherently conversational and sequential
- All context stays in one conversation flow
- Simpler to maintain and extend
- Matches how the best existing plugins (superpowers) structure their skills

## Key Constraints

- Documentation only — no implementation code generated
- Questions asked one at a time
- Multiple choice preferred over open-ended when possible
- Output directory is user-specified each time
- Tech stack recommendations include Next.js + Node.js or C# backend options
