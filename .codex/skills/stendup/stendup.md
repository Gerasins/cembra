---
description: Daily standup helper — collects yesterday's tasks with time, suggests today's plan, and writes a formatted entry with timeline to the monthly standup log.
---

Help the user run their daily standup. Follow the steps below in order, speaking Russian throughout.

## Step 1 — Yesterday

Ask: "Что делал вчера? Перечисли задачи и сколько времени заняла каждая."

Wait for the answer. Collect a list of tasks with time spent on each.

## Step 2 — Today

Gather context clues:
- If there is a git repo in the current directory, run `git log --oneline -10`
- Read any previous entries in `/Users/a.gerasins/Projects/Wiki/Cembra/stendup/` for recurring themes

Propose 3–5 tasks for today as a numbered list, then ask:
"Что планируешь сегодня? Вот мои предложения (можешь добавить или изменить):"

Wait for confirmation or edits before proceeding.

## Step 3 — Write the entry

Target file: `/Users/a.gerasins/Projects/Wiki/Cembra/stendup/MM.YYYY.md`
Use today's real date to build the path (e.g. `06.2026.md`).

If the file does not exist, create it with this header first:
```
# Stendups MM.YYYY
```

Prepend the new entry directly below the header (newest entry always at the top):

```
### DD.MM.YYYY

**Yesterday:** Yesterday/Last Friday I worked on <free-form summary of tasks>.

**Today:** Today I'm planning to <free-form summary of tasks>.

**Timeline:**
| Task | Time |
|------|------|
| <task> | <time> |

---
```

Time format: `2h`, `1.5h`, `30min`.

After saving, reply: "Записал в [filepath]. Удачного стендапа! 🚀"
