# Testing Your Checks

A check is only worth publishing if you've proven it does two things:

1. **Detects** the vulnerable / exposed condition it targets.
2. **Stays quiet** against a safe, patched, or hardened target.

A check that only ever fires (or never fires) is worse than no check — it
erodes trust in every result. Prove both directions before you ship.

## Use disposable Docker targets

The cleanest way to get a known-vulnerable *and* a known-safe target is a
throwaway container. Don't test against a host `python -m http.server` or random
internet hosts — you want a target whose state you fully control.

### Example: the unauthenticated-Redis check

**Vulnerable target** — Redis with no password:

```bash
docker run --rm -d -p 6379:6379 --name redis-vuln redis:7-alpine
ruso scan --script redis.rsl --target tcp://127.0.0.1:6379 -v
# expect: detected
docker rm -f redis-vuln
```

**Safe target** — same image, password required:

```bash
docker run --rm -d -p 6379:6379 --name redis-safe \
  redis:7-alpine redis-server --requirepass secret
ruso scan --script redis.rsl --target tcp://127.0.0.1:6379 -v
# expect: not detected  (PING is refused without AUTH)
docker rm -f redis-safe
```

If the check detects in the first case and stays quiet in the second, it's
doing real work — not just pattern-matching the presence of the service.

### Tips

- Prefer **small, official images** (`*-alpine`, official upstream tags) so pulls
  are fast.
- **Clean up** containers (and pulled images, if you won't reuse them) after
  testing.
- For HTTP checks, many products ship a vulnerable demo image; otherwise toggle
  the relevant setting (auth on/off, header present/absent) to create the
  "safe" variant from the same image.

## The fast inner loop

While iterating, lean on the cheap commands first:

```bash
ruso validate --script check.rsl        # syntax + compile, no network
ruso scan --script check.rsl --target <vuln>  -v   # should detect
ruso scan --script check.rsl --target <safe>  -v   # should NOT detect
```

`validate` catches every parse/compile error without touching the network, so
run it on every edit. Only move to `scan` once it compiles.

## Reading the result

- **`detected`** — the [match chain](../guide/concepts.md#the-match-chain) held
  to the end and a finding was emitted (with your evidence).
- **`not detected`** — at least one `match` failed, or the run hit `stop`.
- **`skipped`** — a required port was already seen closed earlier in this `ruso`
  process (a 30-second per-run port cache).
- **error** — an `assert` failed, a `fail` ran, or a probe errored (e.g. an
  `evidence` regex that didn't match).

Run with `-v` (or `-vv`) to see the per-probe detail behind the verdict.

## Publishing checked work

Checks shared through the registry are expected to be proven this way. Once your
check passes both the vulnerable and safe cases, see
[Publishing & Installing](../registry/publishing.md).
