# devquiver-submit

A Claude Code skill for submitting your product to backlink directories with an AI agent — honestly, one directory at a time.

一个 Claude Code skill：让 AI 帮你一个个提交外链目录，并如实记录结果。

Lets an AI agent work through your backlink to-do list: take the next directory from [devquiver](https://www.devquiver.app), fill its submission form from your copy kit, stop whenever a human is needed (CAPTCHA, account creation, email codes, payment), and write back what actually happened.

让 AI agent 帮你推进外链提交清单：从 devquiver 取下一个目录，按文案包填表，遇到验证码、注册、邮箱验证码、付款就停下交还给你，并把真实情况回写。

## Install · 安装

1. Connect the MCP server (token from the workspace → MCP page, **with write permission**):

   ```
   claude mcp add --transport http devquiver https://www.devquiver.app/api/mcp --header "Authorization: Bearer <your-token>"
   ```

2. Clone into your skills directory:

   ```
   git clone https://github.com/autumn-projects/devquiver-submit ~/.claude/skills/devquiver-submit
   ```

   (or into a project's `.claude/skills/`).
3. Add a site and a page in the workspace, and fill in its copy kit (name, tagline, descriptions).
4. Ask: "Submit my page to the next 5 directories."

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
