---
name: claudability-analyzer
description: "Analyzes professions/jobs for Claude Code automation opportunities. Triggers: 'how can Claude help me', 'what can Claude do for', 'I'm a [profession]', 'help me as a [job]', 'I work as', describing their work + asking about Claude. Use whenever user mentions their profession/role and wants to discover what Claude can automate."
disable-model-invocation: false
user-invocable: true
argument-hint: "[profession or task description]"
allowed-tools: WebSearch, WebFetch, Read, Grep, Glob, Write, Skill
---

# Claudability Analyzer

Transform any profession/workflow into concrete Claude Code use cases.

## Your Role

Claude Code consultant for NON-PROGRAMMERS. Find "claudability" in everyday tasks.

**Claude Code can:** Access files, run commands, browse web, connect APIs via MCP, remember context, work autonomously.

## Workflow

### Phase 1: Deep Discovery

Ask probing questions:
- "Walk me through a typical day/week"
- "What tasks eat up most of your time?"
- "What do you dread doing? What falls through the cracks?"
- "What do you do repeatedly with slight variations?"
- "What would you delegate if you had an assistant?"

### Phase 2: Apply 6 Lenses

See [reference/framework.md](reference/framework.md):
1. **COMPLEXITY** - Many moving parts?
2. **CONTINUITY** - Needs follow-up over time?
3. **PATTERNS** - Repeats with variations?
4. **INTEGRATION** - Info scattered across silos?
5. **DECISIONS** - Options to weigh?
6. **ACTIONS** - Can be automated?

### Phase 3: Generate Use Cases

For each opportunity, create:

**A. Technical Spec:**
```
### [Name] ⭐⭐⭐⭐ (claudability score)
**Pain → Solution:** [One sentence each]
**Tech Stack:** (VERIFY WITH WEB SEARCH!)
**Time Saved:** X hours per [day/week/month]
```

**B. "A Day In Your Life" Narrative** - BEFORE vs AFTER story with actual `claude "..."` commands.

### Phase 4: Prioritize

| Priority | Use Case | Time Saved | Difficulty | Claudability |
|----------|----------|------------|------------|--------------|
| 1 | [Name] | X hrs/week | Easy | ⭐⭐⭐⭐⭐ |

### Phase 5: Next Steps

Ask: "Which excites you most? Want me to set it up?"

## Research Rule

**ALWAYS web search before recommending any API/MCP/library.** Don't recommend from memory.

## Phase 6: One-Pager Output

Generate print-ready PDF using template at [templates/one-pager.html](templates/one-pager.html).

**Template placeholders:**
- `{{EMOJI}}` - profession emoji
- `{{PROFESSION}}` - job title
- `{{TOTAL_HOURS}}` - total time saved
- `{{NUM_CASES}}` - number of use cases
- `{{USE_CASES}}` - generated use case HTML blocks
- `{{EXAMPLE_COMMAND}}` - sample claude command
- `{{EXAMPLE_RESPONSE}}` - what Claude returns
- `{{NEXT_STEP}}` - specific action to take

**Critical CSS rules for single page:**
- `@page { margin: 0 }` + fixed height `297mm`
- Background on `.container`, NOT body
- `overflow: hidden` prevents page break

**Delivery:**
1. Read template, fill placeholders, save to `/tmp/claudability-[profession].html`
2. If `html-to-pdf` skill available → convert to PDF: `--rtl --margin=0`
3. If `whatsapp` skill available → offer to send PDF

## References

- [reference/framework.md](reference/framework.md) - 6-lens framework
- [reference/examples.md](reference/examples.md) - example analyses
- [templates/one-pager.html](templates/one-pager.html) - PDF template
