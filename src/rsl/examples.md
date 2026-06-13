# Example Scripts

Eight runnable scripts ship in [`examples/`](https://github.com/Hopeless-Labs/ruso-script/tree/main/examples)
in the **ruso-script** repository — two per protocol (HTTP, DNS, TCP, UDP),
each verified against a local Docker target. This chapter walks through one of
each, annotated; the rest follow the same shapes.

Run them from a clone of **ruso-script** (after installing the
[CLI](../guide/installation.md)):

```bash
ruso validate --script examples/http_status_ok.rsl          # syntax + compile, no network
ruso scan --script examples/http_status_ok.rsl --target http://127.0.0.1:8080
```

Socket scripts (`dns`/`tcp`/`udp`) take the host from `--target` via
`{{scan_host}}`; the port is the literal in the probe block. HTTP scripts use
`--target` as the base URL.

## HTTP — availability & content

`http_status_ok.rsl` GETs the target root and asserts the status, a body
marker, and a header — the bread-and-butter of an HTTP script:

```rsl
metadata {
    name "HTTP endpoint availability + content check"
    description "GET the target root, assert 200 and that the expected page marker is served"
    severity info
    author "ruso-lab"
}

# Path is relative to --target (the scan base URL).
http home {
    method GET
    path "/"
    timeout 5s
    follow_redirect false
    header "Accept" "text/html"
}

send home

match home.status == 200
match home.body contains "RUSO-HTTP-OK"
match home.header "Content-Type" contains "text/html"

evidence home.body
```

The three `match` lines AND together (the [match chain](../guide/concepts.md#the-match-chain)):
status **and** body marker **and** content type must all hold. `evidence home.body`
captures the body into the report.

> The companion `http_server_version_disclosure.rsl` uses a `HEAD` request and a
> regex on the `Server` header — `match banner.header "Server" regex 'nginx/[0-9]+\.[0-9]+'` —
> to flag a leaked version, staying quiet when `server_tokens off` yields a bare
> `nginx`.

## TCP — cleartext protocol probe

`tcp_redis_unauth.rsl` detects an unauthenticated Redis by sending a RESP `PING`
frame and matching `PONG`:

```rsl
metadata {
    name "Redis exposed without authentication"
    severity critical
    cve ["CVE-2015-4335"]
    cwe ["CWE-306"]
    family "database"
    version "1.0.0"
}

# Host comes from CLI --target via {{scan_host}}. The RESP frame
# `*1\r\n$4\r\nPING\r\n` is sent as raw bytes — control bytes (CRLF) must go
# through `payload bytes` (hex), not a "\r\n" escape in a text string.
# read_idle lets the read return once Redis has replied and gone quiet.
tcp redis_ping {
    host "{{scan_host}}"
    port 6379
    read_idle 300ms
    payload bytes "2a310d0a24340d0a50494e470d0a"
}

send redis_ping

match redis_ping.response contains "PONG"
match redis_ping.response not_contains "NOAUTH"
match redis_ping.response not_contains "ERR"

evidence redis_ping regex 'PONG'
```

Two things make this a *good* check rather than a noisy one: socket responses
are read via `.response` (not `.body`, which is HTTP-only), and the
`not_contains "NOAUTH"` / `not_contains "ERR"` guards keep it quiet against an
auth-protected Redis that answers `PING` with an error.

## DNS — wire mode

`dns_wire_a.rsl` sends a raw DNS A query and confirms the server answers. A
`dns` probe with a `port`/`payload` uses **wire mode** (vs the OS resolver when
only `host` is set):

```rsl
metadata {
    name "DNS wire-format A query"
    severity info
    references ["https://www.rfc-editor.org/rfc/rfc1035"]
}

# Header: ID aaaa, flags 0100 (RD), QDCOUNT 1.
# Question: 03"app" 04"ruso" 04"test" 00, QTYPE A (0001), QCLASS IN (0001).
dns wire_a {
    host "{{scan_host}}"
    port 53
    payload bytes "aaaa0100000100000000000003617070047275736f04746573740000010001"
}

send wire_a

# The response echoes the question section, so the queried labels appear verbatim.
match wire_a.response contains "ruso"

evidence wire_a regex 'ruso'
```

> `dns_wire_txt.rsl` is the same shape with a TXT query — handy because TXT
> rdata is ASCII and often carries tokens or verification strings.

## UDP — service probe

`udp_ntp.rsl` sends a 48-byte NTP client packet and confirms a reply, using an
anchored regex on the server-mode response byte:

```rsl
metadata {
    name "NTP service responds to client request (UDP)"
    severity medium
    cve ["CVE-2013-5211"]
    mitigation "Disable the monlist query, restrict NTP to trusted clients, and rate-limit UDP/123"
}

# First byte 0x1b = LI 0, VN 3, Mode 3 (client); remaining 47 bytes zero.
udp ntp {
    host "{{scan_host}}"
    port 123
    payload bytes "1b00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
}

send ntp

# A server packet's first byte is 0x1c (Mode 4); `^\x1c` confirms an NTP reply.
match ntp.response regex '^\x1c'
```

> `udp_echo.rsl` is the simplest socket script — a text `payload` and
> `match echo.response contains "RUSO-PING"`.

## All eight, by pattern

| Pattern | Example |
|---------|---------|
| Web availability / content | `http_status_ok.rsl` |
| Header / version disclosure | `http_server_version_disclosure.rsl` |
| DNS recon (wire) | `dns_wire_a.rsl`, `dns_wire_txt.rsl` |
| Cleartext protocol test | `tcp_redis_unauth.rsl` |
| Service fingerprint / banner | `tcp_http_banner.rsl` |
| UDP service probe | `udp_ntp.rsl`, `udp_echo.rsl` |

## Adapting one

1. Copy the closest example.
2. Rewrite the metadata for your finding (`name`, `severity`, `cve`/`cwe`,
   `references`, `cvss`/`cvss_score`, a single `mitigation`, `family`, `version`).
3. Adjust `host`/`port`/`payload` (use `payload bytes "<hex>"` for control or
   binary bytes) or the HTTP `path`.
4. Tighten matchers to cut false positives — add `not_contains` guards.
5. Add `evidence` for the report.
6. Prove it both ways — see [Testing Your Scripts](testing.md).

Full syntax is in the [Language Reference](reference.md).
