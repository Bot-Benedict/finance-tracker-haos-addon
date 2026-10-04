# Finance Tracker

Household finance tracker. Home Assistant is the login: the add-on is reachable
only through HA's ingress proxy, so each household member signs in with their
own HA account (turn on multi-factor authentication under Profile → Security).

## Install

1. **Registry login (once).** The image is private, so the Supervisor needs
   credentials to pull it. Create a GitHub personal access token (classic) with
   only the `read:packages` scope; fine-grained tokens do not work with GHCR.
   In the Terminal & SSH add-on:

   ```sh
   read -rs GHCR_TOKEN   # paste the token, press Enter (not echoed, not in history)
   ha docker registries add ghcr.io --username Bot-Benedict --password "$GHCR_TOKEN"
   unset GHCR_TOKEN
   ```

   When the token expires, run `ha docker registries add` again with a new one.
2. Settings → Add-ons → Add-on store → ⋮ → Repositories → add
   `https://github.com/Bot-Benedict/finance-tracker-haos-addon`.
3. Install **Finance Tracker**. The Supervisor pulls the image tagged with this
   add-on's `version`; nothing is built on the box.
4. Configuration tab: set `anthropic_api_key`, save. Info tab: turn on Start on
   boot, Watchdog and Show in sidebar, then Start.

## Updates

A new version appears as a normal add-on update once it has been published.
Update from the add-on page.

## How it works

- `ingress: true` makes HA proxy requests to port 8099 inside the container and
  attach the signed-in user's identity (`X-Remote-User-*` headers).
- The app refuses any request not coming from the ingress proxy
  (`172.30.32.2`). No host port is published.
- `panel_admin: false` shows the sidebar entry to non-admin HA users too. It is
  visibility, not access control: any signed-in HA user can open the add-on's
  ingress URL.
- Data (`finance.db`) and the add-on options (including the API key) live in the
  add-on's `/data` volume, which Home Assistant backups include.
