# Quick Start

Ruso scans a target against a library of ready-made checks. No checks to write,
no config to fill in — install it, point it at something you're allowed to test,
and go. This page gets you from install to your first finding in about a minute.

> **Scan only what you're authorized to test.** Run Ruso against your own
> systems or targets you have explicit permission to assess. Unauthorised
> scanning may be illegal.

## 1. Install

```bash
cargo install --git https://github.com/Hopeless-Labs/ruso-cli.git
```

That's the whole setup. (Prerequisites and other methods: [Installation](installation.md).)

## 2. Run your first scan

Point Ruso at a target and scan it against an entire **family** of community
checks — in one command:

```bash
ruso scan --family web --target https://target.example.com
```

Ruso pulls every published `web` check from the registry, runs them against your
target, and prints a finding for each hit — with severity and the evidence that
proves it. The default registry is the hosted one, so there's nothing to
configure.

Other families to try: `auth`, `database`, `dns`, `network`, `tls`, `cloud`,
`mail`.

## 3. Search for a specific check

Looking for a known issue? Search the registry:

```bash
ruso search log4j --tag rce
ruso search --family database --severity high
```

Each result shows a `<namespace>/<name>` reference you can run directly.

## 4. Run one check by name

```bash
ruso scan --script someuser/log4shell --target https://target.example.com -v
```

A registry reference is fetched and cached on first use, then reused. `-v` shows
the per-probe detail and the evidence behind each finding.

## 5. Read the result

| Verdict | Meaning |
|---------|---------|
| **detected** | The check matched — a finding was emitted (with evidence). |
| **not detected** | The target didn't meet the check's conditions. |
| **skipped** | A required port was already seen closed in this run. |
| **error** | A precondition (`assert`) or probe failed. |

Add `-v` / `-vv` for the detail behind any verdict.

## Where to go next

- **Want to write your own check?** [Write Your Own Script](../rsl/first-check.md)
  builds one from scratch in a few lines.
- **Curious how it works?** [Core Concepts](concepts.md) gives you the mental
  model in five minutes.
- **Want to share checks?** See [Publishing & Installing](../registry/publishing.md).
