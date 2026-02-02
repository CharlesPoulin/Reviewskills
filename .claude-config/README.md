# Claude Code Configuration

This directory contains reusable Claude Code skills and agents for code review.

## Structure

```
.claude-config/
├── README.md
├── skills/
│   └── reviewthis/
│       └── SKILL.md           # /reviewthis skill - Hybrid v3.0 PR code review
└── agents/
    ├── arch-critic.md                  # (opus) - Socratic architectural critic
    ├── architecture-pattern-advisor.md # (sonnet) - Enterprise pattern advisor
    ├── complexity-analyst.md           # (sonnet) - Complexity and maintainability
    ├── solid-principles-analyzer.md    # (sonnet) - SOLID + TDA enforcement
    ├── sre-security-auditor.md         # (haiku) - Security and reliability
    └── test-gap-analyst.md             # (haiku) - Test coverage analysis
```

## Installation

Copy the skills and agents to your global `~/.claude/` directory:

### Windows (Git Bash / WSL)
```bash
# Copy skills (preserving directory structure)
cp -r .claude-config/skills/* ~/.claude/skills/

# Copy agents
cp -r .claude-config/agents/* ~/.claude/agents/
```

### Windows (PowerShell)
```powershell
# Create directories
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\agents"

# Copy skills and agents
Copy-Item -Path ".\.claude-config\skills\*" -Destination "$env:USERPROFILE\.claude\skills\" -Recurse -Force
Copy-Item -Path ".\.claude-config\agents\*" -Destination "$env:USERPROFILE\.claude\agents\" -Recurse -Force
```

### Linux / macOS
```bash
cp -r .claude-config/skills/* ~/.claude/skills/
cp -r .claude-config/agents/* ~/.claude/agents/
```

## Usage

After installation, **restart Claude Code**. The skill will be available:

```
/reviewthis <pr_number>
```

### Example
```
/reviewthis 42
```

This will:
1. Check PR eligibility (not draft, not closed, CI passing)
2. Spawn 6 parallel review agents with different focuses
3. A judge agent filters false positives (score < 80 discarded)
4. Post a formatted comment on the PR

## Agent Models

| Agent | Model | Focus |
|-------|-------|-------|
| arch-critic | opus | Deep logic, history, regression risk |
| architecture-pattern-advisor | sonnet | Clean Architecture, patterns |
| complexity-analyst | sonnet | Cognitive complexity, maintainability |
| solid-principles-analyzer | sonnet | SOLID + TDA compliance |
| sre-security-auditor | haiku | Security, reliability, bugs |
| test-gap-analyst | haiku | Test coverage gaps |

## Skill Format

Skills must follow this structure:
```
skills/
└── <skill-name>/
    └── SKILL.md    # Required - contains frontmatter and instructions
```

The `SKILL.md` file needs YAML frontmatter:
```yaml
---
name: my-skill
description: What the skill does
---

# Instructions here...
```

## Customization

Edit the agent files to customize:
- `model:` - Change the model (opus, sonnet, haiku)
- `color:` - Terminal output color
- Instructions in the markdown body
