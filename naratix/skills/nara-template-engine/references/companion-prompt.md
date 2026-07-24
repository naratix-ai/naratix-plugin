# Companion Prompt patterns

The Companion Prompt is the Template DSL body's inseparable other half. At generation time the engine builds the product context (title, category, mined specifications) and the base system prompt itself, then appends the Companion Prompt — so it carries exactly three things: **brand voice**, **audience**, and **per-placeholder guidance keyed to the DSL's placeholder names**. Language and target word count are injected from the template's own settings; restating them here creates conflicts.

## The shape

```markdown
## Brand voice
Warm, expert, never salesy. Speaks to the reader as "you". Short sentences,
concrete claims — no superlatives without a spec to back them.

## Audience
Home cooks upgrading their first serious kitchen gear. They know what they
want to cook, not what steel hardness means — translate specs into outcomes.

## Section guidance
- headline: a benefit-led hook, not the product name (the title already shows it).
- intro: two or three sentences — what this product is and the single
  strongest reason this audience buys it.
- image_count: choose 0–4. Prefer 2+ for visually distinctive products;
  choose 0 for plain accessories.
- features_title: a short heading for the feature list, in the description's language.
- features: 4–6 bullets. Each bullet = one concrete capability translated
  into a cooking outcome. No spec-sheet dumps.
- closing: one sentence of confident encouragement — no discounts, no urgency.
```

## Rules that make the pair work

1. **Cover every placeholder.** Each placeholder name in the DSL gets a guidance line under `## Section guidance`, addressed by its exact name. An unguided placeholder gets generic filler; a guided one gets the brand.
2. **Arrays get count + per-item shape.** For `array<…>` placeholders say how many items and what one item looks like ("4–6 bullets, each one capability → outcome").
3. **`image_count` is guidance, not markup.** When the DSL uses `@images({{integer::image_count}})`, tell the LLM how to choose the number; the engine clamps it to the images that exist.
4. **Voice and audience come from the profile.** Rephrase the stored `brand_voice` and `audience` into working instructions; don't invent a new voice at authoring time.
5. **Injected values need no guidance.** `product_title` and `product_images` are filled by the engine, never by the LLM — listing them under Section guidance is noise.
6. **Edit in lockstep.** Renaming, adding, or removing a DSL placeholder means regenerating the matching guidance lines in the same delivery. A prompt that guides placeholders the DSL no longer has (or misses ones it gained) is the most common way quality quietly degrades.

## Title prompt pattern

The title branch produces a single self-contained prompt (stored on the title template). Unlike the Companion Prompt it stands alone, so it states everything:

```markdown
Compose a product title.

Style: keyword — dense with the terms a buyer would search, no filler words.
Order: brand, then product type, then the one attribute that differentiates
this product (size, material, or capacity — whichever the specs make notable).
Hard limit: 75 characters. Never exceed it; drop the attribute first if needed.
Language: Polish.
Voice: confident and plain — no exclamation marks, no ALL CAPS.
```

Map the profile's `title_style` onto it directly: `style` → the style line (keyword: search-term-dense / natural: fluent sentence-case / minimal: bare essentials), `component_order` → the order line, `length_cap` → the hard limit, `brand_voice` → one condensed voice line. Ask only for the language.
