# Channel Ceiling walkthrough — the onboarding interview

The Channel Ceiling is the set of HTML tags the customer's sales channel accepts. Templates are authored to the most ambitious form the ceiling allows and never above it — a tag the channel strips or rejects must never reach a description. The customer knows their channel ("we sell on Allegro", "our webshop", "Amazon"); they rarely know its HTML rules by heart, so ask tag by tag in terms of what the *description looks like*, not in terms of tags.

## Opening + escape hatch

Start with the escape hatch so limited channels take one question, not nine:

> "Does your sales channel display formatted descriptions — headings, bullet lists, images — or does it only show plain text? If you're not sure, look at a competitor's product page on the same channel."

Plain text only → `channel_ceiling: []`, `image_placement: "none"`, and the walkthrough is done. Formatted → walk the tags.

## Tag-by-tag walk

Ask in this order; each answer adds the listed tags. Frame every question as "does your channel show/allow …" and offer "not sure" (treat as no — a stripped tag hurts more than a missing one).

1. **Paragraphs** — "Descriptions split into paragraphs?" → `p`. (Nearly always yes for formatted channels.)
2. **Layout blocks** — "Do descriptions use layout sections — boxes or side-by-side areas — beyond plain paragraphs?" → `div`.
3. **Line breaks** — "Line breaks inside a paragraph?" → `br`.
4. **Headings** — "Section headings, like a bold title above each part?" → `h2`, `h3`.
5. **Lists** — "Bullet or numbered lists?" → `ul`, `ol`, `li`.
6. **Emphasis** — "Bold or italic words inside text?" → `strong`, `em`.
7. **Tables** — "Specification tables, rows and columns?" → `table`, `tr`, `td`, `th`.
8. **Images** — "Images *inside* the description text itself (not the product gallery)?" → `img`.

The stored value is the flat tag list, e.g. `["p","br","h2","h3","ul","ol","li","strong","em","img"]`.

## Image placement — only when `img` made the ceiling

> "How should images appear in the description?"

- **hero** — "one strong image at the top"
- **interleaved** — "images woven between the text sections"
- **gallery** — "a group of images at the end"
- **ai_decided** — "let the AI pick how many and where, per product"

Mention the safety: products with few or no images simply show fewer or none — image sections disappear, nothing breaks. If `img` is not in the ceiling, store `"none"` and skip the question.

## Voice, audience, title style

- **Brand voice** — "If your brand were a person talking to a customer, how do they speak? (e.g. warm and familiar, technical and precise, playful…)" Store their words, lightly condensed; it feeds descriptions *and* titles.
- **Audience** — "Who is reading these — professionals buying tools, parents buying toys, …?"
- **Title style** — three questions:
  1. Style: "**keyword** — packed with search terms; **natural** — reads like a sentence; **minimal** — just the essentials?"
  2. Order: "What comes first in a title — brand, product type, or a key attribute? Give the order." (e.g. brand → type → attribute)
  3. Cap: "Does your channel cut titles at a length? How many characters?" (Common caps: Amazon ~200, Allegro 75, eMAG 255 — offer these when the customer is unsure, but store what they confirm; range 10–255.)

## Stored shape

The five answers persist under exactly these names — over MCP via `save-generation-profile`, or standalone into `~/.naratix/generation-profile.json`:

```json
{
    "channel_ceiling": ["p", "br", "h2", "h3", "ul", "ol", "li", "strong", "em", "img"],
    "image_placement": "interleaved",
    "brand_voice": "Warm, expert, never salesy; speaks to the reader as 'you'.",
    "audience": "Home cooks upgrading their first serious kitchen gear.",
    "title_style": {
        "style": "keyword",
        "length_cap": 75,
        "component_order": ["brand", "product_type", "key_attribute"]
    }
}
```

`image_placement` ∈ `none | hero | interleaved | gallery | ai_decided`; `title_style.style` ∈ `keyword | natural | minimal`; `length_cap` 10–255. The save replaces the whole profile — always send all five fields, including ones the customer didn't change.
