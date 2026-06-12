# Glossary

**Script** — one `.rsl` file describing a single security question and what a
positive result looks like.

**RSL (Ruso Scripting Language)** — the language scripts are written in. Source
files use the `.rsl` extension.

**Bytecode (`.rbc`)** — the compact, validated binary a script compiles to. The
runtime executes bytecode; the registry stores and ships it.

**Probe** — a network request defined in a script (`http`, `tcp`, `udp`, `dns`)
that is not performed until `send`.

**Send** — the statement that performs a probe's request and stores the response
under the probe's name.

**Match** — a statement testing a response field against a predicate. All matches
in a run AND together into the **match chain**.

**Match chain** — the running AND of every `match`. Once one fails it latches
false and later `match`/`evidence` short-circuit.

**Assert** — like `match`, but a failure **aborts the run** with an error instead
of latching the chain false. Used for hard preconditions.

**Finding** — the result emitted when a script finishes with the match chain true
and a `name`/`report` set: metadata plus captured evidence.

**Detected** — the CLI verdict that a finding was emitted.

**Evidence** — proof strings attached to a finding (a body excerpt or a regex
capture), recorded only while the match chain is true.

**Metadata** — the `metadata { }` block describing the finding (name, severity,
CWE/CVE, CVSS, family, version, …).

**Family** — a single curated structural category (`web`, `database`, `tls`, …);
the unit for `scan --family`.

**Tags** — many free-form discovery labels per script.

**Target (`--target`)** — what a script runs against. HTTP probes use it as the
base URL; socket probes read it via the `scan_host`/`scan_port`/`scan_url`
variables.

**Session probe** — a TCP/UDP/DNS probe with `session true`, which keeps the
connection open across multiple `send`s and appends responses.

**Registry** — the Ruso registry service where scripts are published, searched,
and installed.

**Namespace** — the owner segment of a script reference (`<namespace>/<name>`);
your username.

**Reference (registry ref)** — `<namespace>/<name>[@<semver-range>]`, how a
published script is addressed.

**PAT (Personal Access Token)** — a longer-lived, scoped credential for
authenticating to the registry without the web flow.

**Yank** — hide a published version from new installs without deleting it;
existing installs keep working.

**Install cache** — `~/.ruso/scripts/<ns>/<name>/<version>.rbc`, where fetched
scripts are stored and reused (override the root with `$RUSO_HOME`).
