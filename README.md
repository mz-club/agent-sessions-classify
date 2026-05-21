# agent-sessions-classify — Session 分类与命名工具

> 管理 Hermes Agent 的 session 列表，对未命名的 session 进行话题分类、统一命名，并清理无用会话。

## 触发方式

📖 English version: [README.en.md](README.en.md)

说以下任意一句话即可激活：
- **"整理我hermes所有会话"**
- "整理我所有会话"
- "帮我整理hermes会话"
- "分类命名所有session"
- "清理会话记录"
- "organize my hermes sessions"
- "classify all my sessions"

## 命名格式

```
<topic_name>-YYMMDDHHMM
```

- 短横线两侧无空格
- 年份2位
- 时间戳无下划线
- 总长度 **不超过28字符**

## 长度规则与缩写对照

使用全称，只有当全称超过28字符时才缩短。

| 全称 | 字符数 | 28限制 | 使用名 |
|------|--------|--------|--------|
| `project_A_trend` | 16 | 16+10=26 ✅ | `project_A_trend` |
| `session_cleanup` | 16 | 16+10=26 ✅ | `session_cleanup` |
| `skill_setup` | 12 | 12+10=22 ✅ | `skill_setup` |
| `env_setup` | 10 | 10+10=20 ✅ | `env_setup` |
| `desktop_app_issue` | 17 | 17+10=27 ✅ | `desktop_app_issue` |

### 话题定义（示例）

| 话题名 | 包含内容 |
|--------|---------|
| `project_A_trend` | 项目A的热门趋势配置、推送排查、第三方集成 |
| `desktop_app_issue` | 桌面应用安装、编译、修复问题 |
| `skill_setup` | 技能安装探索 |
| `env_setup` | 环境安装配置 |
| `session_cleanup` | Session 整理清理任务 |

## 命令速查

```bash
# 查看所有 session
hermes sessions list --limit 60

# 查看单个 session 内容
hermes sessions export --session-id <ID> -

# 重命名
hermes sessions rename <ID> "project_A_trend-2501010915"

# 删除
hermes sessions delete --yes <ID>
```

## 清理规则

**可删除：** cron 日志、单条测试连通性 session、空 session
**保留：** 当前对话、>2条消息的对话、所有需要保留的 session
