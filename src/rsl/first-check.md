# Write Your Own Script

This chapter builds a real HTTP script from scratch, one piece at a time. By the
end you'll understand every line — and be ready to use the
[Language Reference](reference.md) as a lookup table.

Our goal: **detect a web server that leaks its software version in the `Server`
header** (an information-disclosure finding).

## 1. Start with metadata

Every script opens with a `metadata { }` block. It describes the *finding* — not
the logic. Only `name` is strictly required for a script that emits
findings; the rest makes the result useful and the script publishable.

```rsl
metadata {
    name "HTTP server version disclosure"
    description "Flags servers that reveal their exact version in the Server header."
    impact "Version strings let attackers target known CVEs for that exact build."
    severity low
    author "you"
    cwe ["CWE-200"]
    tags ["disclosure", "http", "headers"]
    family "web"
    version "1.0.0"
}
```

## 2. Define a probe

A probe describes a request without sending it. For HTTP, the host comes from
`--target`; you supply the `path` and options:

```rsl
http home {
    method GET
    path "/"
    timeout 10s
    follow_redirect false
    user_agent "ruso/1.0"
}
```

Keywords are case-insensitive — `GET` and `get` are the same.

## 3. Send it

`send` performs the request and stores the response under the probe's name
(`home`):

```rsl
send home
```

## 4. Decide what a finding looks like

Now inspect the response. We want a `200 OK` *and* a `Server` header that
contains a version number. Two `match` statements, AND-ed together by the
[match chain](../guide/concepts.md#the-match-chain):

```rsl
match home.status == 200
match home.header("Server") regex '[0-9]+\.[0-9]+'
```

If either fails, the chain latches false and the script won't detect.

## 5. Capture proof

A finding is far more useful with **evidence** — the exact string that proves it.
`evidence` only attaches while the chain is still true:

```rsl
evidence home regex 'Server:[^\r\n]+'
```

## The complete script

```rsl
metadata {
    name "HTTP server version disclosure"
    description "Flags servers that reveal their exact version in the Server header."
    impact "Version strings let attackers target known CVEs for that exact build."
    severity low
    author "you"
    cwe ["CWE-200"]
    tags ["disclosure", "http", "headers"]
    family "web"
    version "1.0.0"
}

http home {
    method GET
    path "/"
    timeout 10s
    follow_redirect false
    user_agent "ruso/1.0"
}

send home

match home.status == 200
match home.header("Server") regex '[0-9]+\.[0-9]+'

evidence home regex 'Server:[^\r\n]+'
```

## Run it

```bash
ruso validate --script version-disclosure.rsl                 # compiles?
ruso scan --script version-disclosure.rsl --target https://example.com -v
```

A server that returns `Server: nginx/1.25.3` detects; one that sends a bare
`Server: nginx` (or strips the header) does not.

## What to learn next

- **Other transports** — swap `http` for `tcp`/`udp`/`dns`. See
  [Socket probes](reference.md#socket-probes-dns--tcp--udp).
- **Multiple steps** — send several probes, use `if`, `for`, `extract`/`save`.
- **Prove it works** — [Testing Your Scripts](testing.md) shows how to verify a
  script against both vulnerable *and* safe targets before you trust it.
