# Gurkerl Meal Agent

A Claude Code skill that turns your weekly meal picks into a Gurkerl shopping cart. No app, no overhead. Just pick your dishes, the agent handles the rest.

## Product Preferences

When searching and selecting products, follow these rules in order:
1. **Prefer Miil brand** when available
2. **Prefer BIO products** when available
3. Fall back to `favorite-products.md` for specific brand preferences per ingredient
4. If none of the above match, pick the most common/popular option and confirm with the user

## How It Works

1. User picks dishes from `meals/` (each meal is an .md file with ingredients)
2. Agent loads `pantry.md` (stuff that's always at home) and `weekly-staples.md` (recurring weekly order)
3. Agent diffs: what's needed vs. what's at home
4. Agent calls Gurkerl MCP to search products, using `favorite-products.md` for preferred brands/variants
5. Agent adds missing items to cart

## First Time Setup

When a new user opens this project for the first time, walk them through this onboarding flow:

### Step 1: Configure Gurkerl MCP

Check if the Gurkerl MCP server is already connected by trying a product search.

If not configured yet, tell the user to run:

```bash
claude mcp add-json gurkerl --scope user '{
  "type": "stdio",
  "command": "mcp-remote",
  "args": [
    "https://mcp.gurkerl.at/mcp",
    "--transport", "http-only",
    "--header", "rhl-email: youremail@example.com",
    "--header", "rhl-pass: YOUR_PASSWORD"
  ]
}'
```

They need to replace the email and password with their Gurkerl credentials, then restart Claude Code.

### Verifying the MCP Connection

After setup, verify the connection actually works. This is important because auth errors can be subtle:

1. **Don't trust `get_all_user_favorites` or `get_user_info` as a connection test.** These endpoints return 401/null even with valid credentials when the account has no order history. This is misleading — it looks like broken auth but it's not.
2. **Use `batch_search_products` as the reliable connection test.** Search for something simple like `{"queries": [{"keyword": "Milch"}]}`. If this returns products, the MCP connection and auth are working.
3. **`get_typical_order` and `fetch_orders` return empty for new accounts** — this is expected, not an auth failure.

Summary of endpoint reliability for connection testing:
- `batch_search_products` — always works if auth is valid (use this)
- `get_all_user_favorites` — returns 401 on accounts with no favorites (misleading)
- `get_user_info` — returns null data on some accounts (misleading)
- `fetch_orders` / `get_typical_order` — empty on new accounts (expected)

### Step 2: Pull Favorite Products from Gurkerl

The repo ships with example data in `favorite-products.md` from another user's order history. **New users must replace this with their own favorites.**

1. Check if `favorite-products.md` still contains the example/dummy data (look for brands like Miil Topfen, Garofalo Spaghetti, etc. that aren't the user's own). If it does, tell the user this is example data and offer to replace it.
2. Try `get_all_user_favorites` and `fetch_orders` to pull the user's order history / favorites
3. If data is available: present the most frequently ordered items, ask the user to confirm which are true favorites (recurring buys vs. one-time purchases)
4. If endpoints return empty/401 (common for new accounts): skip auto-population and either ask the user to list their preferred products manually, or search Gurkerl for common staples and let them pick
5. Replace `favorite-products.md` with confirmed favorites, including the exact Gurkerl product name so future searches match perfectly

This step is important because the agent relies on `favorite-products.md` to pick the right brand/variant when ordering. Without it, the agent has to guess or ask every time.

### Step 3: Customize Kitchen Files

The repo ships with example data in `pantry.md`, `weekly-staples.md`, and `meals/`. **New users should review and replace these with their own.**

Walk the user through:
1. `pantry.md` — check if it still has the example data. Ask the user what they actually have at home (oils, spices, grains, etc.) and update accordingly
2. `weekly-staples.md` — check if it still has example data. Ask the user what they order every week regardless of meals, or pull from their order history if available
3. `meals/` — the example meals can stay as inspiration, but encourage adding their own standard dishes (use existing meals as format reference)

## Directory Structure

```
meals/           — one .md file per standard dish (name + ingredients)
pantry.md        — always-at-home items (oil, salt, spices, rice, pasta...)
weekly-staples.md — items ordered every week regardless of meals
favorite-products.md — preferred Gurkerl products (brand, size, variant)
SKILL.md         — the Claude Code skill definition
.claude/settings.template.json — MCP config template (committed)
.claude/settings.json — MCP config with real credentials (gitignored)
```

## Adding New Meals

Users can add meals by sharing a recipe URL or pasting ingredients. The agent:

1. Fetches/parses the recipe (name, ingredients, quantities)
2. Auto-classifies effort (niedrig/mittel/hoch) based on ingredient count, cooking time, complexity
3. Asks the user to confirm the classification
4. Saves as a new `.md` file in `meals/`

This means the meal library grows naturally over time. See SKILL.md for the full classification criteria.

## Usage

Once setup is complete:

```
Pick meals: Käsespätzle, Thai Green Curry, Shakshuka
→ Agent reads meal files, diffs against pantry
→ Agent searches Gurkerl for missing items (prefers favorites)
→ Agent adds to cart
→ User reviews and orders
```

## Requirements

- Gurkerl account (gurkerl.at)
- Gurkerl MCP server configured (email + password auth)
- Claude Code with MCP support
