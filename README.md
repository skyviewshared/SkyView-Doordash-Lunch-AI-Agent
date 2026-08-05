# SkyView Lunch Order Agent

A Claude Code custom agent that automates DoorDash group orders. Tell it what everyone wants, give it a Group Order link, and it adds every item to the cart — ready for your review before checkout.

**Two interchangeable backends:**

| Backend | Speed | Platform | Requirements |
|---------|-------|----------|--------------|
| **DoorDash CLI** (`dd-cli`) — preferred | ~30–60s per group order | macOS (Apple Silicon) | [Beta-approved account](https://github.com/doordash-oss/doordash-cli) |
| **Browser automation** — fallback | ~2–7 min | Windows / macOS / Linux | Chrome + Claude in Chrome extension |

The agent auto-detects `dd-cli` and uses it when available; otherwise it drives DoorDash through Chrome. Either way, **it never submits an order without your explicit confirmation.**

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

### Optional: enable the fast dd-cli backend (macOS Apple Silicon)

1. Get beta access to the [DoorDash CLI](https://github.com/doordash-oss/doordash-cli) (waitlist approval required)
2. Download the latest `dd-cli-v<version>-darwin-arm64.tar.gz` from the [Releases page](https://github.com/doordash-oss/doordash-cli/releases), verify the SHA256 checksum, and extract per the bundled `quickstart.txt`
3. Make sure `dd-cli` is on your `PATH`, then run `dd-cli login`
4. Done — the agent detects it automatically and switches to CLI ordering (browser stays as fallback)

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

Review the summary, reply **"confirm"**, and the agent hands off to checkout. Payment and final submission are always done by you.

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
| `skills/lunch-order/SKILL.md` | Skill trigger — install the folder as `~/.claude/skills/lunch-order/` |

---

## Requirements

| Tool | Version |
|------|---------|
| Claude Code | Latest |
| Claude in Chrome MCP | Latest |
| Chrome | Any modern version |
| DoorDash | Active account required |

---

## Author

Built and maintained by **Srotriyo Sengupta**, SkyView Investment Advisors.

(c) 2026 SkyView Investment Advisors. All rights reserved.
