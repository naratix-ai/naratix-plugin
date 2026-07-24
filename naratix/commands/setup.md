---
description: Run (or re-run) the Naratix onboarding interview and overwrite the shop's Generation Profile
---

Run the `nara-template-engine` skill's onboarding interview, skipping the profile check — this command means the user wants to (re)do their setup now, typically after switching sales channel or marketplace.

1. Follow the skill's mode detection (connected shop via the naratix MCP server, or standalone with the local profile file).
2. Go straight to the onboarding interview — even when a Generation Profile already exists. An existing profile's answers may be offered as defaults ("keep this?"), never as a reason to skip a question.
3. On completion, overwrite the profile (connected: `save-generation-profile`; standalone: rewrite `~/.naratix/generation-profile.json`).
4. Since a changed ceiling usually invalidates existing templates, close by offering the description-template branch against the new profile.
