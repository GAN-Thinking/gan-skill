# /gan — Make AI Argue With Itself to Stress-Test Your Ideas

A Claude Code skill. Two AI roles fight each other so your ideas get stronger.

Dead simple to use. Just type `/gan` after any idea — the quality of thinking you get back is dramatically better than asking AI to "give me pros and cons."

## See it in action

**You type:**
```
I want to build a marketplace for AI agents to trade services
/gan
```

**🔴 Discriminator fires back:**
> **Steel Man:** A marketplace connecting AI agents could become the Upwork of the AI era — real demand exists as agents proliferate.
>
> **🔴 Fatal Issues**
> Who pays? Agents don't have wallets. Their owners do — but owners won't let agents autonomously transact with unknown agents. You need trust infrastructure that doesn't exist yet.
>
> **🗡️ If I Were Your Opponent**
> I'd skip the marketplace entirely. I'd build an open protocol — like MCP but for agent-to-agent discovery. No platform fees, no cold start. Your marketplace becomes irrelevant on day one.

---

**You reply:** "Good point, but I have existing relationships with 50 AI dev teams"

**You type:** `/gan`

---

**🟢 Generator absorbs the hit and evolves:**
> **✅ Conceded:** Trust infrastructure is genuinely missing. Fair hit.
>
> **🛡️ Defense:** The "open protocol" attack ignores that protocols don't solve discovery quality. npm is open, but developers still use curated lists because quantity ≠ quality.
>
> **🔄 Evolved Proposal:** Don't build a marketplace. Build a curated referral network seeded with those 50 teams. Private, invite-only, trust built on real relationships...

**Each round gets sharper.** Discriminator can't repeat old attacks. Generator must evolve, not just defend.

## Install

Three ways, pick one:

**Option A — One-line download (fastest):**
```bash
mkdir -p ~/.claude/skills/gan && curl -o ~/.claude/skills/gan/SKILL.md https://raw.githubusercontent.com/GAN-Thinking/gan-skill/main/SKILL.md
```

**Option B — Clone and copy (for future updates):**
```bash
git clone https://github.com/GAN-Thinking/gan-skill.git ~/gan-skill
mkdir -p ~/.claude/skills/gan
cp ~/gan-skill/SKILL.md ~/.claude/skills/gan/SKILL.md
# To update later: cd ~/gan-skill && git pull && cp SKILL.md ~/.claude/skills/gan/SKILL.md
```

**Option C — Manual download:**
1. Download [`SKILL.md`](SKILL.md) from this repo
2. Put it in `~/.claude/skills/gan/SKILL.md`

Then in Claude Code, just type:
```
/gan your idea here
```

## Usage

```bash
/gan                        # Auto-alternates Discriminator ↔ Generator
/gan d                      # Force Discriminator
/gan g                      # Force Generator (Fortify Mode if first)
/gan d hard                 # Destruction mode — assumes failure, finds cause of death
/gan soft                   # Socratic mode — sharp questions instead of statements
/gan :en                    # English output (also :ja :ko :zh-cn :tw)
/gan d hard :en this API    # Combine everything
```

All parameters are optional, any order. Your free-text replies between `/gan` calls are automatically absorbed as context.

## How is this different from Pros & Cons?

Pros & Cons asks one brain to list good and bad simultaneously — you get a balanced-looking but shallow list.

GAN Thinking splits AI into two adversaries. Generator goes all-in building the best case. Discriminator goes all-in destroying it. Neither pulls punches. Then Generator rebuilds from the wreckage, stronger.

| | Pros & Cons | /gan |
|---|---|---|
| Rounds | 1 (snapshot) | Unlimited (iterative) |
| Depth | Surface-level obvious points | Forces 2nd and 3rd layer — "G already addressed that, dig deeper" |
| Stance | Same brain, self-balancing | Two separated minds, maximum tension |
| Output | A list | A battle-tested proposal |

One is an analysis tool. The other is a creation tool.

## Contributing

Found a bug? Have an idea? [Open an issue](https://github.com/GAN-Thinking/gan-skill/issues) or submit a PR.

If you find this useful, a ⭐ helps others discover it.

## License

[MIT](LICENSE)
