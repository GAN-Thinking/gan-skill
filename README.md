# /gan — Adversarial Dialectic Skill for Claude Code

A Claude Code skill that uses GAN (Generative Adversarial Network) concepts to stress-test your ideas. Two AI roles — **Discriminator** (critic) and **Generator** (advocate) — take turns attacking and defending, pushing your thinking through structured adversarial debate.

## How is this different from Pros & Cons?

**In one sentence:** Pros & Cons makes AI a commentator — standing on the sidelines giving opinions. GAN Thinking makes AI both the player and the coach — fighting on the field, reviewing its own performance, then fighting a better round.

This is why GAN Thinking has unique value as a skill — it's not a fancy wrapper around Pros & Cons. The underlying mechanism is fundamentally different.

### How GAN Thinking works internally

AI splits into two distinct roles — Generator and Discriminator. This creates an **iterative adversarial loop**: G produces, D scores, G improves based on feedback, D scores again. The key is that D doesn't just say "good" or "bad" — it provides a **gradient signal** explaining *why* something isn't good enough, giving G a direction to improve.

This means AI is performing multiple rounds of self-correction, and each round has directionality. It's not random retry — it's targeted optimization along the weaknesses D identified.

### How Pros & Cons works internally

AI is asked to list positives and negatives for a proposal simultaneously. This is a **single static evaluation** — stand in front of a plan, look left, look right, list pros and cons, done.

No iteration, no adversarial pressure, no role separation. AI uses the same perspective to see both sides at once.

### The fundamental differences

**1. Time structure.**
Pros & Cons is a snapshot — one-shot output. GAN Thinking is a time series — with rounds and evolution. The same question after three GAN iterations produces dramatically better answers than a one-shot Pros & Cons analysis, because the GAN version has been pressure-tested.

**2. Cognitive depth.**
Pros & Cons tends to stay at the surface layer. AI naturally lists obvious pros and cons, then stops. GAN Thinking forces AI to dig into the second and third layers — because D says "G already addressed that issue, give me deeper vulnerabilities," surfacing blind spots that wouldn't otherwise be found.

**3. Purity of stance.**
Pros & Cons asks AI to think about good and bad using the same brain simultaneously, creating a "self-balancing" tendency — AI unconsciously makes the number of pros and cons roughly equal, the severity roughly matched, giving you an analysis that looks balanced but is actually mediocre. GAN Thinking completely separates the two stances. G goes all-in on making the best case. D goes all-in on finding the most fatal problems. Neither needs to accommodate the other. The tension from this separation is where the real signal comes from.

**4. Nature of the output.**
Pros & Cons produces a list. GAN Thinking produces a **battle-tested proposal**. The former helps you "see the situation clearly." The latter helps you "produce something better." One is an analysis tool. The other is a creation tool.

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
