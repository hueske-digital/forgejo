# Forgejo

This repository contains [Forgejo](https://forgejo.org) – a self-hosted lightweight
software forge (Git hosting).

## Setup instructions

1. Copy `.env.example` to `.env` and fill in the values:
   ```
   cp .env.example .env
   ```
   Set `FORGEJO_DOMAIN` (and adjust `FORGEJO_SSH_PORT` if 222 is taken).

2. Start the stack:
   ```
   docker compose up -d
   ```
   Forgejo generates its `SECRET_KEY` / `INTERNAL_TOKEN` on first run and stores
   them in the `forgejo_data` volume (`/data/gitea/conf/app.ini`).

Setup a host in Caddy pointing to port 3000.

```
# EXTERNAL SERVICE WITH CLOUDFLARE PROXY #

https://git.example.com {

    # import logging
    import cloudflare
    import tls
    import compression
    import header

    handle @cloudflare {
        reverse_proxy forgejo-app-1:3000
    }
    respond 403
}
```

Git over SSH is exposed directly on the host (`FORGEJO_SSH_PORT`, default 222),
since Caddy only proxies HTTP. Clone URLs use that port automatically.

## Login via Pocket ID (OIDC)

Forgejo supports OpenID Connect with auto-discovery, so Pocket ID is configured
in the web UI (no env needed):

1. In Pocket ID, create an OIDC client with the callback URL:
   ```
   https://<FORGEJO_DOMAIN>/user/oauth2/pocket-id/callback
   ```
2. In Forgejo: *Site Administration → Authentication Sources → Add*, choose
   **OAuth2** with provider **OpenID Connect**, name it `pocket-id`, and set:
   - Client ID / Client Secret from Pocket ID
   - Auto Discovery URL: `https://<pocket-id>/.well-known/openid-configuration`

New users can only register through Pocket ID
(`ALLOW_ONLY_EXTERNAL_REGISTRATION=true`); the local registration button is hidden.

## Backups

The PostgreSQL database is dumped daily at 01:00 via Ofelia
(`pg_dumpall` to `/var/lib/postgresql/backup.sql` inside the `db_data` volume).
Repository data and config live in the `forgejo_data` volume – back this up
separately.
