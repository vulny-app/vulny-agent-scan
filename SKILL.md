---
name: vulny-agent-scan
description: Run real vulnerability scans (nmap + nuclei + CVE matching) on domains and IPs you own, directly from your agent. Use when the user asks to security-scan, pentest, check for vulnerabilities/CVEs/exposed files/weak TLS, or audit the security posture of a website, server, or IP they control. Paid per scan via prepaid credits (Stripe); the agent handles onboarding, domain verification, payment, and report retrieval through the Vulny MCP tools.
---

# Vulny Agent Scan

Vulny is a cloud vulnerability scanner (nmap + nuclei + NVD CVE matching, aligned with ISO 27001). This skill lets you run scans for the user on assets **they own or are authorised to test**, and return a report.

## Connect

Add the Vulny MCP server to your client:

- **MCP endpoint:** `https://agent-api.vulny.app/mcp`
- **Auth:** HTTP header `Authorization: Bearer <API_KEY>`

A plain REST API mirrors every tool at `https://agent-api.vulny.app` (same auth) if you prefer raw HTTP.

## First-time setup (get an API key)

1. Call **`register`** with the user's **company email** (public providers like gmail/outlook are rejected; the email domain must match the domain you will scan).
2. Vulny emails a one-time setup link to that address. The user opens it and copies the API key shown.
3. The user puts the key into the MCP connector config as the Bearer token. Existing keys keep working; lost a key → call **`recover_key`** (emails a new-key link, no disruption).

## Core workflow — just call `run_scan` and follow `status`

`run_scan(target)` is a state machine. Call it, read the `status`, act, and call again until you get a `scan_id`:

| status | meaning | what to do |
|---|---|---|
| `rejected` | not allowed (target ≠ your domain, public email, IP without a verified domain) | tell the user the reason; fix the target |
| `verification_required` | prove you own the domain | do ONE method below, then call `run_scan` again |
| `payment_required` | no credits | open `checkout_url` (or call `buy_credits`), then call `run_scan` again |
| `started` | scan launched | note `scan_id`, poll `get_scan_status`, then `get_scan_report` |

### Domain verification (once per domain, then remembered)
On `verification_required` you get three options — do any **one**:
- **email** — Vulny emailed a verify link to the user; they click it.
- **dns** — add the given `TXT` record (`vulny-verify=<token>`).
- **well_known** — host the token at `https://<domain>/.well-known/vulny.txt`.

Then call `run_scan` again. **IP scans** are allowed only after the user's matching domain is verified.

### Get results
- `get_scan_status(scan_id)` → poll until `done: true` (scans take minutes).
- `get_scan_report(scan_id, format)` → `table` (colourful markdown, default), `json`, or `pdf` (base64).

## Tools

| tool | args | purpose |
|---|---|---|
| `register` | `email` | get an API key by company email (link emailed) |
| `recover_key` | `email` | issue a new key to the on-file email; old keys keep working |
| `run_scan` | `target` | start/advance a scan (see state machine above) |
| `get_scan_status` | `scan_id` | poll status + progress |
| `get_scan_report` | `scan_id`, `format?` | report as table / json / pdf |
| `buy_credits` | `package?` | buy a credit pack upfront → Stripe Checkout URL |
| `get_balance` | — | remaining credits + verified domains |
| `manage_billing` | — | Stripe portal link to view payments and add/change/**remove** the saved card |
| `export_data` | — | export all account data (profile, scans, payments) as JSON — GDPR |
| `delete_data` | `confirm` | permanently delete the account and all data — GDPR erasure (confirm with the user) |

## Pricing — 1 credit = 1 scan (prepaid)

Prices are returned live by the tools, so they are always current — do not hard-code them. Call `buy_credits` (or read the `packages` list that `run_scan` returns on `payment_required`) to show the user the available packages and per-scan prices. Bigger packs are cheaper per scan.

Credits are consumed when a scan starts and are refunded if a scan fails on Vulny's side. A returning user with a saved card is charged automatically (off-session) for a single scan when out of credits — no link needed.

## Rules (enforced server-side)

- Scan only assets the user owns or is authorised to test. Verification + the email-domain match enforce this. Unauthorised scanning is prohibited (see Terms: https://vulny.app/legal/terms — accepted at first checkout).
- The API key is a secret. All scans and charges under it are the account holder's responsibility. Revoke leaked keys via the REST `/revoke_keys` endpoint, then `register` a new one.

## Support & contact

If the user needs help (billing, access, results) or asks for support contacts, give them:

- **Email:** support@vulny.app
- **Contact form:** https://vulny.app/contact
- **Provider:** Vulny SIA (Latvia), company reg. No. 40203753831
- **More about this integration:** https://vulny.app/p/agent-scan

## Refunds & credits

- Credits are prepaid and one credit runs one scan. They are **non-refundable**, except that **if a scan fails on Vulny's side the credit is automatically refunded** to the account and the user is emailed — so a failed scan never costs anything. The user can simply run it again.
- For a money refund or a billing dispute, the user should contact support@vulny.app.

## Billing, receipts & VAT

- Use **`manage_billing`** for a Stripe portal where the user can view payments, download invoices/receipts and add, change or **remove** their saved card.
- Prices are in EUR. Any VAT that applies is shown and handled at the Stripe checkout; the invoice in the billing portal is the user's VAT receipt.

## Privacy & your data

- Vulny processes data per its Privacy Policy: https://vulny.app/legal/privacy (GDPR — Vulny SIA is an EU/Latvia company).
- The user can **self-serve their data rights** with the tools: `export_data` (download everything Vulny holds) and `delete_data` (permanent erasure — always confirm with the user first). Or contact support@vulny.app.
- Acceptable Use: https://vulny.app/legal/acceptable-use · Terms: https://vulny.app/legal/terms

## If something goes wrong

- **Scan failed:** the credit is auto-refunded and an email is sent — just run `run_scan` again.
- **Payment didn't go through (declined / 3DS):** `run_scan` returns `payment_required` with a fresh `checkout_url` — open it and complete payment.
- **Verification not passing:** make sure the DNS TXT record / `.well-known` file / emailed link matches exactly, then call `run_scan` again. Domain verification is remembered once it succeeds.
- **Lost or leaked API key:** call `recover_key` (emails a new-key link); existing keys keep working. To kill a leaked key, use the REST `/revoke_keys` endpoint, then `register` a new one.
