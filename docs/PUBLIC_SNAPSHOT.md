# Public snapshot

This tree is a **scrubbed snapshot** of the private ATEMS repository. It is not a continuation
of private git history. The source commit SHA is recorded only on the operator build host under
`public-exports/_tools/atems-source-sha.txt`.

## Configure before you run

1. Copy `config/hosts.example.env` to `config/hosts.env` and set your hostnames (do not commit `hosts.env`).
2. Copy `.env.example` to `.env` and set **`POSTGRES_PASSWORD`**, **`SECRET_KEY`**, and admin passwords to real secrets (use `change-me` only in the example file).
3. Set **`SQLALCHEMY_DATABASE_URI`** (or `DATABASE_URL`) to your PostgreSQL instance.
4. Set **`PORT`** if you expose the API on a non-default host port (default in Compose is `5000`).

Public URL example: `https://atems.example.com` ([RFC 2606](https://www.rfc-editor.org/rfc/rfc2606)).

## Example addresses (RFC 5737)

| Role | Example variable | Address |
|------|------------------|---------|
| ATEMS app host | `ATEMS_HOST` | `192.0.2.22` |
| Loopback | API / Postgres bind | `127.0.0.1` |
| All interfaces | Gunicorn bind | `0.0.0.0` |

## Dropped or rewritten

- `docs/REPO_HOST_MAPPING.md` (cluster inventory)
- `.cursor/` operator rules tied to this deployment
- Non-example env files (`.env`, `.env.compose-host-user`, etc.)
- `MYSQL_CREDENTIAL_CHANGE.md` replaced with a generic rotation note (no live hosts or passwords)
- External `tunnel_network` removed from Compose; services use a local `app` network only
