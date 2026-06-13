# Contributing

Ruso is split across small single-purpose repositories. Most contributions
touch one of the three open crates:

| Repo | What lives there |
|------|------------------|
| [`ruso-script`](https://github.com/Hopeless-Labs/ruso-script) | RSL grammar, parser, and the compiler to bytecode |
| [`ruso-runtime`](https://github.com/Hopeless-Labs/ruso-runtime) | the bytecode VM, probes, matchers, findings |
| [`ruso-cli`](https://github.com/Hopeless-Labs/ruso-cli) | the `ruso` binary, scan orchestration, reporting |
| [`ruso-docs`](https://github.com/Hopeless-Labs/ruso-docs) | this book |

See [Architecture](architecture.md) for how they fit together, and the
[multi-repo dependency graph](extending.md#multi-repo-dependency-graph).

## Where a change belongs

Use the [Extending Ruso](extending.md) decision tree. In short:

- **A new vulnerability check?** No Rust needed — write a `.rsl` script. Start
  with [Write Your Own Script](../rsl/first-check.md).
- **New syntax / a metadata field?** `ruso-script` (grammar + compiler), and
  often the shared contract in `ruso-runtime`.
- **A new probe option, opcode, or matcher field?** `ruso-runtime`, then teach
  the compiler to emit it in `ruso-script`.
- **CLI flags, output, orchestration?** `ruso-cli`.

A change that crosses the bytecode boundary may need a `VERSION` bump — see
[Bytecode Format](bytecode.md#versioning-policy).

## Local setup

Clone the crates you're working on as **siblings**:

```text
parent/
├── ruso-cli/
├── ruso-runtime/
└── ruso-script/
```

`ruso-cli`'s `.cargo/config.toml` has a `paths` override that picks up sibling
`ruso-runtime` / `ruso-script` checkouts automatically, so a change in one is
seen by the others without publishing. When the siblings are absent, Cargo
falls back to the `git` dependencies pinned on `main`.

```bash
cargo build
cargo run -- scan --script ../ruso-script/examples/http_status_ok.rsl --target https://example.com
```

## The quality gate

Each crate must stay green on:

```bash
cargo fmt --all -- --check     # formatting
cargo clippy --all-targets     # lints (treat warnings as things to fix)
cargo test                     # unit + integration tests
```

Run these in every crate you touch before opening a PR. New behaviour should
come with a test — the runtime in particular favours small, network-free
regression tests (build a `BytecodeProgram` and assert on the result).

## Documentation

The docs are this mdBook (`ruso-docs`). Preview locally:

```bash
cargo install mdbook        # once
mdbook serve --open         # live reload at http://localhost:3000
```

`src/SUMMARY.md` is the table of contents. Substantive language or runtime
changes should be reflected here too — and the crate API docs are generated
from `///` comments, so keep those accurate (they ship at
[`/api`](/api/index.html)).

## Pull requests

1. Branch off `main`.
2. Keep the change focused; match the surrounding code's style and comment
   density.
3. Keep the quality gate green; add or update tests.
4. Update docs/changelog when behaviour changes.
5. Open the PR against the relevant repo.
