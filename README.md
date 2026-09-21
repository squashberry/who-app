# WHO — Public Product Website + Control Center

WHO is a beta communication app created by Squashberry.

This repository contains the public product website and a front-end foundation for the future WHO admin/control center.

## Live site

GitHub Pages target:

https://squashberry.github.io/who-app/

## Repository layout

- `index.html` — public WHO product page
- `styles.css` / `script.js` — visual system and interactions
- `privacy.html` — beta privacy direction
- `admin/` — admin/control-center UI foundation
- `assets/who-mark.svg` — brand mark

## Important

The admin page is currently a **local preview UI**. It does not contain production credentials and it does not yet connect to a WHO backend.

The planned backend can expose a small configuration/API surface for:

- app version + minimum version
- force-update messaging
- beta welcome revisions
- announcements and maintenance mode
- user/heartbeat statistics
- caller-intelligence moderation
- opt-in contact contribution
- Google Drive backup/restore status
- reports and moderation
- product settings

## Planned architecture

The existing WHO project can use:

- Cloudflare Workers for the API
- D1 for relational operational data
- R2 for larger private objects
- Cloudflare Access for admin authentication
- Google Drive API for user-owned backups

The public website should never contain admin secrets.

## Beta positioning

The site deliberately marks contact intelligence and Google Drive backups as **NEXT** rather than pretending they are already shipped.
