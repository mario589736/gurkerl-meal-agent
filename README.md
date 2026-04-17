# Gurkerl Meal Agent

Pick your meals for the week. The agent figures out what to buy.

## How it works

1. You have a folder of your standard dishes (`meals/`) with ingredients
2. You have a pantry list (`pantry.md`) of stuff that's always at home
3. You have weekly staples (`weekly-staples.md`) that get ordered every time
4. You have favorite Gurkerl products (`favorite-products.md`) so the agent picks the right brands

You tell the agent what you want to eat this week. It diffs, maps to Gurkerl products, and fills your cart.

Each meal has an `Aufwand` tag (niedrig/mittel/hoch). Ask the agent to "plan my week" and it picks easy meals for weekdays, more complex ones for weekends.

## Setup

### 1. Clone and configure the Gurkerl MCP server

```bash
git clone https://github.com/mario589736/gurkerl-meal-agent.git
cd gurkerl-meal-agent
```

Remove any existing Gurkerl MCP config (safe to run even if none exists):

```bash
claude mcp remove gurkerl
```

Then add the Gurkerl MCP server with your credentials:

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

Replace `youremail@example.com` and `YOUR_PASSWORD` with your Gurkerl account credentials.

> This stores the config in your user-level Claude settings, so it persists across projects and stays out of git.

### 2. Verify the connection

Open Claude Code and test that the MCP connection works:

```
claude
> Search Gurkerl for "Milch"
```

If you get product results back, you're connected. 

> **Troubleshooting:** Don't use the favorites or user info endpoints to test the connection — they return 401/empty on accounts with no order history, which looks like broken auth but isn't. A product search (`batch_search_products`) is the reliable test.

### 3. Customize your kitchen

Edit these files to match your actual setup:
- `pantry.md` — what's always at home
- `weekly-staples.md` — what you order every week
- `favorite-products.md` — preferred brands at Gurkerl
- `meals/*.md` — your standard dishes

### 4. Open Claude Code and pick your meals

```bash
claude
> Diese Woche: Käsespätzle und Thai Green Curry
```

## Example

```
> Diese Woche: Käsespätzle und Thai Green Curry

Checking meals... subtracting pantry... adding weekly staples...

Einkaufsliste:
- 300g Bergkäse (Ländle Bergkäse) ← Favorit
- 3 Zwiebeln
- 1 Bund Schnittlauch
- 200g Tofu (Ja! Natürlich Tofu Natur) ← Favorit
- 1 Brokkoli
- 1 Paprika rot
- 100g Zuckerschoten
- 1 Bund Thai-Basilikum
- 2 Kaffirlimettenblätter
- 1 Limette
- Grüne Currypaste
+ Weekly Staples: Hafermilch, Bananen, Brot, Joghurt, Eier...

Soll ich das in den Gurkerl Warenkorb legen?
```

## File Structure

```
meals/                    — one file per dish
  kaesespaetzle.md
  thai-green-curry.md
  shakshuka.md
  ...
pantry.md                 — always at home, never order
weekly-staples.md         — order every week
favorite-products.md      — preferred brands at Gurkerl
SKILL.md                  — Claude Code skill definition
CLAUDE.md                 — project instructions
```

## Other MCP Clients

The meal files, pantry, and favorites are plain markdown. They work with any MCP client that connects to Gurkerl:

- **Claude Desktop:** Add the Gurkerl MCP server to your `claude_desktop_config.json` ([setup guide](https://www.gurkerl.at/seite/gurkerl-mcp-server))
- **ChatGPT with MCP support:** Same server URL and auth headers
- **Any MCP-compatible client:** The Gurkerl MCP endpoint is `https://mcp.gurkerl.at/mcp/`

The `CLAUDE.md` and `SKILL.md` are optimized for Claude Code, but the data files are universal.

## Add Your Own Meals

**From a URL:** Paste a recipe link (chefkoch.de, food blogs, etc.) and the agent extracts ingredients, creates the meal file, and auto-classifies effort level.

**From text:** Paste ingredients or describe the dish. The agent parses and saves it.

**Manually:** Create a new `.md` file in `meals/`:

```markdown
# Dish Name

Portionen: 2
Aufwand: niedrig | mittel | hoch

## Zutaten

- 200g Ingredient
- 1 Other Ingredient

## Notizen (optional)

Prep tips.
```

Effort levels: **niedrig** (max 5 ingredients, 20 min, one-pot), **mittel** (6-10 ingredients, 20-45 min), **hoch** (10+ ingredients, 45+ min, multi-component).

## License

MIT
