# Core Concepts

A handful of ideas explain almost everything in Ruso. Once these click, the
[Language Reference](../rsl/reference.md) reads like a lookup table.

## Scripts

A **script** is one `.rsl` file. It describes a single security question — "is
this Redis exposed?", "does this server leak its version?" — and what a *yes*
looks like. Scripts are deliberately small: declare metadata, define probes, send
them, and decide whether the responses constitute a finding.

## Probes

A **probe** is a network request you define but don't send yet. Ruso has four
kinds:

| Probe | Transport | Typical use |
|-------|-----------|-------------|
| `http` | HTTP/HTTPS | Web apps, APIs, headers, status codes |
| `tcp` | Raw TCP (optionally TLS) | Banners, line protocols (Redis, SMTP…) |
| `udp` | Raw UDP | NTP, DNS wire, echo |
| `dns` | OS resolver **or** DNS wire | Name resolution, record lookups |

Defining a probe (`http home { … }`) only describes it. Nothing hits the network
until you `send` it.

## Send, then match

Execution is a short script of statements:

```rsl
send home                       # now the request goes out
match home.status == 200        # inspect the response
match home.body contains "admin"
```

`send` performs the request and stores the response under the probe's name.
`match` inspects a **field** of that response (`status`, `body`, `header(...)`,
`response`, `answer`, …) with a **predicate** (`==`, `contains`, `regex`, …).

## The match chain

All the `match` statements in a run form a single **chain** of AND-ed
conditions. The moment one fails, the chain latches **false** and later
`match`/`evidence` statements short-circuit — so a script "detects" only when
*every* match held.

`assert` is the strict cousin: instead of latching the chain false, a failed
`assert` **aborts the run** with an error. Use it for hard preconditions ("must
be 200 before we go on"); use `match` for the actual finding logic. See
[`match` vs `assert`](../rsl/reference.md#match-vs-assert).

## Findings and `detected`

When a script finishes with the chain still true — and it has a `name` or
`report` — the runtime emits a **finding**: the metadata plus any captured
**evidence**. The CLI reports this as `detected`. `stop` ends a run with *no*
finding; `exit` ends it and emits the finding if the chain held.

## Source vs bytecode

You write **source** (`.rsl`); the runtime executes **bytecode** (`.rbc`).
Compiling is explicit (`compile`) or implicit (`scan` does it for you). Bytecode
is a compact binary with a validated structure — it's what the registry stores
and ships, and what `exec` runs directly. See
[Bytecode Format](../internals/bytecode.md) for the wire details.

## Targets and `--target`

The CLI's `--target` sets the *base* a script runs against. For **HTTP** probes it
becomes the base URL (`path` is appended). For **socket** probes it populates the
`scan_host` / `scan_port` / `scan_url` variables you interpolate into the probe:

```rsl
tcp redis { host "{{scan_host}}" port 6379 }
```

This is the single most common footgun — HTTP reads the base URL, sockets read
`host` from the script. See the [footguns list](../rsl/reference.md#common-footguns).

## Families and tags

Metadata carries two kinds of labels:

- **`tags`** — many per script, free-form discovery labels (`"rce"`, `"log4j"`).
- **`family`** — a *single* structural category (`web`, `database`, `tls`, …),
  the unit for "scan everything in this group" (`ruso scan --family web`).

The language accepts any string; the **registry** enforces a curated family set
at publish time.

## The registry

The Ruso registry is where scripts are **published**, **searched**,
and **installed**. A script is addressed as `<namespace>/<name>[@<version-range>]`.
Namespaces are your username; versions are SemVer. Installed scripts land in the
local cache and are reused across runs. See [The Registry](../registry/publishing.md).
