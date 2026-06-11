# Self-Hosting a Registry

The registry is the [`ruso-backend`](https://github.com/Hopeless-Labs/ruso-backend)
service — an Axum (Rust) API backed by **Postgres** and **Redis**, with GitHub
OAuth for login. You'd run your own to host a private library of checks, or to
develop against a local instance.

This is a summary; the backend repo's `README` and `docs/` are the source of
truth for production deployment.

## Stack

| Piece | Role |
|-------|------|
| Postgres | Stores checks, versions, users, tokens |
| Redis | Sessions, rate limiting, download counters |
| GitHub OAuth | User login (issues session tokens and PATs) |

## Development quick start

```bash
git clone https://github.com/Hopeless-Labs/ruso-backend.git
cd ruso-backend

# 1. Bring up Postgres + Redis (host ports 5432 + 6379).
docker compose up -d

# 2. Configure env.
cp .env.example .env
#    Required: GITHUB_CLIENT_ID, GITHUB_CLIENT_SECRET, GITHUB_REDIRECT_URI
#    Optional: COOKIE_SECURE=false for plain-HTTP dev.

# 3. Run the server (defaults to 127.0.0.1:8080).
cargo run
```

Key environment variables:

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | Postgres connection string |
| `REDIS_URL` | Redis connection string |
| `BIND_ADDR` | Listen address (default `127.0.0.1:8080`) |
| `GITHUB_CLIENT_ID` / `GITHUB_CLIENT_SECRET` / `GITHUB_REDIRECT_URI` | OAuth app credentials |
| `COOKIE_SECURE` | Set `false` for plain-HTTP local dev |
| `RUST_LOG` | Log filter, e.g. `ruso_backend=debug` |

## Pointing the CLI at it

Once it's running, aim `ruso` at your instance per-command or via env:

```bash
export RUSO_REGISTRY_URL=http://127.0.0.1:8080
ruso whoami
# or per command:
ruso --registry http://127.0.0.1:8080 search "redis"
```

Credentials are stored per registry URL, so a local instance and the hosted
registry coexist without clobbering each other. See
[Publishing & Installing](publishing.md) for the day-to-day workflow.

## Production

For production, the repo ships `docker-compose.prod.yml` and a `Dockerfile`, and
expects TLS termination in front, real GitHub OAuth credentials, and
`COOKIE_SECURE=true`. Follow the deployment section of the
[`ruso-backend` README](https://github.com/Hopeless-Labs/ruso-backend).
