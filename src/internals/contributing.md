# Contributing

Most contributions touch one of the three open crates, all of which live in the
[`ruso`](https://github.com/Hopeless-Labs/ruso) workspace:

| Crate | What lives there |
|-------|------------------|
| `crates/ruso-script` | RSL grammar, parser, and the compiler to bytecode |
| `crates/ruso-runtime` | the bytecode VM, probes, matchers, findings |
| `crates/ruso-cli` | the `ruso` binary, scan orchestration, reporting |

This book is its own repository,
[`ruso-docs`](https://github.com/Hopeless-Labs/ruso-docs).

See [Architecture](architecture.md) for how the crates fit together, and the
[dependency graph](extending.md#dependency-graph).

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

Clone the workspace once; it holds all three crates:

```bash
git clone https://github.com/Hopeless-Labs/ruso.git
cd ruso
```

```text
ruso/
└── crates/
    ├── ruso-cli/
    ├── ruso-runtime/
    └── ruso-script/
```

They are path dependencies on each other, so a change in one is seen by the
others on the next build — nothing to publish, patch, or override.

```bash
cargo build --workspace
cargo run -p ruso-cli -- scan \
  --script crates/ruso-script/examples/http_status_ok.rsl \
  --target https://example.com
```

## The quality gate

The workspace must stay green on:

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace
```

Run them workspace-wide, not just in the crate you touched — a change in the
runtime can break the CLI's tests. New behaviour should come with a test; the
runtime in particular favours small, network-free regression tests (build a
`BytecodeProgram` and assert on the result).

Every public item in `ruso-runtime` and `ruso-script` carries a `///` doc
comment, and CI builds the docs with `-D missing_docs`, so a new `pub` type,
field, variant, or function without one fails the build. The rustdoc you write
is what readers see at [/api](/api/index.html).

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
