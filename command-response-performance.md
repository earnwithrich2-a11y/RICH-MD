---
name: Command response performance
description: Durable performance constraint for command dispatch and reactions.
---

Keep command reactions non-blocking and run independent command prerequisites concurrently. Do not add awaited work before reaction dispatch unless it is required for access control or correctness.

**Why:** The user explicitly requires commands and reactions to remain very fast and asked that future changes never reduce this speed.

**How to apply:** When editing message dispatch, permissions, settings lookup, metadata lookup, or plugin helpers, preserve immediate reactions and avoid serial independent lookups. Keep command results awaited when reliability requires it. The shared settings cache is intentionally short-lived and must be invalidated by `setSetting`.