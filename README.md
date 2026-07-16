# SkyView Lunch Order Agent

A Claude Code custom agent that automates DoorDash group orders from your browser. Tell it what everyone wants, give it a Group Order link, and it adds every item to the cart — ready for your review before checkout.

---

## Prerequisites

1. **Claude Code** — [Install here](https://claude.ai/code)
2. **Claude in Chrome** MCP extension — install from the Claude Code MCP marketplace, then connect it to your Chrome browser
3. A **DoorDash account** with a saved delivery address
4. Chrome open with DoorDash accessible (you must be logged in)

---

## Installation

1. Clone or download this repo
2. Copy `lunch-order-agent.md` into your Claude Code agents folder:
   - **Mac/Linux:** `~/.claude/agents/`
   - **Windows:** `C:\Users\<YourName>\.claude\agents\`
3. Copy `lunch-order.md` into your Claude Code skills folder:
   - **Mac/Linux:** `~/.claude/skills/`
   - **Windows:** `C:\Users\<YourName>\.claude\skills\`
4. Restart Claude Code

That's it. No API keys, no config files, no environment variables.

---

## How to use

### Step 1 — Create a DoorDash Group Order link

1. Open DoorDash in Chrome and navigate to your restaurant
2. Click **Group Order** (next to the cart icon)
3. Click **Start Group Order** → copy the `drd.sh` share link

### Step 2 — Collect orders

Gather what everyone wants (Teams message, Slack, email, verbally — anything works).

### Step 3 — Run the agent

Open Claude Code and say something like:

> *"Order lunch from [restaurant]. Here are the orders: [paste orders]. Group order link: https://drd.sh/cart/..."*

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
- The agent works with any DoorDash restaurant, not just ones listed in this repo.

---

## Files

| File | Purpose |
|------|---------|
| `lunch-order-agent.md` | Agent definition — install in `~/.claude/agents/` |
| `lunch-order.md` | Skill trigger — install in `~/.claude/skills/` |

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

© 2026 SkyView Investment Advisors. All rights reserved.
