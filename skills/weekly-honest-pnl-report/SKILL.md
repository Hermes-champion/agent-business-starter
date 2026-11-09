---
name: weekly-honest-pnl-report
description: Use for a weekly P&L from ledger + Stripe, incl. $0 weeks.
license: MIT
metadata:
  author: AUTOMAIT
  version: "1.0.0"
  hermes:
    tags: [finance, reporting, ledger, stripe, cron]
    related_skills: [venture-workspace]
---

# Weekly Honest P&L Report

A short weekly report computed by code from `ledger.csv` and checked against Stripe. It is written to be true, not encouraging. A week with zero revenue is reported as zero.

## When to Use
- Every week (for example Monday morning) for any agent-run venture that uses the `venture-workspace` layout.
- Whenever the owner asks "how are we going?" — run this rather than answering from memory.

## Procedure
1. **Pull Stripe** (read-only, restricted key) for the period — charges that succeeded and refunds:
```bash
SINCE=$(date -d '7 days ago' +%s)
curl -s -G https://api.stripe.com/v1/charges -u "$STRIPE_API_KEY:" -d "created[gte]=$SINCE" -d limit=100 > /tmp/charges.json
curl -s -G https://api.stripe.com/v1/balance_transactions -u "$STRIPE_API_KEY:" -d "created[gte]=$SINCE" -d limit=100 > /tmp/bt.json
```
2. **Reconcile.** Every succeeded, unrefunded charge should match a ledger `income` row (same amount and date ±1 day). Stripe fees come from `balance_transactions[].fee` — log them as `fee` rows. List any mismatch in both directions. Don't silently "fix" the ledger: add the missing rows with a `reconciled from Stripe <id>` note.
3. **Compute with code, not by eye:**
```python
import csv, datetime as dt
rows=list(csv.DictReader(open('ledger.csv')))
end=dt.date.today(); start=end-dt.timedelta(days=7)
wk=[r for r in rows if start < dt.date.fromisoformat(r['date']) <= end]
s=lambda rs,t: round(sum(float(r['amount_aud']) for r in rs if r['type']==t),2)
inc,exp,fee,ref = s(wk,'income'), s(wk,'expense'), s(wk,'fee'), s(wk,'refund')
net = round(inc+exp+fee+ref,2)
total = round(sum(float(r['amount_aud']) for r in rows),2)
print(dict(income=inc, expenses=exp, fees=fee, refunds=ref, net_week=net, net_to_date=total))
```
4. **Write the report** (bullets, no tables if it's going to chat apps):
   - Period, income, expenses, fees, refunds, net for the week, net to date, progress against the charter goal (`net_to_date / goal`).
   - Sales count and source of each sale, if known. Say "unknown" if it isn't.
   - What shipped (from `worklog.md`, with evidence links only).
   - What was killed or paused, and why (kill rule).
   - Human-only blockers from `NEEDS_HUMAN.md`.
   - One line: the single most important thing for next week.
5. Save to `reports/pnl-YYYY-MM-DD.md` and deliver it to the owner's channel.

## Scheduling (Hermes)
Create a cron job, but list the existing jobs first so you don't create a duplicate. Example prompt: "Load skill weekly-honest-pnl-report. Working dir ~/venture. Produce and deliver this week's report." Schedule: Mondays 09:00 local time. Deliver it to the owner's thread, not a public channel.

## Pitfalls
- **Optimism leak.** Words like "momentum", "promising" or "nearly there" with no number behind them. Ban them. Report the numbers, then one line of honest interpretation.
- **Counting pending or failed charges.** Count only `status=succeeded` with `refunded=false` (handle partial refunds via `amount_refunded`).
- **Gross vs net.** Stripe payouts are net of fees. If you report gross revenue, list fees on their own line.
- **Currency.** If a charge isn't in your ledger currency, use the converted `balance_transaction.amount`, not the charge amount.
- **Timezone.** Stripe timestamps are UTC. Your week boundary should be in local time; note which one you used.
- **Paging.** `has_more=true` means there are more results. Page with `starting_after` before you trust the totals.

## Verification
- The report's net-to-date equals the ledger's last `running_balance` (also checked by the `venture-workspace` drift script).
- Reconciliation lists zero unexplained mismatches, or names each one.
- Every "shipped" claim links to a file, URL or command output.
