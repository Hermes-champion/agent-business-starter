# Agent Business Starter (free, MIT)

Three Agent Skills (`SKILL.md` folders) for people who let an AI agent — Hermes Agent (Nous Research) or Claude Code — run real business operations, not just write code.

They come from AUTOMAIT. AUTOMAIT is a small Australian business: a sole-trader founder working with an AI operator. The founder owns the business, sets direction and is accountable for it, including ID checks, accounts and payouts; the AI operator does most of the day-to-day work. These are the patterns we use day to day, with the mistakes we've made or planned for written in as pitfalls.

| Skill | What it gives your agent |
|---|---|
| `venture-workspace` | A file layout (charter, ledger, backlog, worklog, needs-human list) so the agent stays consistent across sessions and cron runs |
| `stripe-payment-link-product-launch` | Sell a digital file with a Stripe Payment Link that redirects to a hard-to-guess download page — no store platform needed |
| `weekly-honest-pnl-report` | A weekly profit-and-loss report computed from the ledger and checked against Stripe, including weeks with $0 revenue |

## Install

**Hermes Agent:** copy each folder into your skills directory (default `~/.hermes/skills/<category>/<skill-name>/`), then start a new session. Run `hermes skills list` (or ask "list my skills") to confirm they loaded.

**Claude Code:** copy each folder into `~/.claude/skills/` (personal) or `.claude/skills/` in a project.

Each skill is plain Markdown. Read it before you use it, and change paths, currency and tax wording to fit your business.

## Limits — please read
- These are working notes from one operation, not guaranteed recipes. APIs, fees and platform rules change; check them before you rely on a step.
- Nothing here is legal, tax or financial advice. The Australian notes are general information only.
- Your agent will act with whatever access you give it. Use restricted API keys and spending caps.

## More from AUTOMAIT
These three skills are free and complete on their own. If you want more of the same kind of thing, the **AI Operator Pack** (A$39) at https://automait.au/skills/ is our paid set of skills for agent-run business operations. Buying it is optional; nothing in this repo depends on it.

AUTOMAIT · ABN 82 801 716 831 · a founder + AI operator · https://automait.au
