# Vulny Agent Scan — MCP skill

Give your AI agent the ability to run **real vulnerability scans** (nmap + nuclei + CVE matching) on the domains and IPs you own — directly from the conversation. Vulny exposes its scanner as a **Model Context Protocol (MCP)** server, so agents in Claude, Cursor and other MCP-compatible clients can scan, track progress and return a prioritised report.

Two ways to use it:

- **Already subscribed on [vulny.app](https://vulny.app)?** Generate an agent token in the dashboard (Settings → API → *AI agent (MCP)*) and run **unlimited scans** on your subscription — no credits, no extra charge. The agent scans the targets you've added in your dashboard.
- **No account?** Pay-per-scan with prepaid credits (Stripe) — just `register` with your company email and go.

- Website & docs: https://vulny.app/p/agent-scan
- Operated by **Vulny SIA** (Latvia, reg. No. 40203753831)

## Connect

Add the Vulny MCP server to your MCP-compatible client:

- **MCP endpoint:** `https://agent-api.vulny.app/mcp`
- **Auth header:** `Authorization: Bearer <YOUR_API_KEY>`

A plain REST API mirrors every tool at `https://agent-api.vulny.app` (same auth) if you prefer raw HTTP.

### Get an API key

1. Call the **`register`** tool with your **company email** (public providers like gmail are rejected; the email domain must match the domain you will scan).
2. Vulny emails a one-time link; open it to reveal your API key.
3. Put the key in your MCP connector config as the Bearer token.

## How it works

Just call **`run_scan(target)`** and follow the returned `status`:

| status | what to do |
|---|---|
| `verification_required` | prove you own the domain (DNS TXT, `.well-known` file, or emailed link), then call `run_scan` again |
| `payment_required` | open the `checkout_url` (or buy a package with `buy_credits`), then call `run_scan` again |
| `started` | poll `get_scan_status(scan_id)`, then `get_scan_report(scan_id, format)` |

You may only scan assets you own or are authorised to test — verification and the email‑domain match enforce this.

## Tools

`register` · `recover_key` · `run_scan` · `get_scan_status` · `get_scan_report` (table / json / pdf) · `buy_credits` · `get_balance` · `manage_billing` · `export_data` · `delete_data`

See **[SKILL.md](SKILL.md)** for the full instructions, the complete tool reference, pricing, rules and support details.

## Pricing

Pay-per-scan with prepaid credits — **1 credit = 1 scan**. Connect first, then the current package prices are returned live by the tools (`buy_credits` / `run_scan`), so you always see up-to-date pricing in the conversation. A credit is refunded automatically if a scan fails on Vulny's side.

## Support

- Email: **support@vulny.app**
- Contact: https://vulny.app/contact
- Terms: https://vulny.app/legal/terms · Privacy: https://vulny.app/legal/privacy · Acceptable Use: https://vulny.app/legal/acceptable-use
