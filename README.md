# SkyView Lunch Order Agent

A Claude Code custom agent that automates DoorDash group orders. Tell it what everyone wants, give it a Group Order link, and it adds every item to the cart — ready for your review before checkout.

**Two interchangeable backends:**

| Backend | Speed | Platform | Requirements |
|---------|-------|----------|--------------|
| **DoorDash CLI** (`dd-cli`) — preferred | **1–2 min** (sometimes under 1 min) | macOS (Apple Silicon), Linux x86_64, Windows via WSL2 | [Beta-approved account](https://github.com/doordash-oss/doordash-cli) + dd-cli **v0.2.6 or later** |
| **Browser automation** — fallback | ~2–7 min | Windows / macOS / Linux | Chrome + Claude in Chrome extension |

The agent auto-detects `dd-cli` and uses it when available; otherwise it drives DoorDash through Chrome. Either way, **it never submits an order without your explicit confirmation.**

CLI timings are measured on real multi-person orders (macOS, August 2026).

---

## Prerequisites

1. **Claude Code** — [Install here](https://claude.ai/code)
2. **Claude in Chrome** MCP extension — install from the Claude Code MCP marketplace, then connect it to your Chrome browser
3. A **DoorDash account** with a saved delivery address
4. Chrome open with DoorDash accessible (you must be logged in)
5. *(Optional)* **Microsoft 365 connector** — only needed if you want the agent to collect orders from Teams chat automatically. Without it, just paste the orders into your prompt.

---

## Installation

1. Clone or download this repo
2. Copy `lunch-order-agent.md` into your Claude Code agents folder:
   - **Mac/Linux:** `~/.claude/agents/`
   - **Windows:** `C:\Users\<YourName>\.claude\agents\`
3. Copy the `skills/lunch-order/` **folder** (containing `SKILL.md`) into your Claude Code skills folder, so you end up with `~/.claude/skills/lunch-order/SKILL.md`:
   - **Mac/Linux:** `cp -r skills/lunch-order ~/.claude/skills/`
   - **Windows:** copy the folder to `C:\Users\<YourName>\.claude\skills\lunch-order\`
4. Restart Claude Code

That's it. No API keys, no config files, no environment variables.

### Recommended: enable the fast dd-cli backend

Supported on **macOS (Apple Silicon)**, **Linux x86_64**, and **Windows via WSL2** (install it inside your WSL distro).

1. Get beta access to the [DoorDash CLI](https://github.com/doordash-oss/doordash-cli) (waitlist approval required)
2. Grab the latest release from the [Releases page](https://github.com/doordash-oss/doordash-cli/releases) — `dd-cli-v<version>-darwin-arm64.tar.gz` for macOS or `dd-cli-v<version>-linux-amd64.tar.gz` for Linux/WSL2 (`install.sh` works on both). Verify the SHA256 checksum before extracting.
3. Make sure `dd-cli` is on your `PATH`, then `dd-cli login`. For a remote or headless machine (container, CI, WSL without keychain access) either run `dd-cli login --start` / `dd-cli login --finish` (two-step remote sign-in, v0.2.6+) or run `dd-cli export-token` on a signed-in machine and set the result as `DD_CLI_ACCESS_TOKEN` — the env var takes precedence over the keychain. `export-token` also prints when the token expires.
4. Done — the agent detects it automatically and switches to CLI ordering (browser stays as fallback)

**Upgrade to v0.2.6 (October 2026).** DoorDash warns that workflows on older dd-cli versions may break, so the agent checks `dd-cli --version` on every run and asks you to upgrade if it finds anything older than 0.2.6. Upgrading is just steps 2–3 again; your login carries over.

**What changed since the v0.2.2 docs** (v0.2.3 → v0.2.6), and how the agent uses it:

| Area | Change | Used by the agent? |
|------|--------|--------------------|
| Payment | Pick a saved card with `payment-method set`, or pass `--card-id` on an ASAP order | Yes — the order summary shows which card will be charged; say "use the Amex" to switch |
| Errors | Failed commands return a categorized exit code and an `error.code` (auth, availability, timeout, rate-limit, tool) | Yes — the agent retries, re-authenticates, or falls back to the browser based on the category |
| Search | Facets: `--dashpass-only`, `--price-tier`, `--distance-preference`, `--max-eta-minutes`; `--address-id` to search from a saved address; `find-item-across-retailers` | Yes — store lookup runs from the office address |
| Cart | Weight-priced items; merchant-default modifiers with `default_handling: "exact"` to pin exact selections | Yes — exact mode when someone specifies every option |
| Orders | `order status` follows the full lifecycle (ETA, cancellation reason); `order history --include-group-order` | Yes — the agent polls status until the order is placed |
| Auth | macOS keychain keeps a refresh token (fewer re-logins); two-step remote sign-in; token expiry shown | Yes |
| DoubleDash | Discover and add eligible add-on items to an order just placed | Only if you ask after the order is in |
| Courier | Point-to-point pickup and drop-off | No — not a lunch feature |
| DoorDash for Business | `ddfb team add-members` / `remove-members`; Meal Manager calendar discovery | No — unless the firm moves lunch onto a DoorDash for Business budget |

**`--intent` is still mandatory** (since v0.2.2): every tool-backed command requires `--intent "<who + goal>"` or it errors. The agent supplies this automatically; if you script `dd-cli` yourself, remember to include it.

---

## How to use

### Step 1 — Create a DoorDash Group Order link

1. Open DoorDash in Chrome and navigate to your restaurant
2. Click **Group Order** (next to the cart icon)
3. Click **Start Group Order** → copy the `drd.sh` share link

### Step 2 — Collect orders

Gather what everyone wants (Teams message, Slack, email, verbally — anything works). If you have the Microsoft 365 connector set up, the agent can search your Teams chats for today's orders itself — otherwise just paste them.

### Step 3 — Run the agent

Open Claude Code and say something like:

> *"Order lunch from [restaurant]. Here are the orders: [paste orders]. Group order link: https://drd.sh/cart/..."*

or, with the Teams connector:

> *"Collect today's lunch orders from Teams and order from [restaurant]. Group order link: https://drd.sh/cart/..."*

The agent will:
1. Parse the orders into a structured list
2. Open DoorDash in your Chrome tab
3. Add every item to the cart, handling customizations (protein choice, bread type, temperature, toppings)
4. Present the full cart summary with prices for your review
5. **Wait for your explicit confirmation before touching checkout**

### Step 4 — Confirm and pay

Review the summary, reply **"confirm"**, and the agent hands off to checkout.

- **Browser backend:** payment and final submission are always done by you in Chrome.
- **dd-cli backend:** the summary names the saved card that will be charged (your DoorDash default unless you say otherwise). Only after you reply "confirm" does the agent run `order submit`, and it then reports the confirmation number and ETA from `order status`. The agent never adds a new card — do that in the DoorDash app.

---

## Tips

- **The agent never places an order without your explicit confirmation.** It stops at the cart review step every time.
- If an item can't be found on the menu, the agent flags it immediately and waits for a substitute before continuing.
- For best results, share the `drd.sh` Group Order link (not the restaurant URL) — it goes straight to the right cart.
- The agent works with any DoorDash restaurant, not just ones listed in this repo. (The agent file ships with a cached menu-position table for Brennan's Delicatessen — orders from other restaurants work too, they just run a one-time menu scan first.)

---

## Files

| File | Purpose |
|------|---------|
| `lunch-order-agent.md` | Agent definition — install in `~/.claude/agents/` |
| `skills/lunch-order/SKILL.md` | Skill trigger — install the folder as `~/.claude/skills` |

---

## Requirements

| Tool | Version |
|------|---------|
| Claude Code | Latest |
| DoorDash CLI (`dd-cli`) | v0.2.6 or later (optional, preferred backend) |
| Claude in Chrome MCP | Latest (browser fallback) |
| Chrome | Any modern version (browser fallback) |
| DoorDash | Active account required |

---

## Author

Built and maintained by **Srotriyo Sengupta**, SkyView Investment Advisors.

(c) 2026 SkyView Investment Advisors. All rights reserved.
