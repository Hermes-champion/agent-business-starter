---
name: stripe-payment-link-product-launch
description: Use when selling a digital file via Stripe Payment Link.
version: 1.0.0
author: AUTOMAIT
license: MIT
metadata:
  hermes:
    tags: [stripe, payments, digital-product, launch]
---

# Stripe Payment Link → Digital Download

Sell a PDF, template or zip with no store platform. Stripe hosts checkout; once the buyer pays, Stripe redirects them to a download page with a hard-to-guess URL on your static site.

## When to Use
- You have a finished digital file and a static site (GitHub Pages, Netlify, Cloudflare Pages, etc.).
- Low volume, one fixed price, no accounts or licence keys needed.

Not for: subscriptions with access control, high-piracy-risk products, or cases where you need a merchant of record to handle overseas VAT/GST for you (look at Lemon Squeezy, Paddle or Gumroad instead).

## Prerequisites
- A Stripe account activated by the account owner (the human does KYC; agents can't).
- A **restricted** key (`rk_live_...`) given to the agent via its env/secret store, with: Products write, Prices write, Payment Links write, Charges/Checkout Sessions read, Balance read. Don't give it payout or bank-account permissions.
- Check the key from the agent's own environment before anything else: `curl -s https://api.stripe.com/v1/balance -u "$STRIPE_API_KEY:"` should return JSON with `"object": "balance"`.

## Procedure
1. **Delivery page first.**
   - Slug: `python3 -c "import secrets;print(secrets.token_urlsafe(16))"`.
   - Create `/d/<slug>/index.html` with the download link(s), a thank-you note, support contact and your business identity.
   - Add `<meta name="robots" content="noindex,nofollow">`. Never link it from anywhere public, and leave it out of `sitemap.xml`.
   - Deploy, then check `curl -sI https://<site>/d/<slug>/` returns 200 **over HTTPS**.
2. **Product + price** (amount in cents):
```bash
P=$(curl -s https://api.stripe.com/v1/products -u "$STRIPE_API_KEY:" -d name="<Product name>" -d description="<one line>" | python3 -c 'import sys,json;print(json.load(sys.stdin)["id"])')
PR=$(curl -s https://api.stripe.com/v1/prices -u "$STRIPE_API_KEY:" -d product=$P -d unit_amount=2900 -d currency=aud | python3 -c 'import sys,json;print(json.load(sys.stdin)["id"])')
```
3. **Payment link with redirect:**
```bash
curl -s https://api.stripe.com/v1/payment_links -u "$STRIPE_API_KEY:" \
  -d "line_items[0][price]=$PR" -d "line_items[0][quantity]=1" \
  -d "after_completion[type]=redirect" \
  -d "after_completion[redirect][url]=https://<site>/d/<slug>/"
```
   Save the returned `id` (`plink_...`) and `url` (`https://buy.stripe.com/...`) in `products/<slug>/stripe.md`. This file is not secret, but keep the delivery slug private.
4. **Buy button** on the public product page: a plain `<a href>` to the payment-link URL. The price shown must match the Stripe price exactly and be the total price the buyer pays.
5. **Log** in the worklog: product id, price id, link id. Don't add a ledger entry until money actually arrives.

## Pitfalls
- **HTTP redirect target.** If your site's TLS certificate isn't issued yet, the redirect lands on an insecure page or fails. Ship only after HTTPS works. On GitHub Pages a stuck certificate can often be fixed by removing the custom domain and adding it again.
- **Editing prices.** Stripe prices can't be changed. Create a new price and a new payment link, deactivate the old link (`-d active=false`), and update the button.
- **Unguessable ≠ secure.** Anyone with the URL can download. Say this honestly in your own notes; it's fine for low-priced files. Rotate the slug if it leaks.
- **Bot-blocked dashboard.** The Stripe dashboard login may show a CAPTCHA to browser agents. Work through the API with the restricted key, and never try to solve CAPTCHAs.
- **Tax wording.** If you are not registered for GST/VAT, don't call Stripe receipts "tax invoices". Check what your business must show (business number, name) in your jurisdiction.
- **Test mode vs live mode.** `sk_test`/`rk_test` objects are invisible in live mode. Make sure the key prefix matches the mode you mean to use.

## Verification
```bash
curl -s https://api.stripe.com/v1/payment_links/<plink_id> -u "$STRIPE_API_KEY:" | python3 -c 'import sys,json;d=json.load(sys.stdin);print(d["active"],d["url"],d["after_completion"])'
curl -s "https://api.stripe.com/v1/payment_links/<plink_id>/line_items" -u "$STRIPE_API_KEY:" | python3 -c 'import sys,json;[print(i["price"]["unit_amount"],i["price"]["currency"]) for i in json.load(sys.stdin)["data"]]'
curl -sI https://<site>/d/<slug>/ | head -1        # expect HTTP/2 200
curl -s https://<site>/d/<slug>/ | grep -c noindex   # expect >=1
```
Pass = link active, correct amount and currency, redirect URL is https and returns 200, and the button on the public page points to the same `buy.stripe.com` URL.
