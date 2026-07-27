# Deriving a template from pasted descriptions — the diff method

One example shows what a description looks like; two show what it is *made of*. The whole method is one rule applied everywhere: **what repeats across examples is literal, what differs is content.**

## Collecting examples

Ask for **two or three descriptions of the same kind of product**, pasted as the markup the channel actually renders (copied from the page source or their PIM — not a screenshot). Same kind matters: examples of different kinds differ everywhere, and the diff dissolves into "everything is variable".

One example is workable, but say the trade-off plainly: every fixed-versus-variable call becomes a guess the customer must check, instead of a fact the diff proved.

### Links instead of pastes

Product-page URLs are the cheapest way to collect examples — "give me links to two or three similar products" costs the customer nothing. The ladder, in order:

1. **Connected: `fetch-shop-page`** (format `html`). It downloads through Naratix's scraping infrastructure, so bot walls that block a plain fetch are handled. Extract the description region — the block carrying the long-form product copy — with its classes, and discard the page chrome (navigation, header, footer, scripts). The look mostly lives in linked CSS: take the main `<link rel="stylesheet">` URL from the page and fetch it with the same tool — CSS files sit on CDNs and are rarely bot-walled. Storefront bundles can exceed the fetch cap and come back `truncated` — when they do, prefer the inline critical CSS already in the page's `<head>`, which carries the brand colours and typography on most modern storefronts. Inline critical CSS in `<head>`, `meta theme-color`, and inline `style` attributes are colour evidence too. Rules that target the description's classes can seed the stylesheet; the page's colours and typography pre-fill `brand_look`, confirmed with the customer rather than assumed. Budget the fetches — the tool allows ten per hour, and a derive needs two or three pages plus a stylesheet.
2. **Standalone: a plain fetch.** Works on permissive sites; when it comes back blocked or empty, ask for a paste.
3. **Last resort: a screenshot.** It shows the look — useful for the stylesheet — but carries no markup, so the DSL still needs at least one pasted example.

Say which rung was used. For extra same-category pages where only the content matters for the diff, fetch with format `markdown` — lighter, and the markup evidence is already in hand from the first page.

## The diff

1. Ignore whitespace-only differences.
2. Align the examples by tag skeleton. Where the skeleton itself matches, compare the text inside; where one example has more copies of a sibling block than another (3 bullets vs 5), that block is a **loop candidate**, not a text difference.
3. Text identical in every example → **literal markup**, kept verbatim: brand promises, section headings, boilerplate. This is how a standing promise like "Livrare gratuită peste 200 lei" survives.
4. Text that differs → a **typed placeholder**, named for its role (`headline`, `intro`, `garantie`), one per line, per the DSL reference.
5. A loop-candidate block becomes ONE `@foreach` over an `array<…>` placeholder — never `bullet_1`, `bullet_2`, `bullet_3`.
6. Numbers and units inside varying sentences stay part of the string placeholder — the DSL substitutes only strings inline.

**Single-example fallback:** decide provisionally — text naming the product or its specifications → variable; text that would sit unchanged on any product of this shop → fixed. Mark each low-confidence call as a guess in the split table below.

## The split table — nothing is created before it

Show the customer two lists and wait:

- **Fixed** — kept verbatim, will appear on *every* description. Quote each piece.
- **Variable** — placeholder name, one line on what fills it, and the example values the diff saw.

Guess wrong toward variable and the brand's standing promise disappears; wrong toward fixed and every product repeats one product's sentence. The table is the only defence, and it is not optional — with one example it is where wrong guesses get caught, with three it is where the customer sees their template for the first time.

## Markup, styling, and what the examples prove

- Keep the customer's own class names; lift their CSS into the template's stylesheet. If they pasted markup with classes but no CSS, ask for the stylesheet — or author one from the page's rendered look, under the look-section rules of [template-dsl.md](template-dsl.md).
- **Tags present in the examples are proven accepted by the channel** — the paste is direct evidence. For a shop not yet onboarded, pre-fill `channel_ceiling` with exactly those tags and confirm in one question; skip the tag-by-tag walk. Classes or inline styles in the paste answer the styling question the same way.
- For an onboarded shop whose stored ceiling lacks a tag the examples use (or vice versa), surface the contradiction and let the customer settle it — never silently widen the profile or silently drop their markup.
- Whatever the settled ceiling excludes is dropped from the derived template, with the customer told what was dropped and why.

## Companion Prompt and word count, from evidence

- Describe the voice the examples *demonstrate* — sentence length, person, how technical — and quote one characteristic sentence back as the register anchor. Don't invent adjectives the examples don't show.
- Guide each placeholder with the actual values the diff saw ("features: 4–6 bullets shaped like 'Rezistent la apă până la 50 m'").
- `word_count` = the examples' own average length, rounded — never a shop default.
- Romanian examples answer the diacritics question too: text carrying ă/â/î/ș/ț is evidence the channel keeps them, bare ASCII is evidence it doesn't. Confirm the reading with the customer and pass `use_diacritics` accordingly — the engine strips them when it's off.

## The round-trip check

Before delivering, run the authoring walk (§What silently breaks a template, the look-section rules, §Rules that make the pair work) plus one check specific to deriving: **substitute one example's content back into the derived DSL — it must reproduce that example**, modulo placeholder text. If it can't, structure was lost in the diff, and the template will never produce what the customer showed you.
