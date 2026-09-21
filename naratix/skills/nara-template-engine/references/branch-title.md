# Title template branch

Produces a title template: a single composed prompt, not a DSL pair.

1. The profile's `title_style` holds style, component order, and length cap; voice comes from `brand_voice`. Ask only what is missing, plus the title language.
2. Compose the title prompt against the **Title prompt pattern** in [companion-prompt.md](companion-prompt.md): self-contained, stating the style, the component order, the hard length cap, the language, and one line of brand voice.
3. Deliver:
   - **Connected:** `create-template` with `template_type: product-title`, which takes a `prompt` instead of the DSL fields. Pass `from_template_id` to revise one — same copy-on-write semantics as a description template.
   - **Standalone:** the prompt as a copy block plus *Title Templates → New → paste into the prompt field.*
4. Connected: offer to **try it on one product**, running the [test drive](test-drive.md) but ending in `generate-title` (`product_id`, the new `title_template_id`, the taxonomy). It returns a `run_id`; `list-runs` (`kind: generation`, that `run_id`) returns the finished title once `completed`, so measure it against the cap with the customer. The new title lands in the product's title history.

A title template that has never written a title is a guess. One real title measured against the length cap settles whether the cap is right.

**Completion:** the prompt is delivered, restates all three `title_style` answers (style, order, cap) plus the language, and the customer has either seen one real title or declined.

## SEO meta

`generate-seo` writes `meta_title` and/or `meta_description` for one product. It has no template — pass the language by English name and choose which fields to write (both default to true). The result goes to the product's SEO version history and reads back the same way as a title: `list-runs` with the `run_id` it returned.

Offer it when a customer talks about search listings or snippets rather than the product page itself.
