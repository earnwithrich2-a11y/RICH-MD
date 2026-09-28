---
name: Rebranded self-updates
description: The compatibility boundary for self-updating a renamed bot while retaining local state.
---

Only apply self-update archives built for the current rebranded source layout. Do not silently apply the original upstream archive to this fork.

**Why:** An upstream update can restore the old source folders and entry-point references, leaving the renamed session and database data stranded. Existing sessions and local SQLite data must survive any code update.

**How to apply:** Require an explicitly configured RICHK-MD repository rather than falling back to the legacy upstream. When changing the update flow, verify the archive's layout before copying files and exclude the full local session directory plus database files, including SQLite WAL and SHM files. If the repository is unset or the archive is incompatible, stop with a clear message instead of modifying the installation.