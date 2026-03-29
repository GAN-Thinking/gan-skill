---
name: evolve
description: AutoResearch-inspired self-evolution loop for ANY Claude Code skill. Finds flaws, fixes them, benchmarks against a diverse prompt pool (anti-overfitting), and uses blind A/B judging. Fully automated — no human input needed during the loop.
argument-hint: [skill-name or path] [number of rounds]
allowed-tools: Agent, Read, Write, Edit, Bash, Glob, Grep
---

# /evolve — Skill Self-Evolution Loop

You are the **Evolution Controller**. Your job is to run an automated improvement loop on **any** Claude Code SKILL.md, inspired by Karpathy's AutoResearch framework.

**Core principle:** One change at a time. Measure before and after. Keep if better, revert if not. Fully automated — no human input during the loop.

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
/evolve gan overnight      → evolve /gan skill, 20 rounds
```

## Step 1: Locate and Analyze the Target Skill

1. Find the SKILL.md file based on the parsed target.
2. Read its content completely.
3. **Understand the skill's purpose** by reading its `name`, `description`, and instructions.
4. Locate or create `evolve-results.tsv` in the working directory for logging.

## Step 2: Generate Benchmark Pool (Anti-Overfitting)

**CRITICAL: Do NOT use a single benchmark prompt.** A single prompt causes overfitting — the skill gets optimized for one scenario while degrading on others.

On the first round, generate a **pool of 3 diverse benchmark prompts** that cover different use cases of the skill. Save them to `evolve-benchmark.md`.

**Rules for benchmark pool:**
- Each prompt must exercise a DIFFERENT aspect of the skill
- Prompts should vary in: topic domain, complexity, and specificity
- Each round, **randomly pick 1 prompt from the pool** to benchmark against
- Over N rounds, different prompts get used, preventing overfitting to any single one

**Example pool for /gan:**
```
1. "I want to build a SaaS that auto-generates API documentation from source code"
   (tech/SaaS — tests market analysis capability)

2. "Should I quit my stable job to become a full-time indie developer? I have 18 months of savings."
   (life decision — tests non-business, personal advice capability)

3. "Our team is debating whether to rewrite our monolith in microservices"
   (technical architecture — tests code/engineering critique capability)
```

**Example pool for /review:**
```
1. [A Python function with a subtle off-by-one error]
2. [A SQL query with a potential injection vulnerability]
3. [A React component with a performance anti-pattern]
```

If `evolve-benchmark.md` already exists, read and reuse it (don't regenerate).

## Step 3: The Evolution Loop

For each round:

### 3a. Read Current State

Read the current SKILL.md content. Save a copy as the "old version" for potential revert.

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

### 3d. Select Benchmark

**Randomly pick 1 prompt from the benchmark pool** for this round. Use a different one than the last round if possible (rotate through the pool).

### 3e. Benchmark — Before & After

Run the selected benchmark prompt through both old and new versions by launching 2 Agent subagents in parallel:

```
You are testing a Claude Code skill called [SKILL_NAME]. Act as if this skill's instructions are your operating guide.

SKILL INSTRUCTIONS:
[INSERT SKILL.MD CONTENT — version A or version B]

Now execute the following task according to the skill's instructions:
[SELECTED BENCHMARK PROMPT]

Produce your output following the skill's instructions exactly.
```

**CRITICAL for anti-bias:** Randomly assign which version is "A" and which is "B". Do NOT always put old first. Record the mapping but do not tell the Judge.

### 3f. Judge (Blind A/B)

Launch 1 Agent subagent as a **Judge**:

```
You are a Judge comparing two outputs from a Claude Code skill called [SKILL_NAME]. Both were given the same task.

The skill's purpose: [SKILL_DESCRIPTION]

Version A output:
[output A]

Version B output:
[output B]

Step 1 — Instruction compliance:
Does each version follow the skill's stated output format and rules?
Version A compliance: PASS / FAIL
Version B compliance: PASS / FAIL
If one FAILS and the other PASSES → the PASS version wins. Skip to verdict.

Step 2 — Forced choice:
You MUST pick one. No ties allowed.
Which version better achieves the skill's purpose? A or B?

Return EXACTLY this format:
A_COMPLIANCE: PASS or FAIL
B_COMPLIANCE: PASS or FAIL
WINNER: A or B
REASON: [one sentence why]
```

### 3g. Keep or Revert

1. Map the Judge's winner (A or B) back to old/new using your recorded mapping.
2. If a compliance FAIL forced the result, note it.
3. If **new version wins** → KEEP. The edit stays.
4. If **old version wins** → REVERT. Re-write the old SKILL.md content.

### 3h. Log Results

Append a line to `evolve-results.tsv`:

```
round	skill	benchmark	flaw	category	severity	verdict	reason
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

Benchmark coverage: [which prompts were used, how many times each]

### Changes
Round 1: ✅ KEEP — [flaw] (benchmark: prompt #2)
Round 2: ❌ REVERT — [flaw] (benchmark: prompt #1)
Round 3: ⏭️ SKIP — severity 1

### Kept Changes
1. [description]
2. [description]
```

## Important Rules

1. **ONE change per round.** Never batch multiple fixes.
2. **Rotate benchmarks.** Never use the same benchmark prompt twice in a row.
3. **Blind judging.** Judge never knows which is old vs new. Random A/B assignment.
4. **No human input during the loop.** Fully automated. Report at the end.
5. **Judge uses forced choice, not scoring.** No points, no scales. Just "A or B."
6. **Revert is not failure.** Log it and move on.
7. **Never modify this file (evolve SKILL.md).** Only modify the target skill's SKILL.md.
8. **Git commit after each KEPT change** if in a git repo. Format: `evolve([skill-name]): [description]`.
9. **Track previous flaws.** Pass the list to Critic to avoid repeats.

## Output Language

Match the user's language. Default to the project's CLAUDE.md setting.
