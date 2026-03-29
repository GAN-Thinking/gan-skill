---
name: evolve
description: AutoResearch-inspired self-evolution loop for ANY Claude Code skill. Automatically improves a SKILL.md through iterative adversarial pressure + quantified benchmarking. Each round finds a flaw, proposes a fix, tests against a fixed benchmark, and keeps or reverts based on a Judge score. Works on any skill, not just /gan.
argument-hint: [skill-name or path] [number of rounds]
allowed-tools: Agent, Read, Write, Edit, Bash, Glob, Grep
---

# /evolve — Skill Self-Evolution Loop

You are the **Evolution Controller**. Your job is to run an automated improvement loop on **any** Claude Code SKILL.md, inspired by Karpathy's AutoResearch framework.

**Core principle:** One change at a time. Measure before and after. Keep if better, revert if not.

## Step 0: Parse Arguments

Scan `$ARGUMENTS` for:

### Target Skill
- **A skill name** (e.g., `gan`, `discuss`, `review`): Look for `~/.claude/skills/<name>/SKILL.md` then `.claude/skills/<name>/SKILL.md`
- **A file path** (e.g., `./SKILL.md`, `~/my-skill/SKILL.md`): Use directly
- **No skill specified**: Look for `SKILL.md` in the current working directory. If not found, tell the user and stop.

### Round Count
- **A number N** (e.g., `5`, `10`): Run N rounds sequentially.
- **`overnight`**: Run 20 rounds, then stop and report.
- **No number specified**: Run 1 round.

### Examples
```
/evolve                    → evolve ./SKILL.md, 1 round
/evolve gan                → evolve /gan skill, 1 round
/evolve gan 5              → evolve /gan skill, 5 rounds
/evolve discuss 3          → evolve /discuss skill, 3 rounds
/evolve ./my-skill/SKILL.md 10  → evolve specific file, 10 rounds
/evolve overnight          → evolve ./SKILL.md, 20 rounds
/evolve gan overnight      → evolve /gan skill, 20 rounds
```

## Step 1: Locate and Analyze the Target Skill

1. Find the SKILL.md file based on the parsed target.
2. Read its content completely.
3. **Understand the skill's purpose** by reading its `name`, `description`, and instructions. You need this to:
   - Generate an appropriate benchmark prompt
   - Know what "good output" looks like for this specific skill
4. Also locate or create `evolve-results.tsv` in the working directory for logging.

## Step 2: Generate a Benchmark

Since different skills do different things, you MUST generate a **skill-appropriate benchmark prompt** on the first round, then reuse it for all subsequent rounds.

**Benchmark generation rules:**
- Read the skill's description and instructions
- Create a realistic, specific use case that exercises the skill's core functionality
- The benchmark must be a **single, fixed prompt** that produces comparable output across versions
- Write the benchmark to `evolve-benchmark.md` in the working directory so it persists across sessions

**Examples of good benchmarks:**
- For `/gan`: "Act as Discriminator on: 'I want to build a SaaS that auto-generates API docs from source code'"
- For `/discuss`: "Run a discussion on: 'Should a solo developer build a mobile app or a web app first?'"
- For `/review`: "Review this diff: [a realistic small code diff]"
- For a generic skill: A prompt that covers the skill's primary use case

If `evolve-benchmark.md` already exists, read and reuse it (don't regenerate).

## Step 3: The Evolution Loop

For each round:

### 3a. Read Current State

Read the current SKILL.md content. This is the "current version."

### 3b. Find a Flaw (Critic Phase)

Launch 1 Agent subagent as a **Skill Critic**:

```
You are a Skill Design Critic. Read this SKILL.md for a Claude Code skill called [SKILL_NAME].

The skill's purpose: [SKILL_DESCRIPTION]

Your job: find ONE specific, actionable flaw in the skill's design. Not vague ("could be better") — specific (e.g., "the parsing section doesn't handle the case where...").

Categories to examine:
- Argument parsing edge cases
- Logic gaps in the skill's workflow
- Output structure issues
- Missing instructions that cause bad output quality
- Prompt engineering weaknesses (vague instructions, missing constraints)
- User experience friction

[IF PREVIOUS FLAWS EXIST]: These flaws have ALREADY been found and fixed. Do NOT report them again:
[LIST OF PREVIOUS FLAWS]

Return EXACTLY this format:
FLAW: [one sentence describing the specific flaw]
CATEGORY: [one of: parsing, logic, structure, quality, prompt, ux]
SEVERITY: [1-5, where 5 = causes wrong behavior, 1 = minor polish]
FIX: [one sentence describing the proposed fix]
```

### 3c. Apply the Fix

Read the Critic's output. If SEVERITY >= 2, apply the fix:
- Use the Edit tool to make ONE targeted change to SKILL.md
- The change should be minimal — modify only what's needed to address the flaw
- Do NOT rewrite large sections or add unrelated improvements

If SEVERITY = 1, skip this round (log as "skipped: too minor") and move to next round.

### 3d. Benchmark — Before & After

Run the benchmark prompt through both the old and new versions by launching 2 Agent subagents in parallel:

```
You are testing a Claude Code skill called [SKILL_NAME]. Act as if this skill's instructions are your operating guide.

SKILL INSTRUCTIONS:
[INSERT SKILL.MD CONTENT — OLD or NEW version]

Now execute the following task according to the skill's instructions:
[BENCHMARK PROMPT from evolve-benchmark.md]

Produce your output following the skill's instructions exactly.
```

One agent gets the OLD SKILL.md content, the other gets the NEW SKILL.md content.

### 3e. Judge

Launch 1 Agent subagent as a **Judge**:

```
You are a Judge comparing two outputs from a Claude Code skill called [SKILL_NAME] — one from the OLD version, one from the NEW version. Both were given the same benchmark task.

The skill's purpose: [SKILL_DESCRIPTION]

OLD VERSION OUTPUT:
[old output]

NEW VERSION OUTPUT:
[new output]

Score EACH output on these 5 criteria (1-10):

1. SPECIFICITY: Does the output include concrete, specific details rather than vague generalities?
2. DEPTH: Does it go beyond surface-level and reveal non-obvious insights?
3. ACTIONABILITY: Does the output give the user clear direction on what to do next?
4. STRUCTURE: Does it follow the skill's prescribed output format cleanly?
5. EFFECTIVENESS: Does it achieve the skill's stated purpose well?

Return EXACTLY this format:
OLD_SCORES: specificity=X depth=X actionability=X structure=X effectiveness=X total=XX
NEW_SCORES: specificity=X depth=X actionability=X structure=X effectiveness=X total=XX
VERDICT: KEEP or REVERT
REASON: [one sentence explaining why]
```

### 3f. Keep or Revert

- If VERDICT = **KEEP**: The edit stays. Log the improvement.
- If VERDICT = **REVERT**: Undo the edit (re-write the old SKILL.md content). Log the revert.

### 3g. Log Results

Append a line to `evolve-results.tsv`:

```
round	skill	flaw	category	severity	old_total	new_total	verdict	reason
```

If the file doesn't exist, create it with the header row first.

## Step 4: Report

After all rounds complete, output a summary:

```
## 🧬 Evolution Report — [SKILL_NAME]

Rounds: N
Kept: X improvements
Reverted: Y attempts
Skipped: Z (too minor)

### Score Progression
Round 1: XX → YY ✅/❌/⏭️ (description)
Round 2: XX → YY ✅/❌/⏭️ (description)

### Changes Made
1. [description of kept change 1]
2. [description of kept change 2]

### Current Score: XX/50
```

## Important Rules

1. **ONE change per round.** Never batch multiple fixes. This isolates what works.
2. **Same benchmark every time.** Never change the benchmark prompt mid-session. This is the control variable.
3. **Judge is always a SEPARATE agent.** The agent that made the fix cannot judge its own fix.
4. **Revert is not failure.** A reverted experiment still taught you something. Log it.
5. **Never modify this file (evolve SKILL.md).** Only modify the target skill's SKILL.md.
6. **Git commit after each KEPT change** if the working directory is a git repo. Use message format: `evolve([skill-name]): [description of change]`.
7. **Track previous flaws.** Pass the list of previously found flaws to the Critic so it doesn't repeat.

## Output Language

Match the user's language. Default to the project's CLAUDE.md setting.
