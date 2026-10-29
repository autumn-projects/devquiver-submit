---
name: devquiver-submit
description: Submit a website or tool page to backlink directories (AI tool lists, launch platforms, startup directories) using the devquiver MCP server and the browser, one directory at a time, and record what actually happened. Use when the user wants to "submit to directories", "build backlinks", "list my product on launch sites", or "work through my devquiver to-do list". Requires the devquiver MCP tools (list_pages, next_directories, get_directory, get_copy_kit, list_submissions, record_submission, import_directory) and a browser tool.
---

# devquiver-submit

You work through a user's backlink to-do list: pick the next directory, fill its submission form from the user's copy kit, stop at anything that needs a human, and write back what you *observed* — so the directory library stays honest.

The submission is made on the user's behalf and is public. Treat the final click as theirs unless they have told you to run unattended.

## Before you start

1. Confirm the devquiver MCP tools are available. If not, tell the user to connect the remote MCP server `https://www.devquiver.app/api/mcp` with the header `Authorization: Bearer <token>` (token from the workspace → MCP page; it needs **write** permission or `record_submission` will fail with 403). The exact command depends on the agent:
   - Claude Code: `claude mcp add --transport http devquiver https://www.devquiver.app/api/mcp --header "Authorization: Bearer <token>"`
   - Codex: in `~/.codex/config.toml` add `[mcp_servers.devquiver]` with `url = "https://www.devquiver.app/api/mcp"` and `bearer_token_env_var = "DEVQUIVER_TOKEN"`, then set that environment variable.
   - WorkBuddy and other JSON-config clients: add `"devquiver": { "url": "https://www.devquiver.app/api/mcp", "headers": { "Authorization": "Bearer <token>" } }` under `mcpServers` (WorkBuddy: `~/.workbuddy/mcp.json`), then enable it.
   Never ask the user to paste the token into the chat; they put it in their own config.
2. `list_pages` → pick the page (a tool page URL, not necessarily the homepage). If several, ask which.
3. `get_copy_kit` for that page. If name / tagline / descriptions are empty, ask the user to fill them in the workspace first — never invent copy.
4. Browser access: you need a browser tool that can open pages, click and fill forms (e.g. Claude in Chrome, a Playwright/Chrome DevTools MCP server, or the agent's built-in browser). If the agent has none, you can still pick directories, prepare the copy for each form and record results, while the user does the clicking. Check you can reach the target site.

## Two modes · 两种模式

Ask the user which mode they want if they haven't said. Most sessions alternate: one autonomous pass, then one collaborative pass to clear what got stuck.

**Autonomous pass (全自动轮)** — the user has said "run unattended" / "don't stop, just record blockers".
- **Only directories this page has never touched.** `next_directories` returns everything not yet submitted, including ones already recorded for this page as `todo`, `drafting` or `skip`. Before working the list, call `list_submissions` for the page with no status filter and drop **every** directory that has any record for it. An autonomous pass works only on directories with no record at all for this page. Everything already recorded (blocked, half-filled, waiting on a badge, retry later) belongs to the collaborative pass, even if you think you could clear it now — say so in the end-of-pass report instead of re-probing.
- Submit everything you can finish yourself without asking before each final click.
- Anything that needs a human (login wall you may not pass, CAPTCHA, email code, payment decision, a badge or link that must go on the user's site first) is **not a reason to stop the session**: record it as `drafting` with a note that says exactly what is needed and where, then move to the next directory.
- Write notes so the collaborative pass can batch them: start with the blocker type, e.g. `卡点: Cloudflare 验证`, `卡点: 需登录`, `卡点: 需放徽章 <badge html>`, `卡点: 选收入/经营数据`.
- At the end, report counts per blocker type, not a wall of text.

**Collaborative pass (人机协作轮)** — the user is at the keyboard.
1. `list_submissions` with `status: ["drafting"]` for the page.
2. Group them by blocker type and hand the user one group at a time ("these 4 need you to pass a Cloudflare check — open them, pass it, tell me").
3. When the user says a group is cleared, finish each one (fill the rest, submit, verify) and re-record it.
4. Badges / links on the user's site: place them only through whatever mechanism the user has set up for this (a file in their repo, a database row, a CMS) and only with their go-ahead; confirm the badge is in the live page's HTML before clicking the directory's "verify" button — verification bots fetch raw HTML and don't run JavaScript.

## The loop (one directory at a time)

1. `next_directories` with `page`, `limit` 5, and `free_only: true`. Entries marked `locked: true` are outside today's free picks — they have no submit link. Don't try to work around that: tell the user the free plan opens 10 directories a day (plus ones they've submitted to) and that the full library is a one-time unlock.
2. Pick the best fit, then `get_directory` for the details (paywall, login, badge rule, confirmation email, notes). Read the notes — they are the point of the tool.
3. **Decide whether this page should skip it** (record `skip` with a one-line reason, then move on). `skip` is a choice about *this page*, not a verdict on the directory: a directory that doesn't fit this product may be right for another, and some users do want paid listings. So:
   - Paid listing, or the form asks for payment at the last step → don't pay on your own. Skip it unless the user has said they want paid listings; if they have, tell them the price and get an explicit yes before paying.
   - Category mismatch for *this* product (e.g. AI-only directory for a non-AI product; B2B review sites; developer-infra lists).
   - The directory *requires* a link back to some other unrelated site, or a badge you can't place.
   - It only takes one listing per domain and another page of the same site already holds it.
   Never write 'not applicable' or similar into `observed`/shared fields; put the reason in this page's `note` only.
4. Open the submit URL. Fill the form from the copy kit:
   - Website / URL field = the page URL the user chose.
   - Use only facts from the copy kit. Don't invent founding dates, social handles, pricing, user counts. Leave optional fields blank rather than guess.
   - Don't put numbers that go stale (credit counts, prices) in descriptions.
   - Respect length limits; shorten the short description rather than truncating mid-word.
   - Read each value back after typing. Some React/Framer inputs drop characters (e.g. `/`) or autoscroll; fill one field at a time, then verify. If typing fails, set the value through the page's own input events rather than retrying blindly.
5. **Before the final submit, show the user a summary** (directory, URL, the key fields) and ask to proceed — unless they've explicitly told you to run unattended.
6. After submitting, look for a real confirmation (success message, queue notice, email prompt). A button that does nothing is *not* success: check the network tab / console for failed requests before claiming anything.
7. `record_submission` (see below), then the next directory.

## Stop and hand back to the human

Never do these yourself — stop, record a blocker, and tell the user exactly what is needed:

- CAPTCHA, Cloudflare/"verify you are human" checks, arithmetic or quiz bot-checks.
- Creating an account, choosing a password, entering any password.
- Email confirmation codes / magic links that arrive in the user's inbox.
- Payment, billing, or entering card details.
- "Sign in with Google": only click it if the user has told you it's OK and the browser is already signed in to the account they want. Pick the account they named; if unsure, ask. Note that a first Google sign-in on a site usually **creates an account** there — the user's permission must cover that, not just logging in to existing accounts. Only approve the basic name/email/profile consent; stop if a site asks for wider Google permissions.
- Anything that asks for ID, phone number, or personal data the copy kit doesn't contain.

If a form needs the user (login wall, verification), save progress as `drafting` with a note saying what's blocking, then move to the next directory. Don't loop: **two failed attempts at the same step is a blocker, not a retry.**

## Recording results (`record_submission`)

- `status`:
  - `submitted` — you clicked submit and saw a confirmation, or it's queued for review.
  - `drafting` — filled but blocked (needs email confirmation, login, votes, human check). Put the blocker in `note`.
  - `live` — **only** after you've opened the public listing and confirmed it's up with a link to the user. Never mark `live` on a guess.
  - `rejected` — the directory said no.
  - `skip` — this page chose not to submit (paid, doesn't fit this product…), with the reason in `note`. It says nothing about whether the directory suits other products.
- `note`: submission ID, queue/review dates, conditions (e.g. "needs 10 upvotes", "confirmation email sent — check spam"). No personal data (no emails, passwords) in notes.
- `observed`: **only what you actually saw on the page.** It's a partial update; omit anything you didn't verify.
  - `pricing`: `free` | `reciprocal` | `paid`
  - `paywall`: `none` | `entry` (paid from the start) | `final` (paid only at the last step)
  - `login`: `none` | `email` | `google`
  - `badge`: `none` | `any` | `home` (badge must be on the homepage) | `listing` (must be on the listing page)
  - `confirmEmail`: `none` | `yes` | `spam-folder`
  - `listingUrl`: the public page that links to the user, once it exists.
  Values that differ from the library are treated as change signals (queued for review, or applied directly if the token belongs to a maintainer) — so accuracy matters more than filling every field.

## Honesty

- If a directory requires a badge or link on the user's site, don't edit their site unasked — record it in the note and tell the user.
- Use `import_directory` for directories you discover that aren't in the library. If it's already there, it returns the library entry and the user's own records — use those instead of re-testing.

## When you finish a session

Give a short report: submitted (with names), blocked (with exactly what each needs from the user), skipped (with reasons), and what's next. Don't claim a backlink exists until it's live and verified, and don't promise rankings or indexing — directories decide that, not us.
