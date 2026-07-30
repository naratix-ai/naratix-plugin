# Quality checks — which engine answers which question

A template can only write from what the products actually carry. When a test drive comes back thin, or a customer asks why some descriptions read worse than others, the answer is usually the data rather than the template. Three engines will say which, by labelling the products they find fault with.

**This branch needs a connection.** Standalone, say so rather than attempting it.

Offer it when a customer asks how good their data is, when generated content keeps coming out empty in the same places, and **before a big generation run** — a free check first can stop a paid run producing thin content across a whole catalogue.

**Running a check is free.** All three are deterministic and make no AI calls, so `run-quality-check` never bills anything. Say that plainly — customers expect anything that scans a catalogue to cost money, and they will under-use it if left to assume.

## The three engines

Pick one with `engine`:

- **`health`** — scores each product's own data completeness: title quality, description, images, identifiers. Needs no setup, so it is the one to reach for first.
- **`consistency`** — evaluates the shop's check rules and flags products that break them. Needs a `taxonomy_id`.
- **`applicability`** — finds attributes that should be filled and are not, and attributes filled that do not belong on that kind of product. Needs a `taxonomy_id`.

## Choosing the products

Set `scope`: `shop` (the whole catalogue), `taxonomy`, `category` (that category and everything beneath it), or `products` with an explicit list.

Narrow further with `review_status` and `labels`. Passing labels is how to re-check only what is currently flagged, which is what a second pass wants. A label name the shop does not have comes back with the list of ones it does, so a wrong guess self-corrects.

`consistency` and `applicability` take a selection of any size, a whole catalogue included. `health` caps one run, because it scores every product in a single pass — over the limit it refuses and reports the count and the limit. The fix is a narrower scope, not a retry.

## Nothing pushes the result back

The call returns immediately with a run that is still going. Say so rather than going quiet, then poll `list-runs` (`kind: quality-check`) to find out whether it finished. It lists runs from all three engines, each tagged with its `engine`, so match on that rather than assuming the newest row is yours. Expect a catalogue-sized run to take a while — check back at a sensible interval instead of hammering it.

A consistency run has no id at dispatch time (the engine mints it inside the job), so `list-runs` is the only way to see one at all. It is also how to pick the thread up when a customer leaves and returns later. The runs are still there.

## Turning findings into the next action

Read the findings with the customer and name what each one implies. Attributes that are simply missing are a job for Naratix's data mining — run from the app, not from this wizard — rather than something a better template would fix. A rule the customer disagrees with is a rule to change, not a product to fix.

Completion: the customer knows which engine ran over what, that it cost nothing, and either has the findings or knows the run is still going and how it will be checked.

## Rules have to exist first

`consistency` and `applicability` evaluate rules. A taxonomy that has never been cold-started has no rules, so a check comes back clean for products nobody has examined — which reads exactly like good news and is not.

Confirm with `list-quality-rules` (read-only, free) before running either engine, and before promoting generated drafts.

## Cold-start is the one thing here that spends money

`cold-start-rules` writes those rules — real metered LLM work, sized by how much taxonomy there is.

- Call it **without** `confirm` first. It spends nothing and returns how much work it would be. That number is what goes to the customer, before anything runs.
- Only after an explicit yes, call it again with `confirm: true`.
- Never cold-start "to see what happens". It is not a free look.
- `health` has no cold-start — its scoring is built in, not derived. Another reason to start there.
