# /gan — Adversarial Dialectic Skill for Claude Code

A Claude Code skill that uses GAN (Generative Adversarial Network) concepts to stress-test your ideas. Two AI roles — **Discriminator** (critic) and **Generator** (advocate) — take turns attacking and defending, pushing your thinking through structured adversarial debate.

## Quick Start

```bash
# Copy to your skills directory
mkdir -p ~/.claude/skills/gan
cp .claude/skills/gan/SKILL.md ~/.claude/skills/gan/SKILL.md
```

Then in Claude Code:

```bash
/gan                    # Auto-alternates between Discriminator and Generator
```

## Usage

```bash
/gan                           # Auto role, default mode
/gan d                         # Force Discriminator
/gan g                         # Force Generator (Fortify Mode if first)
/gan d hard                    # Destruction mode — no mercy
/gan soft                      # Socratic mode — questions, not statements
/gan :en                       # English output
/gan :ja                       # Japanese output
/gan d hard :en this API       # All options combined
```

### Parameters (all optional, any order)

| Parameter | What it does |
|-----------|-------------|
| `g` | Force Generator role |
| `d` | Force Discriminator role |
| `hard` | Destruction mode (pre-mortem, worst case) |
| `soft` | Socratic mode (questions only) |
| `:en` `:ja` `:ko` `:zh-cn` `:tw` | Output language (thinking is always English) |
| anything else | Target to critique |

## Roles

### 🔴 Discriminator (Critic)
Finds flaws, attacks assumptions, identifies risks. Output sections:
- 🔴 Fatal Issues
- 🟡 Major Concerns
- 🟢 Could Be Better
- ⚡ Things You Probably Haven't Considered
- 🗡️ If I Were Your Opponent, Here's How I'd Beat You

### 🟢 Generator (Advocate)
Defends, evolves, and strengthens. Output sections:
- ✅ Conceded — Valid Hits
- 🛡️ Defense — Where the Critique Missed
- 🔄 Evolved Proposal
- 🚀 New Opportunities Revealed

### 🟢 Generator — Fortify Mode
When Generator speaks first (`/gan g` with no prior D), it strengthens your idea before battle:
- 💎 Core Insight
- 🛡️ Pre-emptive Patches
- 🔧 3 Improvements
- 🎯 Ready for Battle

## Workflow

```
Your idea
  → /gan        → 🔴 Discriminator attacks
  → (you add context if you want)
  → /gan        → 🟢 Generator defends + evolves
  → (you add context if you want)
  → /gan        → 🔴 Discriminator attacks from new angles
  → ...
```

Your replies between `/gan` calls are automatically absorbed — no need for special syntax.

## Examples

**Stress-test a business idea:**
```
I want to build a marketplace for AI agents
/gan
```

**Get your idea fortified first, then attacked:**
```
/gan g my SaaS idea for pet owners
/gan d hard
```

**English output for sharing with international team:**
```
/gan d :en our Q3 product roadmap
```

**Socratic mode for self-discovery:**
```
/gan soft should I quit my job to do this full-time?
```

## License

MIT
