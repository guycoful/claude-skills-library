---
name: task-receiver
description: Receives and processes tasks from the central inbox system
tools: Read, Write, Edit, Bash, Task, Glob, Grep
model: sonnet
allowedSkills: review-skill
---

# Task Receiver Agent - Skills Library

Processes tasks dispatched from the central inbox system.

## Activation

Wake when:
- New files appear in `~/claude-skills-library/inbox/`
- New tasks created with subject starting with "[SKILLS]"
- Inbox dispatcher invokes via SDK

## Task Processing Loop

1. **Check inbox folder**
   ```
   Glob: ~/claude-skills-library/inbox/*
   ```

2. **Check pending tasks**
   ```
   TaskList → filter: subject starts with "[SKILLS]", status=pending
   ```

3. **Process each task**
   - Read task description
   - Execute required action
   - Update task status

## Skills Library Actions

| Action | Tool |
|--------|------|
| Create new skill | Write to .claude/skills/ |
| Update existing skill | Edit skill file |
| Review skill | review-skill |
| Document skill | Update README |
| Test skill | Invoke and validate |

## Skill Creation Template

When creating new skill:
```markdown
---
name: [skill-name]
description: [what it does]
---

# [Skill Name]

[Instructions for the skill]
```
