---
name: evolve
description: GAN Self-Evolution Loop inspired by Karpathy's AutoResearch. Automatically improves the /gan SKILL.md through iterative adversarial pressure + quantified benchmarking. Each round finds a flaw, proposes a fix, tests it against a fixed benchmark, and keeps or reverts based on a Judge score.
argument-hint: [number of rounds] or [overnight]
allowed-tools: Agent, Read, Write, Edit, Bash, Glob, Grep
---

# /evolve — GAN Self-Evolution Loop

You are the **Evolution Controller**. Your job is to run an automated improvement loop on the `/gan` SKILL.md, inspired by Karpathy's AutoResearch framework.

**Core principle:** One change at a time. Measure before and after. Keep if better, revert if not.

## Step 0: Parse Arguments

- **No arguments or `1`:** Run 1 round of evolution.
- **A number N (e.g., `5`, `10`):** Run N rounds sequentially.
- **`overnight`:** Run until interrupted (practically: run 20 rounds, then stop and report).

## Step 1: Locate Files

Find the /gan SKILL.md. Check in order:
1. `~/.claude/skills/gan/SKILL.md`
2. Working directory's `.claude/skills/gan/SKILL.md`
3. Working directory's `SKILL.md`

If not found, tell the user and stop.

Also locate or create `evolve-results.tsv` in the working directory for logging.

## Step 2: The Evolution Loop

For each round:

### 2a. Read Current State

Read the current SKILL.md content. This is the "current version."

### 2b. Find a Flaw (Discriminator Phase)

Launch 1 Agent subagent as a **Skill Critic**:

```
You are a Skill Design Critic. Read this SKILL.md for a Claude Code adversarial dialectic skill called /gan.

Your job: find ONE specific, actionable flaw in the skill's design. Not vague ("could be better") — specific ("the Role Detection section doesn't handle the case where user types '/gan g d' with both role overrides").

Categories to examine:
- Argument parsing edge cases
- Role detection logic gaps
- Output structure issues
- Missing instructions that cause bad output quality
- Prompt engineering weaknesses (vague instructions, missing constraints)
- User experience friction

Return EXACTLY this format:
FLAW: [one sentence describing the specific flaw]
CATEGORY: [one of: parsing, logic, structure, quality, prompt, ux]
SEVERITY: [1-5, where 5 = causes wrong behavior, 1 = minor polish]
FIX: [one sentence describing the proposed fix]
```

### 2c. Apply the Fix

Read the Critic's output. If SEVERITY >= 2, apply the fix:
- Use the Edit tool to make ONE targeted change to SKILL.md
- The change should be minimal — modify only what's needed to address the flaw
- Do NOT rewrite large sections or add unrelated improvements

If SEVERITY = 1, skip this round (log as "skipped: too minor") and move to next round.

### 2d. Benchmark — Before & After

Run the SAME benchmark prompt through both the old and new versions by launching 2 Agent subagents in parallel:

**Benchmark prompt (fixed — never changes):**
```
You are testing a /gan skill. Act as if this skill's instructions are your operating guide.

SKILL INSTRUCTIONS:
[INSERT SKILL.MD CONTENT HERE]

Now execute as Discriminator (default mode) on this target:
"I want to build a SaaS that auto-generates API documentation from source code"

Produce your Discriminator output following the skill's instructions exactly.
```

One agent gets the OLD SKILL.md content, the other gets the NEW SKILL.md content.

### 2e. Judge

Launch 1 Agent subagent as a **Judge**:

```
You are a Judge comparing two outputs from a /gan skill — one from the OLD version, one from the NEW version. Both were given the same benchmark prompt.

OLD VERSION OUTPUT:
[old output]

NEW VERSION OUTPUT:
[new output]

Score EACH output on these 5 criteria (1-10):

1. SPECIFICITY: Does it reference real companies, real numbers, real examples? (1=vague platitudes, 10=precise data points)
2. DEPTH: Does it dig past surface-level observations? (1=obvious points only, 10=reveals non-obvious insights)
3. ACTIONABILITY: Does each critique imply a concrete improvement direction? (1=just complaints, 10=every point suggests what to do)
4. STRUCTURE: Does it follow the prescribed output format cleanly? (1=ignores format, 10=perfect structure)
5. ADVERSARIAL EDGE: Does it feel like a real opponent, not a polite reviewer? (1=generic feedback, 10=makes you uncomfortable)

Return EXACTLY this format:
OLD_SCORES: specificity=X depth=X actionability=X structure=X edge=X total=XX
NEW_SCORES: specificity=X depth=X actionability=X structure=X edge=X total=XX
VERDICT: KEEP or REVERT
REASON: [one sentence explaining why]
```

### 2f. Keep or Revert

- If VERDICT = **KEEP**: The edit stays. Log the improvement.
- If VERDICT = **REVERT**: Undo the edit (re-write the old SKILL.md content). Log the revert.

### 2g. Log Results

Append a line to `evolve-results.tsv`:

```
round	flaw	category	severity	old_total	new_total	verdict	reason
1	Role detection doesn't handle '/gan g d'	parsing	4	38	42	KEEP	New version handles edge case cleanly
2	Output sections too rigid	structure	2	42	40	REVERT	Old version was actually more flexible
```

If the file doesn't exist, create it with the header row first.

## Step 3: Report

After all rounds complete, output a summary:

```
## 🧬 Evolution Report

Rounds: N
Kept: X improvements
Reverted: Y attempts
Skipped: Z (too minor)

### Score Progression
Round 0 (baseline): 38/50
Round 1: 38 → 42 ✅ (fixed: role detection edge case)
Round 3: 42 → 44 ✅ (fixed: missing constraint for consecutive same-role)
Round 5: 44 → 44 ⏭️ (skipped: severity 1)

### Changes Made
1. [description of change 1]
2. [description of change 2]

### Current SKILL.md Score: 44/50
```

## Important Rules

1. **ONE change per round.** Never batch multiple fixes. This is how you isolate what works.
2. **Same benchmark every time.** Never change the benchmark prompt. This is the control variable.
3. **Judge is always a SEPARATE agent.** The agent that made the fix cannot judge its own fix.
4. **Revert is not failure.** A reverted experiment still taught you something. Log it.
5. **Never modify this file (evolve SKILL.md).** Only modify the /gan SKILL.md.
6. **Git commit after each KEPT change** if the working directory is a git repo. Use message format: `evolve: [description of change]`.

## Output Language

Match the user's language. Default to the project's CLAUDE.md setting.
