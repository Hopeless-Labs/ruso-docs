# Quick Start

This page takes you from zero to a real scan in a couple of minutes. It assumes
you've [installed](installation.md) the `ruso` binary.

## 1. Write a check

Save this as `redis.rsl` — it probes for an unauthenticated Redis instance:

```rsl
metadata {
    name "Exposed Redis (no auth)"
    description "Sends PING to Redis; an unauthenticated server replies PONG."
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

## 2. Validate the syntax

No network — just parse and compile:

```bash
ruso validate --script redis.rsl
```

A clean exit means the check compiles. Syntax errors point at the offending
line.

## 3. Scan a target

`scan` compiles and runs in one step. Spin up a throwaway Redis to test against:

```bash
docker run --rm -d -p 6379:6379 --name redis-demo redis:7-alpine
ruso scan --script redis.rsl --target tcp://127.0.0.1:6379 -v
docker rm -f redis-demo
```

You'll get a finding (detected) with the captured `redis_version` evidence. Point
the same check at a password-protected Redis and it won't detect — the server
refuses `PING` without `AUTH`.

## 4. Compile and ship (optional)

To distribute a check without its source, compile it to bytecode and run that:

```bash
ruso compile --script redis.rsl          # → redis.rbc
ruso exec --bytecode redis.rbc --target tcp://127.0.0.1:6379
```

The `.rbc` file is a compact, validated binary the runtime executes directly.

## 5. Use a shared check

Instead of writing your own, pull one from the registry and scan with it:

```bash
ruso scan --script someuser/log4shell --target https://target.example.com -v
```

A `<namespace>/<name>` reference is fetched from the registry into your local
cache on first use, then reused.

## Where to go next

- [Core Concepts](concepts.md) — the mental model behind checks, probes, and findings.
- [Your First Check](../rsl/first-check.md) — a guided walk through writing RSL.
- [Language Reference](../rsl/reference.md) — every keyword, field, and predicate.
