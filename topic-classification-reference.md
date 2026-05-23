# Session Topic Classification Reference

> Quick lookup table for classifying Hermes Agent session titles.

## Skip Rule (v2.0.0)

When running the skill, first check each session's title against:

```
^[a-z][a-z_]+-\d{10}$
```

- **Match** → Skip entirely (already properly named)
- **No match** → Process (classify, rename, or delete)

## Keyword → Topic Mapping (Examples)

| Keywords in first user message | Topic name | Action |
|--------------------------------|------------|--------|
| trend, trending, push, scheduled task, data push | `project_A_trend` | Rename |
| desktop app, desktop issue, office not working, module install | `desktop_app_issue` | Rename (abbreviated) |
| install env, setup environment, environment config | `env_setup` | Rename |
| install skill, setup plugin, learn this skill | `skill_setup` | Rename |
| session cleanup, organize sessions, delete session | `session_cleanup` | Rename |
| connection test, connectivity test, verification test | *(delete)* | Delete |
| agent tool usage, /approve, /retry | *(delete)* | Delete (pure permission/tool ops) |
| [IMPORTANT: You are running as a scheduled cron job | *(delete)* | Delete |
| continue unfinished task, token restored, resume work | *(check parent)* | Investigate first |

## Length Check Calculation

Formula: `len(f"{topic_name}-YYMMDDHHMM")`

| Topic | Full | With timestamp | Fits 28? |
|-------|------|----------------|----------|
| `project_A_trend` | 16 | 26 | ✅ |
| `session_cleanup` | 16 | 26 | ✅ |
| `skill_setup` | 12 | 22 | ✅ |
| `env_setup` | 10 | 20 | ✅ |
| `desktop_app_issue` | 17 | 27 | ✅ |

## Abbreviation Rules

- Use FULL topic names when `len(topic) + 10 ≤ 28`
- Only abbreviate when the full name exceeds 28 chars
- For project names, keep recognizable abbreviation: `hermes` → `hms`, not outright removal
- **Never drop the project identifier** — `desktop_app_issue` fits as-is, no abbreviation needed
- In this example set, all topic names fit within 28 chars — no abbreviation required

## Timestamp Extraction Priority

1. `started_at` field (preferred, already a Unix timestamp)
2. `created_at` field
3. `messages[0].timestamp`
4. Filename heuristic (session ID prefix)

Format with: `datetime.fromtimestamp(ts).strftime('%y%m%d%H%M')`
