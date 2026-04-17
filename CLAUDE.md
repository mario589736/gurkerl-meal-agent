# Gurkerl Meal Agent

A Claude Code skill that turns your weekly meal picks into a Gurkerl shopping cart. No app, no overhead. Just pick your dishes, the agent handles the rest.

## How It Works

1. User picks dishes from `meals/` (each meal is an .md file with ingredients)
2. Agent loads `pantry.md` (stuff that's always at home) and `weekly-staples.md` (recurring weekly order)
3. Agent diffs: what's needed vs. what's at home
4. Agent calls Gurkerl MCP to search products, using `favorite-products.md` for preferred brands/variants
5. Agent adds missing items to cart

## First Time Setup

When a new user opens this project for the first time, walk them through this onboarding flow:

### Step 1: Configure Gurkerl MCP

Check if `.claude/settings.json` exists with real credentials (not placeholder values).

If not configured yet:
1. Tell the user to copy the template: `cp .claude/settings.template.json .claude/settings.json`
2. Ask them to fill in their Gurkerl email and password in `.claude/settings.json`
3. Tell them to restart Claude Code so the MCP server connects

### Step 2: Pull Favorite Products from Gurkerl

Once the MCP connection is live, help the user bootstrap their `favorite-products.md`:

1. Use the Gurkerl MCP to pull the user's order history / previously ordered products
2. Present the most frequently ordered items
3. Ask the user to confirm which ones are true favorites (recurring buys vs. one time purchases)
4. Update `favorite-products.md` with confirmed favorites, including the exact Gurkerl product name so future searches match perfectly

This step is important because the agent relies on `favorite-products.md` to pick the right brand/variant when ordering. Without it, the agent has to guess or ask every time.

### Step 3: Customize Kitchen Files

Walk the user through:
1. `pantry.md` — review the defaults, remove what they don't have, add what's missing
2. `weekly-staples.md` — adjust to their actual recurring order
3. `meals/` — add their standard dishes (use existing meals as format reference)

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
