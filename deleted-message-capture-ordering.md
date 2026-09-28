---
name: Deleted-message capture ordering
description: The timing constraint between storing an incoming message and handling its revoke.
---

Store incoming messages before a later revoke event can be processed. If capture becomes asynchronous, give recovery a way to wait for the pending save for that message.

**Why:** A deferred, fire-and-forget save can run after a quick deletion is received, leaving recovery unable to find the original message. Immediate storage is a deliberate tradeoff for reliable recovery.

**How to apply:** Do not defer anti-delete capture without a per-message ordering barrier. When optimizing command latency, keep the save/revoke ordering guarantee rather than replacing the save with an untracked scheduled callback.