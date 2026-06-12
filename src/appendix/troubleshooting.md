# Troubleshooting

Common errors and the footguns behind them. For the authoritative field/keyword
detail, see the [Language Reference](../rsl/reference.md).

## "not a .rsl script file"

`--script` points at a file without the `.rsl` extension. Source scripts must be
`.rsl`; compiled bytecode must be `.rbc`. Rename the file or use the right flag
(`--bytecode` for `.rbc`).

## The script never detects (or always detects)

This is the single most important thing to rule out — test against **both** a
vulnerable and a safe target ([Testing Your Scripts](../rsl/testing.md)). A script
that fires unconditionally usually has a `match` that's too loose (e.g. matching
the mere presence of a service rather than the vulnerable behaviour).

## HTTP probe ignores the host I set

HTTP probes take their host from `--target` (the base URL); the `host` field in a
probe block is for **socket** probes (`tcp`/`udp`/`dns`). For sockets, interpolate
the scan target:

```rsl
tcp redis { host "{{scan_host}}" port 6379 }
```

## DNS matches never work

DNS has two modes with **different match fields**:

- `host` only → OS resolver → match on `.answer`.
- `host` + `port`/`payload` → DNS wire → match on `.response` / `.banner`.

Using `.answer` on a wire probe (or `.response` on a resolver probe) never
matches. See [DNS modes](../rsl/reference.md#dns-modes).

## `evidence home.body` errors on a TCP probe

`.body` is HTTP-only. For sockets use `.response`, or a probe-scoped regex:

```rsl
evidence home.response
evidence home regex 'PONG'
```

## Certificate verification failed

The target presents a certificate Ruso won't trust. If you *intentionally* trust
it (a self-signed lab box), pass `--insecure` on the CLI, or set
`verify_ssl false` on that specific HTTP probe. Never disable verification for
real targets.

## A target shows up as `skipped`

A required port was already seen closed earlier in the same `ruso` process —
results are cached for ~30 seconds per run to avoid hammering a dead port. It's a
performance optimisation, not an error.

## `repeat` fails to parse

`repeat N … end` was removed from the language. Use `for item in <list>` to
iterate, or `retry <probe> <n>` to re-send a probe.

## Publish rejected: missing version / bad family

- Publishing requires a `version` (SemVer) in metadata — local `validate`/
  `compile` don't.
- `family` must be one of the registry's curated set (`auth`, `cloud`,
  `database`, `dns`, `mail`, `misc`, `network`, `tls`, `web`). The language
  accepts any string; the registry enforces the set at publish time.

## Authentication failures against the registry

Confirm you're logged in to the *right* registry — credentials are stored per
registry URL. Check with `ruso whoami`, and remember `--registry` /
`$RUSO_REGISTRY_URL` override the default. Re-`login` if a token expired.

## Still stuck?

Run with `-v` / `-vv` for per-probe detail, and consult the
[CLI Reference](../cli/reference.md) for exact flag behaviour. Bugs go to the
relevant repository's issue tracker.
