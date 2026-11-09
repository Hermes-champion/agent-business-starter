---
name: venture-workspace
description: Use when an agent runs a business. Sets up state files.
version: 1.0.0
author: AUTOMAIT
license: MIT
metadata:
  hermes:
    tags: [business, operations, workspace, ledger]
---

# Venture Workspace

A small, fixed set of plain files that hold everything an AI operator needs to stay consistent across sessions, sub-agents and scheduled runs. The agent forgets; the files don't.

## When to Use
- You're starting an agent-run venture (digital products, async services, a side business).
- Different sessions or cron jobs give inconsistent answers about what's been done, what's been spent or what's next.
- You want a weekly report you can trust (see `weekly-honest-pnl-report`).

Not for: general note-taking, or code projects that already have an issue tracker.

## Procedure
1. Choose one persistent root, e.g. `~/venture/`. In Docker or sandboxed setups, check the path survives a container rebuild **before** you write to it (create a file, rebuild or restart, check it's still there).
2. Create these files:

```
~/venture/
  CHARTER.md        # goal, deadline, rules the agent sets itself, strategy tracks
  business.md       # non-secret facts: trading name, business number, tax status, site, email
  ledger.csv        # every cent in or out
  TODO.md           # prioritised backlog; the work loop takes the top unchecked item
  worklog.md        # append-only: dated entries of what was done, with evidence
  NEEDS_HUMAN.md    # things only the human owner can do (ID checks, sign-ups, payouts)
  PORTFOLIO.md      # experiments with a metric and a kill date
  suppression.csv   # people who asked not to be contacted; check before any message
  products/<slug>/  # one folder per product
  reviews/          # verifier/legal review files (a gate, not a formality)
  research/         # dated research notes with source URLs
```

3. **CHARTER.md** — keep it to one screen:
   - The goal as a number and a date (e.g. "cover $X/month by YYYY-MM-DD").
   - Who does what: the agent operates; the human handles only identity, accounts and money movement.
   - Rules, for example: be openly AI-operated; no fake reviews; no cold outreach unless the owner approves a reviewed plan; spend cap per item without a logged reason; kill anything with no traction after N days; never publish the owner's personal name.
4. **ledger.csv** — header exactly:
   `date,type,category,description,amount_aud,running_balance`
   - `type` ∈ `income|expense|refund|fee|note`. Expenses are negative.
   - Log income only when money has actually arrived (for example a Stripe charge that succeeded), never on "sale expected".
   - Put an invoice or reference number in `description` so the entry can be audited. Never put card numbers or API keys there.
5. **TODO.md** — sections `## Now`, `## Next`, `## Later`; markers `[ ]` todo, `[~]` in progress, `[x]` done (add the date and evidence), `[!]` blocked (say why and who unblocks it).
6. **worklog.md** — append, never rewrite. One line per action: `- YYYY-MM-DD HH:MM <what> — evidence: <path|URL|command output>`.
7. Start every session and every cron prompt with: "Read CHARTER.md, the top of TODO.md and the last 30 lines of worklog.md before acting."

## Pitfalls
- **Secrets in state files.** API keys go in the agent's env/secret store, logins in a vault — never in `business.md` or the ledger. Grep before sharing anything: `grep -rniE 'sk_live|rk_live|ghp_|password|BEGIN PRIVATE' ~/venture`.
- **Running balance drift.** Don't let the model add up the ledger in its head. Recompute with code (see the verification step below).
- **"Done" without evidence.** Any `[x]` with no file, URL or command output behind it should be treated as not done.
- **Blocking on the human.** Write human-only tasks to `NEEDS_HUMAN.md`, then move on to the next item. Surface the list in the daily or weekly report.
- **Owner's identity leaking.** If the owner wants to stay private, keep their name out of every public file and grep for it before anything ships.

## Verification
```bash
cd ~/venture && ls CHARTER.md business.md ledger.csv TODO.md worklog.md NEEDS_HUMAN.md
python3 - <<'PY'
import csv; bal=0
for r in csv.DictReader(open('ledger.csv')):
    bal += float(r['amount_aud'] or 0)
    if abs(bal - float(r['running_balance'] or 0)) > 0.005:
        print('DRIFT at', r['date'], r['description'], 'expected', round(bal,2))
print('balance', round(bal,2))
PY
```
Pass = all files exist, no DRIFT lines, and the secrets grep prints nothing.
