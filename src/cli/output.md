# Output & Reporting

`scan` and `exec` always print results to the terminal. Adding
`--report <file>` *additionally* writes a machine-readable report — JSON or
CSV, chosen by the file extension.

## Terminal output (always on)

Findings **stream as they're found**, one line each:

```text
[CRITICAL] 127.0.0.1  Redis exposed without authentication
```

The `[SEVERITY]` tag is colour-coded (critical = magenta, high = red,
medium = yellow, low = cyan, info = grey) and the target is bold. With `-v`,
each run instead logs its outcome as it completes: `[OK]`, `[SKIP] … (reason)`,
or `[ERROR] … (msg)`.

A multi-run scan ends with a **per-target summary table** and a duration footer:

```text
┌─────────────┬──────────┬────────┬─────────┬───────┐
│ target      │ detected │ failed │ skipped │ clean │
├─────────────┼──────────┼────────┼─────────┼───────┤
│ target.test │        0 │     48 │       0 │     0 │
└─────────────┴──────────┴────────┴─────────┴───────┘
scan duration 1.4s · 48 runs across 1 target
```

Colour is applied only when stdout is a terminal; piped/redirected output (and
any run with `NO_COLOR` set) stays plain, so escape codes never pollute logs or
`grep`.

## Writing a report file

```bash
ruso scan --script ./scripts/ --target targets.txt --report findings.json
ruso scan --script ./scripts/ --target targets.txt --report findings.csv
```

The **extension decides the format** — `.json` or `.csv`, nothing else. Any
other extension fails fast:

```text
[ERROR] unsupported --report file `out.txt`: use a .json or .csv extension
```

There is no separate format flag; the terminal output above always prints
regardless, and `--report` just adds the file.

## JSON

The report is **grouped by target** and findings-focused: each target carries
its own outcome counts and the findings detected against it. Clean, failed, and
skipped runs are reflected only in the counts — there are no per-run rows.

```json
{
  "summary": { "total_runs": 48, "detected": 2, "failed": 1, "skipped": 0, "clean": 45 },
  "targets": [
    {
      "target": "https://target.test",
      "summary": { "detected": 2, "failed": 1, "skipped": 0, "clean": 45 },
      "findings": [
        {
          "script": "redis_unauth.rsl",
          "name": "Redis exposed without authentication",
          "severity": "critical",
          "description": "…",
          "impact": "…",
          "author": "…",
          "cve": ["CVE-2022-0543"],
          "cwe": ["CWE-306"],
          "references": ["https://…"],
          "cvss": ["CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H"],
          "cvss_score": ["9.8"],
          "mitigation": "…",
          "version": "1.0.0",
          "family": "database",
          "tags": ["redis", "exposure"],
          "evidence": ["redis_version:7.2.4"]
        }
      ]
    }
  ]
}
```

Notes:

- **`script`** on each finding records which `.rsl` produced it (file path or
  registry ref).
- The top-level `summary` aggregates every run; each target's `summary` is the
  same buckets for that target only.
- Empty lists and absent optional fields are **omitted** from the JSON.

Every metadata field from the script's `metadata { … }` block rides on the
finding:

| Field | Source in `.rsl` |
|-------|-------------------|
| `cve` / `cwe` / `references` | the matching list literals |
| `cvss` | repeatable `cvss "…"` vector strings |
| `cvss_score` | repeatable `cvss_score 9.8` (stored as strings) |
| `mitigation` | single `mitigation "…"` line |
| `version` / `family` | `version "X.Y.Z"` / `family "web"` |
| `tags` | `tags ["…", "…"]` |

## CSV

One row per finding, ordered by target. List fields are joined with ` | `.
Columns:

```text
target, script, severity, finding_name, description, impact, author,
cve, cwe, references, cvss, cvss_score, mitigation, version, family,
tags, evidence
```

A scan with no findings produces just the header row. CSV is findings-only — the
outcome counts live in the JSON `summary` (and the terminal table).

## Using it in CI

```bash
ruso scan --family web --target https://staging.internal --report findings.json
# Inspect counts, e.g. fail the job if anything was detected:
jq -e '.summary.detected == 0' findings.json
```

`scan`/`exec` exit non-zero when any run **failed** (a `fail`/`assert` or a
transport/decode error), so a build can gate on the exit code and use the
report file for detail.
