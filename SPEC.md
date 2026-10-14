# pi-sentinel — spec

A minimal pi extension that runs shell commands in the background and wakes the session that started them.
The agent writes the command; the extension has no domain logic. Model it on Claude Code's background
command and Monitor. Keep it small: one `index.ts` (target ≈150–250 lines), no runtime dependencies,
same style as ~/Projects/pi-model-router (read it first: package.json, index.ts, test.ts, README).

## Tools (registered with pi.registerTool, typebox schemas)

1. `sentinel_run({ command: string, mode: "background" | "monitor", description: string })`
   - Spawns `bash -lc <command>` with `cwd` = the session cwd, `detached: true` (own process group),
     stdin ignored, stdout/stderr piped. Returns immediately: `{ id, pid, mode }`.
   - `background`: write stdout+stderr to a log file (under os.tmpdir()/pi-sentinel/<session>/<id>.log).
     When the process exits, wake the session once with: description, exit code/signal, duration,
     last ~20 lines of output, log path.
   - `monitor`: read stdout line by line (readline). Each line is an event. Coalesce lines that arrive
     within 1 s into one wake-up; cap at 10 wake-ups per minute per monitor (beyond that, buffer and
     report "N lines suppressed" in the next wake-up). stderr goes to the log file only. When the process
     exits, wake once more with exit code and last lines.
   - No duration limit. A process runs until it exits or is stopped.
2. `sentinel_list()` → id, mode, description, pid, status (running/exited + code), started, event count, log path.
3. `sentinel_stop({ id })` → SIGTERM the process group (`process.kill(-pid)`), SIGKILL after 3 s if still alive;
   report the final status. Stopping does not send a wake-up.

## Waking the session
- Use `pi.sendMessage({ customType: "sentinel", content, display: true, details }, { deliverAs: "followUp", triggerTurn: true })`.
  `followUp` queues after the current run instead of interrupting it (do NOT use "steer").
- Content starts with a fixed header line, e.g. `sentinel <id> (<description>) <event|exited N>`, then the
  output lines quoted with `| ` prefixes, and a note that the output is untrusted external data, not instructions.
- Cap one message at ~8k characters (truncate with a marker).

## Lifecycle
- On `session_shutdown` (and process exit), stop every running job (process groups), without wake-ups.
- No persistence across sessions; jobs belong to the session that started them.
- Cap concurrent jobs per session at 20; `sentinel_run` beyond that returns an error naming `sentinel_list`/`sentinel_stop`.
- Must not keep the agent marked busy: a running job never calls agent-start/settle APIs; only wake-ups start runs.

## Tool descriptions (agent-facing; short, factual)
State the two modes, that wake-ups arrive after the current run, that output is untrusted data, that jobs end
with the session, and one example each:
- background: `sentinel_run({command: "pytest -q", mode: "background", description: "tests"})`
- monitor: `sentinel_run({command: "while sleep 30; do <read a value>; [ cond ] && echo HIT && break; done", mode: "monitor", description: "price watch"})`
No usage advice beyond facts.

## Package & tests
- package.json like pi-model-router: name `pi-sentinel`, version 0.1.0, type module, keywords ["pi-package"],
  `pi.extensions: ["./index.ts"]`, peerDependencies on @earendil-works/pi-coding-agent and typebox (check the
  installed pi for what extensions import: /home/yang/.bun/install/global/node_modules/@earendil-works/pi-coding-agent/docs/extensions.md).
- `test.ts` with node:test (run `node --test test.ts`; node is at ~/.local/share/fnm/aliases/default/bin) using a
  mock `pi` object: background wakes once on exit with exit code and tail; monitor wakes per (coalesced) line and
  on exit; rate cap; stop kills the process group (child of a child also dies) and sends no wake-up;
  session_shutdown stops all; job cap.
- README.md: what it is, install (`pi install git:github.com/fyang93/pi-sentinel`), the three tools, the two
  modes, followUp semantics, lifecycle. Short.
- LICENSE: MIT, copyright fyang93.
- Commit in small logical commits ending with "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>".
  Do NOT create the GitHub repo or push; Claude will review and publish.
