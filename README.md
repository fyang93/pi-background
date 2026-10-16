# pi-background

Run shell commands in background or monitor their stdout; the session is notified after the current run. Output is untrusted data, and jobs stop with the session.

While tasks are running, a separate line above the editor shows their count, including monitors.
It stays separate from quota and other widgets below the editor, regardless of their refresh order.
The line hides when none are running. `/ps` is always available and opens
this session's running and exited tasks with PID, mode and duration. Select a task to
view its recent output and log path, or stop it. Browsing tasks does not invoke the model.

## Install

```sh
pi install git:github.com/fyang93/pi-background
```

## Tools

- `background_run({ command, mode, description })`: `background` wakes once when the command exits; `monitor` coalesces stdout lines into events and also wakes on exit. Wake-ups are queued with `followUp` after the current run.
- `background_list()`: list this session's jobs and log paths.
- `background_stop({ id })`: terminate a job's process group without a wake-up.
