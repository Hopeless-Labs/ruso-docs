# Installation

Ruso is distributed as a single binary, `ruso`. While the crates are not yet on
crates.io, you install from source with Cargo.

## Prerequisites

- **Rust** (stable) and Cargo — install via [rustup](https://rustup.rs).
- A C toolchain (for building native TLS/crypto dependencies). On Debian/Ubuntu:
  `sudo apt install build-essential pkg-config libssl-dev`.

## Install with Cargo (recommended)

Install the `ruso` binary straight from the repository:

```bash
cargo install --git https://github.com/Hopeless-Labs/ruso-cli.git
```

Cargo resolves the `ruso-runtime` and `ruso-script` dependencies automatically.
The binary lands in `~/.cargo/bin/ruso` (make sure that's on your `PATH`).

## Build from a checkout

If you want to hack on the CLI, clone it and build:

```bash
git clone https://github.com/Hopeless-Labs/ruso-cli.git
cd ruso-cli
cargo build --release
# binary at ./target/release/ruso
```

To work against local checkouts of the runtime and language crates, clone them
as siblings — the `paths` override in `.cargo/config.toml` picks them up
automatically:

```text
parent/
├── ruso-cli/
├── ruso-runtime/
└── ruso-script/
```

When the siblings are absent, Cargo falls back to the git dependencies.

## Verify

```bash
ruso --help
ruso --version
```

You should see the command list (`scan`, `validate`, `compile`, `exec`, plus the
registry commands). Next: [Quick Start](quick-start.md).

## Configuration locations

Ruso keeps two things on disk:

| Path | Purpose |
|------|---------|
| `~/.ruso/scripts/<ns>/<name>/<version>.rbc` | Local install cache for registry checks. Override the root with `$RUSO_HOME`. |
| `$XDG_CONFIG_HOME/ruso/credentials.json` | Registry credentials, per registry URL (mode `0600` on Unix). |

Neither is created until you install a check or log in to a registry.
