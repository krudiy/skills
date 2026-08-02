---
name: coolapk-search
description: 检索酷安（Coolapk）社区里的帖子/用户/话题/应用，并查看帖子详情与评论。当用户想“搜酷安”“查酷安帖子”“酷安社区搜索”“coolapk”或需要获取酷安帖子包含的信息（正文、评论、图片、作者/时间/点赞/话题）时使用。基于本地自包含 Python 客户端，游客模式即可读取公开内容。
---

# 酷安帖子检索（coolapk-search）

本技能通过酷安 v6 接口实时检索社区公开内容，无需登录（游客模式）。

## 运行环境

使用受管 Python 的 venv（已安装 bcrypt）：
```
PYTHON="C:/Users/Administrator/.workbuddy/binaries/python/envs/default/Scripts/python.exe"
SKILL_DIR="C:/Users/Administrator/.workbuddy/skills/coolapk-search"
```

## 调用方式（均由 agent 执行）

默认输出**紧凑 JSON**（便于解析）；加 `--text` 输出人类可读文本。

### 1. 搜索帖子（最常用）
```
"$PYTHON" "$SKILL_DIR/coolapk_client.py" search "关键词" [--page 2] [--text]
```
- 其他类型：`--type user`（用户） / `--type feedTopic`（话题） / `--type app`（应用）
- 翻页：`--page N`

### 2. 帖子评论（游客模式可用，推荐）
```
"$PYTHON" "$SKILL_DIR/coolapk_client.py" comments <帖子ID> [--page 2] [--text]
```
- 评论接口对游客开放，稳定返回用户名、时间、点赞、内容。

### 3. 帖子详情 + 评论
```
"$PYTHON" "$SKILL_DIR/coolapk_client.py" feed <帖子ID> --comments [--text]
```
- 帖子 ID 来自搜索结果里的 `id` 字段（如 `https://www.coolapk.com/feed/123456` 中的 `123456`）。
- ⚠️ 游客模式下 `/v6/feed/detail` 常被酷安验证码（err_request_captcha_v2）拦截，**无法获取完整正文**；此时命令会给出提示，但评论仍可取。正文摘要请用 `search` 获取。若要稳定拿到完整正文，需走登录模式（见下）。

### 4. 浏览首页动态（可选）
```
"$PYTHON" "$SKILL_DIR/coolapk_client.py" home --tab hot   # hot | latest | follow
```

## 使用建议
- 用户说“搜酷安 xxx / 酷安里关于 xxx 的帖子” → 用 `search "xxx" --text` 取结果（已含正文、作者、时间、点赞、话题、图片、链接），挑几条有价值的口头总结并附链接。
- 用户想看某帖讨论 → 用 `comments <id> --text`（游客即可）。
- 关键词检索是按“匹配到的帖子”返回并支持翻页；酷安全平台帖子量极大，无法逐条枚举，但可翻页取回所有匹配项。
- 遇到空结果或报错：先确认网络；401/403/429 客户端已内置重试；若 `feed` 提示验证码，属酷安游客限制，改用 `search` 拿正文摘要。

## 已知限制（游客模式）
- 完整帖子正文需过验证码，`feed` 详情端点对游客拦截；`search` 返回的正文摘要通常足够检索使用。
- 酷安对游客有请求频率限制，客户端已做重试退避；高频大量检索建议改为登录模式（在 config.json 写入账号 Cookie：uid/username/token，并扩展 client 发送 Cookie 头）。

## 登录模式（可选升级）
当前为游客只读。如需完整正文/更高限额，可在 `config.json` 增加 `uid`/`username`/`token`，并在 `coolapk_client.py` 的 `build_headers()` 中附加 `Cookie: uid=...; username=...; token=...`；`coolapk-mcp` 项目已验证该方式可读写。

## 示例输出字段（search 的 results 每项）
`id, title, message(正文摘要), username, uid, time, like_count, reply_count, pic(图片链接), topic(话题), url(原文链接)`

## 合规
仅用于个人检索公开内容，默认游客只读，不爬取隐私、不执行任何写操作。
