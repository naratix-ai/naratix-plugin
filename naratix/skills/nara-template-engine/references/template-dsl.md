# Template DSL — syntax reference

The Template DSL is pseudo-HTML: everything that is not a construct below passes through to the store verbatim, so the static markup you write must stay inside the shop's Channel Ceiling. The placeholders define the JSON schema the LLM fills at generation time — placeholder names become the response fields, and the Companion Prompt guides them by name.

## Typed placeholders — `{{TYPE::name}}`

Each distinct `name` becomes a required LLM response field. `name` must be word characters only (letters, digits, underscore).

| Placeholder | LLM produces |
|---|---|
| `{{string::intro}}` | a string, substituted in place |
| `{{array<string>::highlights}}` | a list of strings (loop it) |
| `{{array<string::question, string::answer>::faq}}` | a list of objects with those fields (loop it) |

**Only string values substitute inline.** The renderer replaces a placeholder in the body only when the LLM's value is a string — an inline `{{integer::…}}`, `{{number::…}}`, or `{{boolean::…}}` is never substituted and ships as raw placeholder text in the customer-facing description. The one integer with a purpose is the `@images` count directive (below), which is consumed by the image parser, not printed. So: printable content is `string`, repeatable content is `array<…>`; a numeric value the text should mention (weight, capacity) is just part of a string placeholder's guidance.

**One placeholder per line.** The parser is greedy: two `{{…::…}}` on the same line merge into one corrupt match and the first placeholder is never filled. Put each placeholder in its own tag on its own line.

**Engine-reserved names** — `product_title`, `product_images`, `image_count`, `img` belong to the engine: use them only in the exact constructs this reference shows, never as content placeholders you invent guidance for (except `image_count`, whose choice the Companion Prompt guides).

## Injected values (never LLM-generated)

- `{{string::product_title}}` — the product's real title, injected verbatim and escaped. Use it wherever the title should appear; the LLM cannot rewrite or translate it.
- `product_images` — the product's real image URLs, injected as an array for loops (below).

## Data loops

```
@foreach({{array<string::question, string::answer>::faq}} as $item)
    <h3>{!! $item['question'] !!}</h3>
    <p>{!! $item['answer'] !!}</p>
@end
```

- The closing tag is exactly `@end`.
- Loop-body variables use `{!! $var !!}` / `{!! $var['field'] !!}` and are HTML-escaped.
- Scalar-array loops: `@foreach({{array<string>::highlights}} as $h) <li>{!! $h !!}</li> @end`.
- Free per-iteration counters on object items: `{!! $item['__iteration'] !!}` (1-based), `{!! $item['__index'] !!}` (0-based), `{!! $item['__iteration_padded'] !!}` (`01`, `02`, …) — handy for anchors and ids. The LLM is never asked for them.
- Loops nest; counters stay independent.

## Image constructs — loops only

Images come exclusively from these two constructs. Both clamp to the images the product actually has, so a product with two images shows two and a product with none shows nothing — the whole block disappears, no broken markup.

**`@images` block** — repeats its body once per image, `{{img}}` is the URL:

```
@images({{integer::image_count}})
    <img src="{{img}}" alt="">
@endimages
```

- `@images(3)` caps at a literal 3; `@images` alone uses every image; `@images({{integer::image_count}})` lets the LLM choose the count (guide the choice in the Companion Prompt) — the count is clamped to availability either way. This directive is the only place an `integer` placeholder belongs.
- Closing tag is exactly `@endimages`.

**`product_images` loop** — the injected URL array as a normal data loop, for markup that needs counters:

```
@foreach({{array<string>::product_images}} as $image)
    <img src="{!! $image !!}" alt="">
@end
```

Its length follows the LLM's `image_count` too.

**Hard guardrail**: indexed refs `{{img::N}}` / `{{img::N-}}` exist in the engine but render a literal `NO_IMAGE` string into the description when the product lacks that image. Never emit them; a loop construct expresses every placement.

### Placement recipes (from the profile's `image_placement`)

- **hero** — one image up top: `@images(1) <img src="{{img}}" alt=""> @endimages` right after the opening section.
- **interleaved** — images woven between content: ONE `@images({{integer::image_count}})` block whose body wraps each image in its own break (`<div>`/`<p>` per iteration). Repeating several `@images(1)` blocks shows the *same first image* each time — one block, many iterations.
- **gallery** — all images together at the end: one `@images` block whose body is a compact `<img>` row/list.
- **ai_decided** — `@images({{integer::image_count}})` and a Companion Prompt line telling the LLM how to choose the count (e.g. "pick 0–4 images; skip images for accessories").
- **none** (or no `img` in the ceiling) — no image construct at all.

## Giving the template a look

Classes and CSS are ordinary markup to the engine — it substitutes placeholders and passes everything else through. The template carries its own stylesheet, and that is what the description is rendered with, including in a shared preview link.

Author a look only to the profile's `styling`:

- **`classes`** or **`both`** — put class names in the body and ship the CSS as the template's stylesheet:

```
<h2 class="nx-headline">{{string::headline}}</h2>
<ul class="nx-benefits">
@foreach({{array<string>::benefits}} as $benefit)
    <li>{!! $benefit !!}</li>
@end
</ul>
```

```css
.nx-headline { font-size: 1.4rem; margin: 0 0 .5rem; }
.nx-benefits { padding-left: 1.2rem; }
.nx-benefits li { margin-bottom: .4rem; }
```

- **`inline`** — the same declarations go in `style` attributes and there is no stylesheet, because the channel strips the classes that would point at it.
- **`none`** — no classes, no CSS. Bare markup is the correct output here, not a lesser one.

Two rules keep the pair honest: **every class in the CSS exists in the body, and every class in the body is styled.** An unmatched class is either dead CSS or an unstyled element — both look like a mistake to the customer.

Prefix class names (`nx-`) so they can't collide with the channel's own stylesheet, and style only your own classes — never bare element selectors like `h2 { … }`, which would reach outside the description on some channels.

The stylesheet must be self-contained: no `@import` — the shared preview blocks external stylesheets, so an imported font or reset silently vanishes there. (`@font-face` with an `https` source does render.)

**One light scheme — never `prefers-color-scheme`.** The description lives inside the merchant's page, on a background the stylesheet does not control. A media query that turns the component dark while the surrounding page stays white produces an unreadable block — observed on a real shop, and rejected by the customer on sight. Author for a light background with contrast that holds on white, and leave theming to the merchant's page.

## Empty sections keep their headings

The DSL has no conditional, so a section built from a loop still renders its surrounding markup when the LLM returns an empty array. A product with no attributes leaves a spec heading above an empty `<table>` — observed on a real generation, not theoretical.

Put a section's heading **inside** its loop whenever the section may legitimately be empty:

```
@foreach({{array<string::label, string::value>::specs}} as $spec)
    <tr><th>{!! $spec['label'] !!}</th><td>{!! $spec['value'] !!}</td></tr>
@end
```

…with the `<h2>` and `<table>` only where specs are guaranteed. When they aren't, prefer a single loop that carries its own heading row, or accept that thin products show an empty block and tell the customer so. The Companion Prompt's "never invent a value" rule is what makes the array empty rather than fabricated — that's the guardrail working, not a fault.

## What silently breaks a template

The engine does not validate DSL bodies — malformed constructs leak into customer-facing output as raw text. Before delivering, walk the body once against this list:

1. Two placeholders on one line (greedy-parser corruption — see above).
2. An inline non-string placeholder (`{{integer::…}}` outside an `@images` directive) — never substituted, ships as raw text.
3. `@foreach` without its `@end`, or `@images` without `@endimages` — the raw directive text ships in the description.
4. Blade spellings that look right but aren't: `@endforeach` (must be `@end`), `{{ $item }}` in a loop body (must be `{!! $item !!}`).
5. Placeholder names with hyphens, dots, or spaces — never matched, never filled.
6. A reserved name used as an authored placeholder.
7. Tags outside the ceiling — the engine won't strip them; the channel will.
8. An `array<…>` placeholder written inline, outside a loop — only strings substitute in place, so the raw placeholder text ships. Arrays exist to be looped.

## Worked example

Ceiling `["p","br","h2","ul","li","strong","img"]`, placement `interleaved`:

```
<h2>{{string::headline}}</h2>
<p>{{string::intro}}</p>
@images({{integer::image_count}})
<p><img src="{{img}}" alt=""></p>
@endimages
<h2>{{string::features_title}}</h2>
<ul>
@foreach({{array<string>::features}} as $feature)
    <li>{!! $feature !!}</li>
@end
</ul>
<p><strong>{{string::closing}}</strong></p>
```

Every placeholder name here (`headline`, `intro`, `image_count`, `features_title`, `features`, `closing`) must have a guidance line in the Companion Prompt.

## Length and paragraph count

**Length is set on the template** (`word_count`) and reaches the model as a target for the whole description.

**Paragraph count is not a setting on a DSL template.** The template's old `number_of_paragraphs` field only drives the legacy Blade layouts — on a DSL body it does nothing. Structure comes from what you write: one placeholder per section, or a loop when the count should vary. To get "three paragraphs", either write three placeholders or loop over an `array<string>` and say how many items you want in the Companion Prompt. Density per section is controlled the same way — through the prompt's per-placeholder guidance, not a toggle.

## Around the body

The template's header and footer fields are static HTML prepended/appended to every description — they are not DSL-parsed. Language and target length come from the template itself (its language and description-length settings), not from the DSL body.
