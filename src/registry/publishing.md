# Publishing & Installing

The registry is how scripts are shared. It stores compiled scripts under a
`<namespace>/<name>` slug, versioned with SemVer, and lets anyone search,
install, and run them. The hosted registry lives at
`https://ruso.hopeless-labs.com`.

This chapter is the *workflow*. For every flag, see the
[CLI Reference](../cli/reference.md).

## Addressing a script

Everything in the registry is addressed the same way:

```text
<namespace>/<name>[@<semver-range>]
   alice   /log4shell @ ^1.2
```

- **namespace** — your username (the registry has no separate orgs; put an
  organisation name in the script's `author` field instead).
- **name** — a slug derived from the script's metadata `name`.
- **range** — an optional SemVer range; omit it to mean "newest non-yanked".

## Logging in

Authenticate once per registry. Use a Personal Access Token (PAT) or a session
token from the backend's web flow:

```bash
echo "ruso_pat_…" | ruso login
```

The credential is stored per registry URL in
`$XDG_CONFIG_HOME/ruso/credentials.json` (mode `0600`), so the same machine can
be logged into a local backend *and* the hosted one simultaneously. Check who
you are with `ruso whoami`; clear it with `ruso logout`.

## Publishing

Publishing requires a `version` in the script's metadata. The namespace defaults
to your username:

```bash
ruso publish ./myscript.rsl --visibility public
```

The CLI uploads the **source**; the registry compiles and stores it. To publish
a new version, bump `version` in the metadata and run `publish` again. Before
publishing, make sure the script passes both the vulnerable and safe cases — see
[Testing Your Scripts](../rsl/testing.md).

> **Tip:** the slug comes from the metadata `name` (lowercased, hyphenated, max
> 39 chars). Keep `name` short and use `report` for a longer human title.

## Finding scripts

Free-text search with optional filters (tag, severity, CVE, namespace, family):

```bash
ruso search "log4j" --tag rce
ruso search --family database --severity high
```

`ruso info <ns>/<name>` shows a script's versions, tags, and a ready-to-paste
install snippet.

## Installing and running

`install` downloads a version into the local cache
(`~/.ruso/scripts/<ns>/<name>/<version>.rbc`):

```bash
ruso install someuser/log4shell@^0.2
```

But you rarely need to install explicitly — `scan` and `exec` accept a registry
reference directly and fetch on a cache miss:

```bash
ruso scan --script someuser/log4shell --target https://target.example.com -v
```

Filesystem paths always win over reference matching, so a local file or
directory named like a slug still works.

### Scanning a whole family

Run every published script in a category against a target:

```bash
ruso scan --family web --target https://target.example.com
```

## Managing your published scripts

| Command | Effect |
|---------|--------|
| `ruso yank <ns>/<name>@<version>` | Hide a version from new installs (idempotent, owner-only). Existing installs keep working. |
| `ruso unyank <ns>/<name>@<version>` | Restore a yanked version. |
| `ruso edit <ns>/<name>` | Update description / visibility of a script you own. |
| `ruso pat list/create/revoke` | Manage Personal Access Tokens from the terminal. |

## Pointing at a different registry

Registry URL precedence: `--registry <url>` > `$RUSO_REGISTRY_URL` > the built-in
default (`https://ruso.hopeless-labs.com`). Use `http://127.0.0.1:8080` for a
local or private registry instance.
