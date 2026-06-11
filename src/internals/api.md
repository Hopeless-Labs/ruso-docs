# API Reference

Generated `rustdoc` for the two library crates at the core of Ruso. Reach for
these when embedding Ruso or building directly on the runtime or compiler.

- [**`ruso_runtime`**](/api/ruso_runtime/index.html) — the bytecode VM:
  `Executor`, `ExecutorConfig`, `decode_bytecode`, `Finding`, `SocketProbeSpec`,
  and the instruction set (`Instr`, `opcode`).
- [**`ruso_script`**](/api/ruso_script/index.html) — the RSL parser and compiler:
  `parse`, `compile`, the AST (`ast::Program`, `Stmt`), and bytecode helpers.

These docs are regenerated from crate source on every documentation build. For
the narrative explanation behind the types, read
[Architecture](architecture.md), [The Compiler](compiler.md),
[Bytecode Format](bytecode.md), and [The Runtime VM](runtime.md).
