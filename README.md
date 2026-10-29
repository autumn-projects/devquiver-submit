# devquiver-submit

An agent skill (Claude Code · Codex · WorkBuddy) for submitting your product to backlink directories with an AI agent — honestly, one directory at a time.

一个 agent skill（Claude Code · Codex · WorkBuddy）：让 AI 帮你一个个提交外链目录，并如实记录结果。

Lets an AI agent work through your backlink to-do list: take the next directory from [devquiver](https://www.devquiver.app), fill its submission form from your copy kit, stop whenever a human is needed (CAPTCHA, account creation, email codes, payment), and write back what actually happened.

让 AI agent 帮你推进外链提交清单：从 devquiver 取下一个目录，按文案包填表，遇到验证码、注册、邮箱验证码、付款就停下交还给你，并把真实情况回写。

## Install · 安装

Works with **Claude Code**, **OpenAI Codex** and **Tencent WorkBuddy**: they all read the same `SKILL.md`. You need two things: the devquiver MCP server connected, and this folder in the agent's skills directory.

支持 Claude Code、OpenAI Codex、腾讯 WorkBuddy，三者读同一份 `SKILL.md`。需要两步：接入 devquiver 的 MCP，把本目录放进对应的 skills 目录。

Get a token from the devquiver workspace → MCP page, **with write permission**. 在 devquiver 工作台 → MCP 页生成 token，**勾选写权限**。

### Claude Code

```
claude mcp add --transport http devquiver https://www.devquiver.app/api/mcp --header "Authorization: Bearer <your-token>"
git clone https://github.com/autumn-projects/devquiver-submit ~/.claude/skills/devquiver-submit
```

### OpenAI Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.devquiver]
url = "https://www.devquiver.app/api/mcp"
bearer_token_env_var = "DEVQUIVER_TOKEN"
```

Set `DEVQUIVER_TOKEN` in your environment, then:

```
git clone https://github.com/autumn-projects/devquiver-submit ~/.agents/skills/devquiver-submit
```

### WorkBuddy

Add to `~/.workbuddy/mcp.json` (or Connectors → custom MCP), then turn it on under My MCP:

```json
{
  "mcpServers": {
    "devquiver": {
      "url": "https://www.devquiver.app/api/mcp",
      "headers": { "Authorization": "Bearer <your-token>" }
    }
  }
}
```

Put this folder at `~/.workbuddy/skills/devquiver-submit/` and restart WorkBuddy. Custom skills and MCP may need WorkBuddy's paid/enterprise plan.

### Then · 然后

1. Add a site and a page in the workspace, and fill in its copy kit (name, tagline, descriptions).
2. Give the agent a browser tool (Claude in Chrome, a Playwright / Chrome DevTools MCP server, or its built-in browser). Without one it can still pick directories and prepare the copy while you do the clicking.
3. Ask: "Submit my page to the next 5 directories."

## What it uses · 用到的 MCP 工具

`list_pages` · `get_copy_kit` · `next_directories` · `get_directory` · `import_directory` · `list_submissions` · `record_submission`

## Limits · 说明

- The free plan opens 10 directories a day (plus ones you've submitted to); the full library is a one-time unlock. Locked entries can't be worked on.
- It never solves CAPTCHAs, creates accounts, enters passwords or pays for listings, and it asks before the final submit unless you tell it to run unattended.
- Two modes: an **autonomous pass** that submits what it can and records every blocker as `drafting`, and a **collaborative pass** that pulls those drafts, groups them by blocker (login, CAPTCHA, badge…) and finishes them once you clear each group.
  两种模式：全自动轮能提交的直接提交、卡住的记成 drafting；人机协作轮把 drafting 按卡点分组交给你，你处理完一组它接着做完。
- It records only what it observed, so the library stays accurate. It can't promise indexing or rankings — directories decide that.

## License

MIT
