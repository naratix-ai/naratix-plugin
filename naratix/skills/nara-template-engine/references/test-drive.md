# Test drive — try a template on a real product

Offer this after authoring a template, and after a re-mapping puts a different template in front of a category. It needs a connection; standalone, say a test drive needs one rather than attempting it.

1. `list-test-products` (pass the taxonomy the template serves) returns candidates best-first, each with a verdict and a reason. **Show the top few and let the customer pick one — picking for them is not allowed, even when the ranking makes the answer look obvious.** Repeat each reason in their words: a `bare` product makes any template look worse than it is, and they deserve to know that before judging the output.
2. One test writes one description (or one title) and bills one generation. Per the skill's Conduct rules, say so first and wait for an explicit yes to *that product*.
3. `generate-description` (`subject_type: product`, `subject_id`, template, taxonomy) returns a link immediately. Hand it over right away and say it fills itself in — the customer can watch it arrive rather than wait.
4. `get-description-status` when they want to know it is done; report ready, or say plainly that it failed and nothing on the product changed.
5. Read the result with them against what they asked for. If a section came out empty, name the cause — usually a product with no details rather than a fault in the template — and offer to try a `ready` product before changing anything.

For a title template, run the same shape but end in `generate-title`; see [branch-title.md](branch-title.md).

**Completion:** the customer has the link and knows whether the description landed, or declined the offer.

## When several products come back thin

One thin result is a thin product (step 5). Several thin the same way is the catalogue, not the template — stop adjusting the template and offer a free quality check instead: [quality-checks.md](quality-checks.md).
