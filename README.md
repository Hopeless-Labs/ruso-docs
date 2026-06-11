# ruso-docs

Source for **[The Ruso Book](https://docs.ruso.hopeless-labs.com)** — the official
documentation for the [Ruso](https://github.com/Hopeless-Labs) vulnerability-scanning
ecosystem (the Ruso Scripting Language, CLI, runtime, and registry).

Built with [mdBook](https://rust-lang.github.io/mdBook/).

## Local preview

```bash
cargo install mdbook        # once
mdbook serve --open         # live-reloading preview at http://localhost:3000
```

`mdbook build` writes the static site to `book/` (git-ignored).

## Structure

- `src/SUMMARY.md` — the table of contents (chapter order lives here).
- `src/guide/`, `src/rsl/`, `src/cli/`, `src/registry/`, `src/internals/`,
  `src/appendix/` — chapter sources.
- `book.toml` — mdBook configuration.

Several chapters are adapted from the per-crate `docs/` in `ruso-script`,
`ruso-runtime`, and `ruso-cli`; substantive language/runtime changes should be
reflected here too.

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the book
and publishes it to GitHub Pages. The custom domain
`docs.ruso.hopeless-labs.com` is set via `src/CNAME` (copied into the build
output) plus a DNS `CNAME` record pointing at GitHub Pages.

## License

Apache License 2.0.
