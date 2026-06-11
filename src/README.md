# The Ruso Book

**Ruso** is a vulnerability scanner driven by a library of small, shareable
checks. Point it at a target and scan against community checks straight from the
registry — no setup, no plugins to compile:

```bash
ruso scan --family web --target https://target.example.com
```

Need something bespoke? Write your own check in the **Ruso Scripting Language
(RSL)** — a few readable lines describing a probe and what a positive result
looks like. Either way, scanning for a known issue is usually a one-liner.

> **Development status:** Ruso is under active development. The language,
> bytecode format, CLI, and APIs may change without notice. Not recommended for
> production use yet.

## How it fits together

Ruso is intentionally **not** a monorepo. Each piece does one job and has a
stable contract with its neighbours:

| Component | Repo | What it does |
|-----------|------|--------------|
| **RSL** | [`ruso-script`](https://github.com/Hopeless-Labs/ruso-script) | Parses `.rsl` source and **compiles** it to bytecode |
| **Runtime** | [`ruso-runtime`](https://github.com/Hopeless-Labs/ruso-runtime) | A small **VM** that executes the bytecode, runs probes, and emits findings |
| **CLI** | [`ruso-cli`](https://github.com/Hopeless-Labs/ruso-cli) | The `ruso` binary — the driver you actually run |
| **Registry** | [`ruso-backend`](https://github.com/Hopeless-Labs/ruso-backend) | Publish, install, and search shared checks |
| **Checks** | [`ruso-labs`](https://github.com/Hopeless-Labs/ruso-labs) | A library of ready-made `.rsl` checks |

The flow is a short pipeline:

```text
check.rsl ──[ruso-script: parse + compile]──▶ bytecode (.rbc)
                                                  │
                                  [ruso-runtime: VM executes]
                                                  │
                                                  ▼
                                    probes ⇄ target   →   finding
```

A source check (`.rsl`) compiles to **bytecode** (`.rbc`) — a compact, validated
binary that the runtime executes. You can run a check in one step (`scan`
compiles and runs), or split the steps (`compile` then `exec`) and ship the
`.rbc` without the source.

## Who this book is for

- **Want to run or write checks?** Start with the [User Guide](guide/installation.md)
  and [Writing Checks](rsl/first-check.md).
- **Want to share checks?** See [The Registry](registry/publishing.md).
- **Want to hack on Ruso itself?** Jump to [Internals & Contributing](internals/architecture.md).

## A taste of RSL

```rsl
metadata {
    name "Exposed Redis (no auth)"
    severity high
    family "database"
    version "1.0.0"
}

tcp redis {
    host "{{scan_host}}"
    port 6379
    payload "PING\r\n"
}

send redis
match redis.response contains "PONG"

evidence redis regex 'redis_version:[0-9.]+'
```

Eight lines: declare what you're looking for, send a probe, decide what a hit
looks like, and capture proof. The next chapters take you from install to your
first real check.
