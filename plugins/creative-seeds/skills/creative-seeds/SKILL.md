---
name: creative-seeds
description: Inject creative absurdity to break AI pattern-thinking before creative tasks. Randomly samples surreal micro-stories to spark unconventional thinking. Use before brainstorming, naming, writing, or any task needing fresh perspective.
allowed-tools: Bash, Read
---

# Creative Seeds

Before any creative task, randomly sample 3 seeds from the collection to disrupt default patterns.

## How to Use

Run this to pull 3 random seeds:

```bash
perl -MList::Util=shuffle -e '@l=<>; print join("\n---\n", (shuffle @l)[0..2])' ~/.claude/skills/creative-seeds/seeds.md 2>/dev/null || shuf -n 3 ~/.claude/skills/creative-seeds/seeds.md
```

Or read the full seeds file and randomly pick 3 entries.

## When to Invoke

- Before brainstorming names, concepts, taglines
- Before writing anything creative
- When output feels generic or predictable
- When the user asks for fresh / unexpected ideas
- Before any task where "thinking differently" matters

## Instructions

1. Read `~/.claude/skills/creative-seeds/seeds.md`
2. Pick 3 random seeds (by number, use current timestamp mod 50 for randomness)
3. Read them — let the absurdist logic seep in
4. Then proceed with the actual creative task with that mental residue active
5. Do NOT explain the seeds to the user unless asked — just use them internally to shift your thinking frame

The seeds are not prompts. They are cognitive disruptors. They work by contaminating your pattern-matching with nonsense, forcing novel connections.
