# Skill: lunch-order

## Trigger
User says something like "collect lunch orders", "parse lunch orders from Teams", "place our lunch order", or "order lunch from DoorDash".

## What this skill does
1. Searches Teams chat for recent lunch order messages
2. Parses them into a structured list (person → items)
3. Asks the user to confirm the order list and the DoorDash restaurant URL
4. Uses Claude in Chrome to open DoorDash, add each item to the cart, and pause before final checkout for user confirmation

---

## Step 1 — Find lunch order messages in Teams

Use `chat_message_search` (MCP tool `mcp__1fad1557-69fb-4e80-b8cb-5f63fa03c790__chat_message_search`) to search recent Teams messages.

Search query: `"lunch order"` OR `"I'll have"` OR `"can I get"` OR `"order:"` — try a few if the first returns nothing.

If the user specifies a chat name or channel, include it in the search. Otherwise search broadly across recent messages from today.

## Step 2 — Parse orders

From the raw messages, extract a clean order list in this format:

```
Person: [name or Teams display name]
Items: [comma-separated list of food items]
Notes: [any special instructions, e.g. "no onions"]
```

If a message is ambiguous, include it as-is under an "Unclear" entry and flag it to the user.

Show the parsed order list to the user and ask:
- "Does this look right? Any corrections?"
- "What's the DoorDash restaurant URL or restaurant name to order from?"
- "What's the delivery address?" (if not already known)

Wait for confirmation before proceeding.

## Step 3 — Open DoorDash in Chrome

Use `mcp__Claude_in_Chrome__navigate` to open the DoorDash restaurant page the user provided.

If only a restaurant name was given (no URL), navigate to `https://www.doordash.com` and use `mcp__Claude_in_Chrome__find` to search for the restaurant by name, then navigate to its menu page.

## Step 4 — Add items to cart

For each person's order:
1. Use `mcp__Claude_in_Chrome__find` to locate the menu item on the page
2. Use `mcp__Claude_in_Chrome__javascript_tool` or `mcp__Claude_in_Chrome__form_input` to add it to the cart
3. If there are notes (e.g. "no onions"), look for a customization/special instructions field and fill it in
4. If an item is not found on the menu, flag it to the user immediately with: "⚠️ Could not find '[item]' for [person] — please confirm the item name or choose a substitute."

After all items are added, take a screenshot with `mcp__Claude_in_Chrome__preview_screenshot` and show it to the user with the full cart summary.

## Step 5 — Final confirmation before checkout

Show the user:
- Cart contents (items + prices if visible)
- Total cost
- Delivery address

Ask: "Ready to place this order? Type 'confirm' to proceed or 'cancel' to stop."

Only proceed to checkout after explicit user confirmation. Do NOT click "Place Order" or submit payment without this confirmation.

If confirmed, proceed through DoorDash checkout. If a login or payment step requires credentials the agent doesn't have, pause and ask the user to complete that step manually in the browser.

## Error handling
- If Teams search returns no results: ask the user to specify the chat name, date range, or paste the orders directly
- If DoorDash shows a CAPTCHA or login wall: notify the user and ask them to resolve it in the browser, then continue
- If an item is unavailable or the restaurant is closed: stop and notify the user before attempting further actions
