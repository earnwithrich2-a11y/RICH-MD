---
name: Post-merge dependency validation
description: Constraints that keep automatic task-merge setup reliable.
---

Keep all directly imported runtime modules declared, avoid npm shims for Node built-ins, and ensure the clean dependency tree uses firewall-safe transitive versions. The startup smoke check must use an explicit nonzero test port.

**Why:** A merged dependency lock failed repeatedly: an unnecessary built-in shim was blocked, a vulnerable transitive archive package required a safe override, and undeclared imports only appeared after a clean install.

**How to apply:** After dependency or task-agent merges, run the configured post-merge setup rather than relying on the existing node_modules directory. Preserve the startup validation script and treat any clean-install missing-module error as a manifest defect.