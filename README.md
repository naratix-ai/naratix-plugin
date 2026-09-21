# Naratix Claude Code plugins

Set up [Naratix](https://naratix.ai) product-content generation for your shop from Claude Code — no technical knowledge needed. The `naratix` plugin bundles:

- **The Nara template wizard** (`nara-template-engine` skill) — an interview about your sales channel and brand that authors your description templates, title templates, and category mappings. Works connected to your shop, or standalone with paste-ready output.
- **The Naratix connection** — the shop's MCP server, authorized through your browser (no API keys to paste).
- **`/naratix:setup`** — re-runs the onboarding interview, e.g. after switching marketplaces.

## Install

```
/plugin marketplace add naratix-ai/naratix-plugin
/plugin install naratix@naratix
```

On first use you'll be asked to authorize the Naratix connection in your browser with your normal Naratix account.

## Use

Just say what you want — "set up product descriptions for my shop", "our titles need to fit Allegro's 75 characters", "apply the new template to everything under Kitchen" — or run `/naratix:setup` to start from the beginning.

The Naratix connection also offers three guided starts as slash commands in any AI app that shows MCP prompts: **Set up my shop**, **Set up titles** and **Check my catalogue**.

Everything the wizard writes to your shop is non-destructive: updates create new versions, deletes only archive, and nothing already generated is ever lost.
