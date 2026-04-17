# Gurkerl Meal Agent

A Claude Code skill that turns your weekly meal picks into a Gurkerl shopping cart. No app, no overhead. Just pick your dishes, the agent handles the rest.

## How It Works

1. User picks dishes from `meals/` (each meal is an .md file with ingredients)
2. Agent loads `pantry.md` (stuff that's always at home) and `weekly-staples.md` (recurring weekly order)
3. Agent diffs: what's needed vs. what's at home
4. Agent calls Gurkerl MCP to search products, using `favorite-products.md` for preferred brands/variants
5. Agent adds missing items to cart

## Directory Structure

```
meals/           — one .md file per standard dish (name + ingredients)
pantry.md        — always-at-home items (oil, salt, spices, rice, pasta...)
weekly-staples.md — items ordered every week regardless of meals
favorite-products.md — preferred Gurkerl products (brand, size, variant)
SKILL.md         — the Claude Code skill definition
```

## Usage

Connect the Gurkerl MCP server, then:

```
Pick meals: Käsespätzle, Thai Green Curry, Shakshuka
→ Agent reads meal files, diffs against pantry
→ Agent searches Gurkerl for missing items (prefers favorites)
→ Agent adds to cart
→ User reviews and orders
```

## Requirements

- Gurkerl MCP server configured (email + password auth)
- Claude Code with MCP support
