# Extending Ruso

Guide for maintainers adding features without breaking the generic probe model.

## Decision tree

```
New vulnerability check for existing transport?
  └─ Yes → add .rsl under examples/ or your checks repo (no Rust)

New socket behavior (TLS client cert, SCTP, …)?
  └─ Extend SocketProbeSpec + session layer + grammar

New control flow (while, goto)?
  └─ New Instr + compiler + executor + opcode doc + VERSION bump if needed

New protocol family unrelated to HTTP/socket bytes?
  └─ New ProbeKind variant + executor arm (rare; prefer socket first)
```

## Adding a check (script only)

1. Create `my_check.rsl` with metadata + probe + `send` + `match`.  
2. `ruso validate --script my_check.rsl`  
3. Test with `ruso scan` against a lab target.  
4. Ship the `.rsl` file; no runtime release required.

Use hex payload for binary protocols; `session` (and multiple `send`s) for multi-step dialogs.

Optional `cve`, `cwe`, `references`, `cvss`, and `cvss_score` lines (repeatable) plus a single `mitigation` line (free text; declaring it more than once is a compile error) attach advisory IDs, CVSS vectors/scores, remediation text, and URLs to findings and CLI/JSON reports.

## Adding a metadata field

Example: a new `tags` list on checks.

| Step | File |
|------|------|
| 1 | `grammar.pest` — keyword + `metadata_stmt` arm |
| 2 | `ast.rs` — `Stmt` variant |
| 3 | `parser/metadata.rs` + `spec_build.rs` |
| 4 | `spec.rs` — `CheckMetadata` field |
| 5 | `binary.rs` — `write_metadata` / `read_metadata` (append at end of metadata block) |
| 6 | `report.rs` / `disasm.rs` — `Finding` + listing |
| 7 | `docs/RSL_REFERENCE.md`, `BYTECODE.md`, `RUNTIME.md` |
| 8 | `ruso-cli` `report.rs` if exposed in scan output |
| 9 | Recompile all `.rbc` files (`VERSION` is not bumped during development) |

## Adding a socket probe option

Example: `connect_timeout 5s`.

| Step | File |
|------|------|
| 1 | `grammar.pest` — `kw_connect_timeout ~ duration` in `socket_item` |
| 2 | `ast.rs` — field on `SocketProbe` |
| 3 | `parser/socket.rs` — parse duration → ms or store string |
| 4 | `spec.rs` — `SocketProbeSpec` field + `Default` |
| 5 | `spec_build.rs` — copy field |
| 6 | `binary.rs` — encode/decode after existing fields |
| 7 | `session.rs` / `executor.rs` — use value on connect |
| 8 | `disasm.rs` — show in probe dump |
| 9 | `docs/RSL_REFERENCE.md` — document |
| 10 | Recompile all `.rbc` files if layout changes |

Backward compatibility: old bytecode cannot read new fields—bump version and recompile all stored `.rbc` files.

## Adding an instruction

Example: `close <probe>` to drop session.

| Step | File |
|------|------|
| 1 | `grammar.pest` + `ast::Stmt` |
| 2 | `compile.rs` — emit `Instr::Close(probe_id)` |
| 3 | `bytecode.rs` — enum variant |
| 4 | `binary.rs` — `OP_CLOSE`, write/read |
| 5 | `opcode.rs` — constant + table row |
| 6 | `executor.rs` — remove from `context.sessions` |
| 7 | `disasm.rs` — format line |
| 8 | Syntax test in `syntax_tests.rs` |

Prefer reusing `Send` + `session false` + new probe if semantics allow—fewer opcodes stay easier to reason about.

## Adding a matcher field

Example: `cert_cn` on TLS socket responses (hypothetical).

1. `contract::FieldKind` variant.  
2. `matcher.rs` — read from response (may need to store TLS info on `SocketResponse`).  
3. `grammar.pest` — `kw_cert_cn` in `qualified_field`.  
4. `match_expr.rs` — map to `FieldKind`.  
5. Binary matcher encode in `binary.rs` if already generic—field kinds are tagged on wire.

## HTTP extensions

Extend `HttpItem` in AST, `http_item` in grammar, `probes.rs`, `write_http_spec` / `read_http_spec`, and `execute_http`.

HTTP stays separate from `SocketProbeSpec` intentionally.

## Contract sharing

Types in `crates/ruso-runtime/src/contract.rs` are shared with the compiler via `ruso_script::ast` re-exports. Changing matchers affects **both** crates, and because they are path dependencies in one workspace, both are rebuilt and tested together in a single commit—there is no publish-then-bump dance.

## Testing checklist

- [ ] `cargo build --workspace`  
- [ ] `cargo test -p ruso-script` (parser)  
- [ ] `cargo test -p ruso-runtime` (unit tests)  
- [ ] `ruso validate` on affected examples  
- [ ] Round-trip `compile` → `decode` → `encode` for bytecode changes  
- [ ] One live `scan` against lab service  

## Anti-patterns

| Avoid | Prefer |
|-------|--------|
| `OP_REDIS`, `OP_SMTP` | `tcp` + payload |
| Forking executor per CVE | Parameterized `.rsl` |
| Raw SQL in scripts | Not applicable—keep I/O in runtime |
| Changing the wire layout silently | Changelog note telling users to recompile |

## Dependency graph

```text
ruso-cli → ruso-script → ruso-runtime
```

All three are members of one Cargo workspace and depend on each other by path, so working on all three at once needs no sibling clones, no `[patch]` table, and no path edits you have to remember to revert.

An outside crate depends on the libraries with a **git dependency** on `main`. Both packages come out of the same workspace, so both lines carry the same URL:

```toml
ruso-runtime = { git = "https://github.com/Hopeless-Labs/ruso.git", branch = "main" }
ruso-script = { git = "https://github.com/Hopeless-Labs/ruso.git", branch = "main" }
```

The crates share one version number and are released together. When publishing to crates.io later, a change to the runtime contract (`VERSION`, public types) still needs its own semver thought even though the version bumps in lockstep.

## Future directions (not implemented)

Documented for planning only:

- `--target` overrides socket `host` / `port`  
- `while` / `for` with variables  
- Byte-level matchers (`match hex`)  
- ICMP probe kind  
- Check signing / attestation of bytecode  

These should follow the same probe-table + small-ISA pattern.
