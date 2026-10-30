# FlowmataGAS

**Visual workflow automation that runs entirely inside your own Google account. No server. No infrastructure. Free and open-source.**

> **A public preview is coming.** The code is being prepared for release now — watch this repo to get it the day it lands.

---

FlowmataGAS is an n8n / Make / Zapier / Activepieces-style automation tool with one unusual property: **the whole thing — the visual editor, the execution engine, the scheduler, the storage — runs on Google Apps Script, as *you*, inside your own account.** There is no backend to host, no database to run, no container to babysit, and no third party holding your credentials.

You make a copy of a spreadsheet, authorize it once, and start building on a node canvas.

If you're the kind of person who already lives in Google Sheets, Drive, Gmail and Calendar — and who would rather own their automation than rent it — this is for you.

---

## Why it's different

These aren't marketing lines; each is a deliberate engineering decision, verifiable in the code.

**It survives a hard 6-minute execution ceiling.**
Google Apps Script kills any invocation after 6 minutes. Most tools would just fail. FlowmataGAS **checkpoints and resumes**: a run pauses at a node boundary, persists its exact position (including mid-loop), and continues in the next invocation. A run longer than 6 minutes just… works. Your long workflows finish.

**Side effects happen at most once — not "at least once and hope."**
Checkpoint/resume is at-least-once by nature: a crash mid-node re-drives the run. The systems you already use mostly accept that and let a retry duplicate a write or an email. FlowmataGAS takes a **claim** before any side-effecting node runs; a retried run finds the claim and skips the duplicate. Loop iterations get their own claim, so iteration 2 doesn't collide with iteration 1.

**A real edge clock and real webhooks, through a tiny Cloudflare relay.**
Apps Script is severely limited for webhooks: every response is a 302 redirect, you can't set a status code other than 200, and you can't read the incoming request's headers. So the *decisions stay in Apps Script*, while a small Cloudflare Worker + Durable Object keeps the clock and answers the door: cron-accurate schedules, real `200`/`404` responses, signature verification (GitHub / Stripe / HMAC), and Telegram push. Optional — the base tool runs on Apps Script alone.

**Two expression engines, and imports start sandboxed.**
Expressions are real JavaScript inside `{{ }}`. The **native** engine runs with your permissions; the **sandboxed** engine (a JS interpreter) can't reach your account at all. An imported workflow starts on the sandboxed engine until you review and switch it — so running someone else's template is safe by default.

**Everything you can inspect is in the spreadsheet.**
Workflow definitions, run history, the system log, schedules, the dead-letter queue — all in visible tabs of the container spreadsheet. You can read, audit, and (carefully) repair your own automation's state. No black box.

**A visual editor that stays out of your way.**
A drag-and-drop node canvas with dark and light themes, three flow directions (left-to-right, right-to-left, top-to-bottom), pan, zoom and a minimap, one-click auto-arrange for messy graphs, custom connection styling (colour, path, line style, thickness, labels), undo/redo on every change, and a revision history you can diff and restore.

**No build step, ever, for the user.**
You don't `npm install` anything and you don't need a CLI. A template spreadsheet carries the whole product; a copy of it *is* the install.

---

## How it compares

Honest, not flattery. Each of these is excellent at what it's for; FlowmataGAS makes a different trade.

| | **FlowmataGAS** | **n8n** | **Make** | **Zapier** | **Activepieces** |
|---|---|---|---|---|---|
| **What it runs on** | Your Google account (Apps Script) | Node.js server / their cloud | Their cloud | Their cloud | Node.js server / their cloud |
| **Infra you operate** | **None** | Docker + DB + backups | None (they host) | None (they host) | Docker + DB |
| **Who holds your Google credentials** | **You** (runs *as* you) | OAuth app / them | Them | Them | OAuth app / them |
| **Cost model** | **Free, open-source** | Free self-host / paid cloud | Paid tiers | Expensive tiers | Free self-host / paid cloud |
| **Long-run reliability** | Checkpoint/resume built-in | Queue + workers (you run them) | Managed | Managed | Queue + workers |
| **Google-native depth** | **Full surface, no OAuth setup** | Good, via OAuth | Good, via OAuth | Good, via OAuth | Good, via OAuth |
| **Integrations count** | ~50 nodes + 50 connectors | 400+ | 1,500+ | 6,000+ | 200+ and growing |
| **AI** | Node + step-by-step Agent | Strong | Basic | Basic | Growing |
| **Scheduling precision** | Cron via relay | Cron | Interval | Interval | Cron |
| **Webhook realism** | Real status codes, signatures | Full | Full | Full | Full |
| **Team / multi-user** | Not in v1 | Yes | Yes | Yes | Yes |

**Where FlowmataGAS wins:** you operate nothing, your Google credentials never leave your account, retries can't duplicate side effects, and Google Sheets/Drive/Gmail/Calendar are first-class with zero OAuth setup.

**Where it doesn't:** the others have hundreds to thousands more integrations, real team workspaces, and managed scale. If you need 6,000 app connectors, a shared team space, or you're not Google-centric, use one of them — they're genuinely good.

---

## What you can build with it

- **Triggers** — manual, schedules (intervals, daily, weekly, cron), webhooks with idempotency + signature checks, Google Form/Sheet/Drive/Gmail/Calendar events, and connector polls.
- **Actions** — the full Google surface (Sheets, Drive, Gmail, Calendar, Docs with template fill), HTTP requests with auth, plus 16 connectors / 81 operations (Slack, Notion, GitHub, Airtable, Telegram, and more).
- **Logic** — If, Switch, Merge, Filter, Loop (nested), Set, Sort, Limit, Wait, Code, sub-workflows, error workflows, Human-in-the-Loop approvals (email and Telegram), and a writable Data Store.
- **AI** — an AI node (Ask, Extract, Classify, Summarize, Generate image) across Anthropic, OpenAI-compatible providers, Gemini, AWS Bedrock and Cloudflare Workers AI — and an AI Agent that works a task step-by-step with workflow tools, HTTP tools and MCP servers, with at-most-once claims on its writes.

---

## The honest constraints (read these before you fall in love)

FlowmataGAS is for **tech-savvy builders, developers and power-users** — and it's upfront about what that means:

- **It runs as you.** Workflows execute with *your* Google permissions. That's the feature (no OAuth dance for Google services) and the responsibility (don't share the container spreadsheet).
- **Platform quotas are real.** A consumer Google account has daily trigger-runtime and UrlFetch limits. The tool *shows* you the budget in the app bar before it bites, and the relay moves the heavy scheduling off the quota — but this is not infinite scale.
- **One account owns everything.** No multi-tenant team workspace in v1 (it's a deliberate non-goal).
- **Preview status.** The import/export format, node catalog and connector list are stable enough to build on, but this is a preview — expect rough edges and tell us what you hit.

---

## Status: what "preview" means here

The system is real and dogfooded — the reproducible end-to-end scenarios (invoice intake, webhook intake, Gmail triage, binary import) run against live services in the test harness, and the test suite is large and runs both in Node and on the real Apps Script runtime.

What's being finished for the public drop:

- TODO: more detailed roadmap.
- Additional Nodes.
- A starter set of example workflows.

**Watch this repository** to be notified when the code and the template go up — and when the first community template gallery opens.

---

## License

Free and open-source.

---

*Built by someone who got tired of renting automation they could own.*
