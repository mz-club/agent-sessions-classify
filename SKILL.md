---
name: agent-sessions-classify
description: "Use when classifying, renaming, and tidying Hermes Agent session titles. Groups related sessions under unified topic names with timestamps, deletes test/cron sessions, and ensures consistent naming conventions."
version: 1.1.0
author: "MZ"
license: MIT
metadata:
  hermes:
    tags: [sessions, classification, housekeeping, hermes-admin]
    related_skills: [hermes-agent]
---

# Session Classification & Housekeeping

## Overview

Manage Hermes Agent session list (`hermes sessions list`). Classify untitled sessions into topic groups, rename with consistent `topic_name-YYMMDDHHMM` format, and clean up noisy sessions (cron logs, connection tests, single-message tests).

## When to Use

### Trigger Phrases — Say any of these to activate:

**中文：**
- "整理我hermes所有会话"
- "整理我所有会话"
- "帮我整理hermes会话"
- "分类命名所有session"
- "清理会话记录"

**English:**
- "organize my hermes sessions"
- "classify all my sessions"
- "clean up my session list"
- "rename all sessions"

- `hermes sessions list` shows many untitled (`—`) sessions
- User asks to clean up, rename, or organize sessions
- After major troubleshooting cycles, need to categorize sessions for future reference
- User wants to delete stale cron sessions or short test sessions
- You see sessions with uuid-style IDs (e.g. `d40acd6b-1e67-403a-a567-c494b56fb5a7`) that have no meaningful title

## Workflow

### Step 1: Survey Existing Sessions

```bash
hermes sessions list --limit 60
```

Review the Title, Preview (first user message), and ID columns. Look for:

- Sessions with `—` as title (untitled)
- Sessions with `cron_` prefix (cron job logs)
- Sessions with uuid-style IDs from api_server
- Sessions with 1-2 user messages that are clearly test messages

### Step 2: Inspect Untitled Sessions

For each untitled session, export and read its first user message to determine the topic:

```bash
hermes sessions export --session-id <SESSION_ID> - | python3 -c "
import json, sys
data = json.load(sys.stdin)
msgs = data.get('messages', [])
user_msgs = [m for m in msgs if m.get('role') == 'user' and m.get('content')]
first = str(user_msgs[0]['content'])[:150] if user_msgs else '(none)'
ts = data.get('started_at') or data.get('created_at') or ''
print(f'msgs={len(msgs)} first=\"{first}\"')
"
```

Key clues for topic classification:

| First message contains | Likely topic |
|---|---|
| "github", "trend", "推送", "定时任务" | `github_trend` |
| "hermes-desktop", "hermes desktop", "office无法显示", "next和ws模块" | `hermes_desktop_issue` |
| "安装v2ray" | `v2ray_setup` |
| "multi-search-engine", "clawhub.ai/.../multi-search" | `multi_search` |
| "cn-trends", "推送热点" + date 5/10-5/12 | `github_trend` (early phase) |
| "安装此技能", "学习此技能" + "cn-trends" | `github_trend` |
| "检查...未能推送", "Weixin频道", "推送到微信", "iLink" | `github_trend` |
| "测试连通性", "连通性测试", "验证能否推送" | Delete (connection test) |
| "继续上述未完成的任务" | Check parent session's topic |
| "[IMPORTANT: You are running as a scheduled cron job" | Delete (cron log) |

### Step 3: Group by Topic and Time

Group all sessions by topic. Within each topic, sort by timestamp (oldest first).

Use the **earliest timestamp** from:
- `started_at` field (preferred)
- `created_at` field
- Export JSON's `messages[0].timestamp`
- If none available, use a filename-based heuristic

**Naming convention:**
```
<topic_name>-YYMMDDHHMM
```

No spaces around the dash. Year is 2-digit. No underscores in timestamp.

**Length rule:** Total title (topic_name + dash + YYMMDDHHMM) must be ≤ 28 chars. Use full topic names when they fit, shorten only when necessary.

| Full name | Length | Shortened (if needed) |
|-----------|--------|----------------------|
| `github_trend` | 13 chars | 13+10=23 ✅ fits as-is |
| `session_cleanup` | 16 chars | 16+10=26 ✅ fits as-is |
| `multi_search` | 13 chars | 13+10=23 ✅ fits as-is |
| `v2ray_setup` | 11 chars | 11+10=21 ✅ fits as-is |
| `hermes_desktop_issue` | 22 chars | 22+10=32 ❌ shorten to `hms_desktop_iss` (keep `hms` as project abbreviation + `desktop` + `iss` for issue) = 17+10=27 ✅ |

### Step 4: Delete Unnecessary Sessions

**Sessions safe to delete** (ask user first):
- `cron_*` sessions (old cron job logs, no user messages)
- Single-message connection test sessions ("连通性测试", "验证能否推送")
- Empty sessions with 0 messages
- Sessions where the only user message is "[IMPORTANT: You are running as a scheduled cron job..."

```bash
hermes sessions delete --yes <SESSION_ID>
```

**Do NOT delete:**
- The current active session (where you are talking to the user)
- Sessions with meaningful conversation history (>2 user messages)
- Weixin sessions (platform=weixin) that may have ongoing context
- Sessions the user specifically asked to keep

### Step 5: Rename Sessions

```bash
hermes sessions rename <SESSION_ID> "<topic_name>-YYMMDDHHMM"
```

**Important:** The title is truncated in `hermes sessions list` display after ~28 characters. Always verify total length ≤ 28. Examples:
- `github_trend-2605160759` (23 chars) ✅ fits
- `hms_desktop_iss-2605161230` (27 chars) ✅ fits
- `hermes_desktop_issue-2605161230` (31 chars) ❌ gets cut to `hermes_desktop_issue-2` — useless

### Step 6: Verify

```bash
hermes sessions list --limit 60
```

Check that:
- All topics are grouped together visually
- Timestamps are in correct chronological order
- No untitled sessions remain (unless intentionally left)
- No cron/test sessions remain (if user asked to clean them)

## Handling Ambiguous Boundaries

Some sessions span multiple topics (e.g., a session starts with "连通性测试" but moves into "GitHub Trending推送排查"). Rules:

1. **Look at the FULL conversation** — not just the first message. If >50% of user messages relate to github_trend, classify as github_trend.
2. **Merge into the broader topic** — early WeChat connectivity tests are part of the github_trend problem, not a separate topic.
3. **If truly mixed and no dominant topic**, use the topic of the LAST substantive user message.

## Common Pitfalls

1. **Renaming the current active session.** Session you're talking in right now should be `session_cleanup` or the original topic — don't rename to something confusing.
2. **Using wrong timestamp.** Don't use `hermes sessions list` display time (relative like "2d ago"). Always get the actual timestamp from export JSON.
3. **Creating duplicate topic names.** Before inventing a new topic, check if the session could be absorbed into an existing topic group (especially `github_trend` which spans many sub-topics).
4. **Forgetting to sync three docs.** When updating github_trend workflow, remember the three-doc sync: README.md + prompt.txt + SKILL.md.
5. **Over-naming.** Sessions with only 1-2 user messages and no useful content should be DELETED, not named.
6. **Title truncation.** `hermes sessions list` truncates titles after ~28 chars. `github_trend-2605160759` (23 chars) fits fine. `hermes_desktop_issue-2605161230` (31 chars) gets cut to `hermes_desktop_issue-2` — useless. Always verify: `len(f"{topic}-{ts}") ≤ 28`. For `hermes_desktop_issue`, keep project abbreviation: `hms_desktop_iss` (17 chars + 10 = 27 ✅).

## Verification Checklist

- [ ] `hermes sessions list` shows all sessions with meaningful titles
- [ ] Sessions grouped by topic appear adjacent in the list
- [ ] No `—` (untitled) sessions remain (unless user-specified exceptions)
- [ ] No `cron_*` sessions visible (if user asked to delete them)
- [ ] Titles in correct `<topic_name>-YYMMDDHHMM` format (no spaces, 2-digit year, no underscores)
- [ ] Total title length ≤ 28 chars (confirm with `len()` before renaming)
- [ ] Topics within a group sorted chronologically
- [ ] Current conversation session has appropriate title (`session_cleanup` or similar)
- [ ] Memory updated with any new conventions discovered during classification

## Reference Files

- `references/topic-classification-reference.md` — Quick lookup table for keyword→topic mapping, abbreviation rules, and length calculations

## Three-Doc Sync

This skill follows the same three-doc sync pattern as mygithub_trend. When updating:
1. SKILL.md (this file) — for agent reuse via `skill_load`
2. `/home/ts/文档/agent-sessions-classify/agent-sessions-classify_readme.md` — human-readable documentation
3. `/home/ts/文档/agent-sessions-classify/agent-sessions-classify_prompt.txt` — AI-readable prompt for future sessions

Always update all three together.
