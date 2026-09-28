---
name: Plugin test isolation
description: Why tests that import real bot commands need a bounded child process
---

Tests that load real command modules may keep Node's test process alive even after all assertions pass. Isolate such handler tests in a bounded child process, and explicitly exit the child after checks finish.

**Why:** Both loading all plugins and importing a single command module left the test runner waiting on live handles. Isolating the command invocation let the parent test finish without weakening the behavioral assertions.

**How to apply:** For a regression test that must run real command code, spawn a child with a short timeout; assert its exit status and observed behavior. Keep the parent process free of plugin imports.