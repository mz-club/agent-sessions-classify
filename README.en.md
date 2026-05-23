# agent-sessions-classify — Hermes Agent Session Organizer

> Classify, rename, and clean up Hermes Agent session titles — only process sessions that don't follow the naming convention. Skip already-properly-named sessions, group related sessions under unified topic names, and remove noisy test/cron sessions.

## Trigger

Say any of the following to activate:
- "organize my hermes sessions"
- "classify all my sessions"
- "clean up my session list"
- "rename all sessions"
- "整理我hermes所有会话" (Chinese)

## v2.0.0 New Feature: Skip Already-Named Sessions

Before processing, each session title is checked against:

```
^[a-z][a-z_]+-\d{10}$
```

(Starts with lowercase letters + underscores, dash, exactly 10 digits = YYMMDDHHMM)

- **Match** → Skip entirely, no changes
- **No match** → Process (classify, rename, or delete)

This includes: untitled `—` sessions, sessions with random manual titles,
sessions with wrong format (missing timestamp, wrong separator).

## Naming Convention

```
<topic_name>-YYMMDDHHMM
```

- No spaces around the dash
- 2-digit year
- No underscores in timestamp
- Total length **must not exceed 28 characters** (list truncation limit)

## Name Length Rules

Use full topic names when they fit within 28 chars. Shorten only when necessary.

| Full name | Length | 28-limit | Final name |
|-----------|--------|----------|------------|
| `project_A_trend` | 16 chars | 16+10=26 ✅ | `project_A_trend` |
| `session_cleanup` | 16 chars | 16+10=26 ✅ | `session_cleanup` |
| `skill_setup` | 12 chars | 12+10=22 ✅ | `skill_setup` |
| `env_setup` | 10 chars | 10+10=20 ✅ | `env_setup` |
| `desktop_app_issue` | 17 chars | 17+10=27 ✅ | `desktop_app_issue` |

## Common Abbreviations

| Full | Short | Note |
|------|-------|------|
| `hermes` | `hms` | Project abbreviation |
| `github` | `gh` | Common shorthand |
| `desktop` | `desktop` | Keep as-is |
| `issue` | `iss` | Problem/fix |
| `trend`/`trending` | `tr` | Trending topic |
| `setup` | `setup` | Keep as-is |
| `cleanup` | `clean` | Housekeeping |
| `search` | `srch` | Search topic |

> Use abbreviations only when `topic_name` exceeds 28 chars. Prefer full names.

### Topic Definitions (Examples)

| Topic Name | Contains |
|------------|----------|
| `project_A_trend` | Project A trends configuration, push troubleshooting, 3rd-party integration |
| `desktop_app_issue` | Desktop app installation, build, fixes |
| `skill_setup` | Skill exploration and setup |
| `env_setup` | Environment configuration |
| `session_cleanup` | Session housekeeping tasks |

## Quick Commands

```bash
# List all sessions
hermes sessions list --limit 60

# Inspect a session
hermes sessions export --session-id <ID> -

# Rename
hermes sessions rename <ID> "project_A_trend-2501010915"

# Delete
hermes sessions delete --yes <ID>
```

## Cleanup Rules

**Delete:** cron logs, single-message connection tests, empty sessions
**Keep:** Current session, conversations with >2 messages, user-specified sessions
