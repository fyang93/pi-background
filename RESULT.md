# Result: pi-sentinel

## Files

- `index.ts` — `sentinel_run`, `sentinel_list`, and `sentinel_stop`; background/monitor process management, session wake-ups, logging, limits, and cleanup.
- `package.json`, `.gitignore` — package manifest and local dependency exclusion.
- `test.ts` — nine process-level tests.
- `README.md` — install and tool overview.
- `LICENSE` — MIT, copyright fyang93.
- `SPEC.md` — included updated project specification.

## Verification

- `node --test test.ts` — **9 passed, 0 failed**.
- `node --check index.ts` — passed.
- `git diff --check` — passed.
- Checked for old `pi-ghostshell`/`ghostshell` and `ghost_*` identifiers outside Git history — none remain.

## Spec deviations

None known. Monitor exit notifications contain stdout only; stderr is logged, as specified. Per-monitor rate limiting, shutdown cleanup, and process-group termination are covered by tests.
