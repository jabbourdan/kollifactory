---
name: check-kollifactory
description: Check that Kollifactory is set up correctly in this project (config, Jira login, git or GitHub, checks, app preview) and say what to fix. Use when the user types /check-kollifactory, asks whether the factory is ready or why it won't run, or before a first /kollifactory run. Read-only.
---

# Check Kollifactory

Run the check and tell the user what it found. Change nothing.

```bash
factory/bin/check
```

Add `--run-checks` when the user asks for a full check or before a first run:
it also runs `factory/checks.sh` on the current checkout, which can take a
while.

The script prints `ok`, `warn` or `FAIL` per line, with a fix under each
`warn` and `FAIL`, and exits 1 when anything fails.

Then report, in this order:

1. One line: ready or not, and the mode (local or remote).
2. Each `FAIL`, with its fix in plain words. These stop the factory.
3. Each `warn`, briefly. These don't stop it.

Don't repeat the `ok` lines. Don't fix anything yourself: if something fails,
say that `/init-kollifactory` walks through the setup, and offer to run it.

If `factory/bin/check` doesn't exist, Kollifactory isn't installed in this
project: point the user at the reference folder's `INSTALL.md`.
