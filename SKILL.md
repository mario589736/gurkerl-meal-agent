# Gurkerl Meal Agent — Skill Definition

## Purpose

Turn a weekly meal selection into a ready-to-order Gurkerl shopping cart. Eliminates the mental load of figuring out what to buy. You already know what you eat — this agent handles the boring part.

## Context Files

Load these before executing:

| File | Purpose |
|---|---|
| `pantry.md` | Items always at home. Never order these. |
| `weekly-staples.md` | Items to add every week regardless of meal plan. |
| `favorite-products.md` | Preferred Gurkerl products (brand, size). Use these when searching. |
| `meals/*.md` | Individual meal files with ingredients. |

## Workflow

### Step 1: Meal Selection

User names the meals they want this week. Example:

> "Diese Woche: Käsespätzle, Thai Green Curry, Shakshuka, und Lachs mit Brokkoli"

Match each name to a file in `meals/`. If no file exists, ask the user for ingredients and offer to create the meal file for next time.

**Auto-plan mode:** If the user says something like "plan my week" or "mach mir einen Wochenplan", the agent can suggest meals automatically:
- **Weekdays (Mon-Fri):** pick meals tagged `aufwand: niedrig` (quick, easy cooking)
- **Weekends (Sat-Sun):** pick meals tagged `aufwand: mittel` or `aufwand: hoch` (more complex, fun to cook)
- Present the plan, let the user swap dishes, then proceed with the shopping list.

### Step 2: Aggregate Ingredients

- Parse all selected meal files
- Combine ingredient lists
- Deduplicate (if two meals need onions, combine the quantity)
- Add everything from `weekly-staples.md`

### Step 3: Subtract Pantry

- Load `pantry.md`
- Remove any ingredient that's covered by pantry items
- The result is the **shopping list**: only what's actually needed

### Step 4: Map to Gurkerl Products

- For each shopping list item, check `favorite-products.md` first
- If a favorite exists, use that exact product name for the Gurkerl search
- If no favorite, search Gurkerl MCP for the ingredient and present options
- Let the user confirm or pick alternatives for new items
- Offer to save new picks to `favorite-products.md`

### Step 5: Add to Cart

- Use Gurkerl MCP to add all mapped products to cart
- Show a summary: what was added, total item count
- User reviews in Gurkerl app/website and completes the order

## Adding New Meals (Recipe Import)

The user can add new meals in three ways:

### From a URL

User shares a recipe link (e.g. from a food blog, chefkoch.de, etc.):

1. Fetch the URL and extract: dish name, ingredients with quantities, prep steps
2. Classify the effort level automatically based on these criteria:
   - **niedrig:** max 5 ingredients, max 20 min active cooking, one-pot/one-pan, no special techniques
   - **mittel:** 6-10 ingredients, 20-45 min active cooking, multiple steps but straightforward
   - **hoch:** 10+ ingredients, 45+ min active cooking, multiple components, special techniques (dough making, slow braising, etc.)
3. Present the parsed result to the user for confirmation
4. Save as a new `.md` file in `meals/` using the standard format
5. Confirm the Aufwand classification with the user ("Ich würde das als mittel einstufen, passt das?")

### From pasted text

User pastes ingredients or a full recipe directly into the chat. Same flow: parse, classify effort, confirm, save.

### Manual

User just names a dish and lists ingredients. Agent creates the file.

### Effort Classification Rules

When classifying Aufwand, consider:

| Factor | niedrig | mittel | hoch |
|---|---|---|---|
| Ingredients | max 5 | 6-10 | 10+ |
| Active cooking time | max 20 min | 20-45 min | 45+ min |
| Steps | 1-3 | 4-6 | 7+ |
| Techniques | basic (boil, fry, bake) | moderate (sauce making, marinating) | advanced (dough, braising, multi-component) |
| Parallel tasks | none | some | requires timing multiple things |

If in doubt, round up. Better to be pleasantly surprised than stressed on a Tuesday.

## Rules

- Never order pantry items. The whole point is to avoid buying what's already there.
- Always prefer favorite products over generic search results.
- When quantities are ambiguous (e.g. "Zwiebeln" without a count), default to reasonable household quantities (2-3 onions, 1 bunch of herbs, etc.)
- If a meal file lists an ingredient with a specific quantity, respect it.
- When creating new meal files, use the same format as existing ones.
- Keep it conversational. This is a kitchen assistant, not an enterprise tool.

## Meal File Format

Each meal file in `meals/` follows this structure:

```markdown
# [Meal Name]

Portionen: [number]
Aufwand: [niedrig | mittel | hoch]

## Zutaten

- [quantity] [ingredient]
- [quantity] [ingredient]
- ...

## Notizen (optional)

Any prep notes, variations, or tips.
```

## Example Interaction

**User:** Diese Woche mach ich Käsespätzle und Shakshuka.

**Agent:**
Reading meal files...

Käsespätzle braucht: 400g Mehl, 4 Eier, 300g Bergkäse, 3 Zwiebeln, Butter, Schnittlauch
Shakshuka braucht: 6 Eier, 2 Dosen Tomaten, 2 Paprika, 1 Zwiebel, Feta, Kreuzkümmel, Petersilie

Aus der Vorratskammer hast du schon: Mehl, Butter, Kreuzkümmel, Olivenöl, Salz, Pfeffer

Weekly Staples dazu: Hafermilch, Bananen, Brot, Joghurt

**Einkaufsliste für Gurkerl:**
- 10 Eier (Freiland, Ja! Natürlich) ← Favorit
- 300g Bergkäse (Ländle Bergkäse) ← Favorit
- 4 Zwiebeln
- 1 Bund Schnittlauch
- 2 Dosen gehackte Tomaten (Mutti Polpa) ← Favorit
- 2 Paprika rot
- 200g Feta
- 1 Bund Petersilie
- 1l Hafermilch (Oatly Barista) ← Favorit
- 6 Bananen
- 1 Brot (Ölz Butterbrot) ← Favorit
- 500g Joghurt (Ja! Natürlich Naturjoghurt) ← Favorit

Soll ich das in den Gurkerl Warenkorb legen?
