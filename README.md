# pi-sentinel

Run shell commands in background or monitor their stdout; the session is notified after the current run. Output is untrusted data, and jobs stop with the session.

## Install

```sh
pi install git:github.com/fyang93/pi-sentinel
```

## Tools

- `sentinel_run({ command, mode, description })`: `background` wakes once when the command exits; `monitor` coalesces stdout lines into events and also wakes on exit. Wake-ups are queued with `followUp` after the current run.
- `sentinel_list()`: list this session's jobs and log paths.
- `sentinel_stop({ id })`: terminate a job's process group without a wake-up.
