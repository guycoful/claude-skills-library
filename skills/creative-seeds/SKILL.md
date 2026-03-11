---
name: creative-seeds
description: Inject creative absurdity to break AI pattern-thinking before creative tasks. Randomly samples surreal micro-stories to spark unconventional thinking. Use before brainstorming, naming, writing, or any task needing fresh perspective.
allowed-tools: Bash
---

# Creative Seeds

Before any creative task, run the script below — it selects 3 random seeds and returns them directly.

## How to Use

Run this to pull 3 random seeds:

```bash
perl -MList::Util=shuffle -e '@l=<>; print join("\n---\n", (shuffle @l)[0..2])' ~/.claude/skills/creative-seeds/seeds.md 2>/dev/null || shuf -n 3 ~/.claude/skills/creative-seeds/seeds.md
```

## Instructions

1. Run the bash command above — it returns 3 randomly selected seed paragraphs
2. Read the output — let the absurdist logic seep in
3. Then proceed with the actual creative task with that mental residue active
4. Do NOT explain the seeds to the user unless asked — just use them internally to shift your thinking frame

The seeds are not prompts. They are cognitive disruptors. They work by contaminating your pattern-matching with nonsense, forcing novel connections.
