# WHO backend direction

## Recommended starting stack

**Cloudflare Workers + D1 + R2 + Cloudflare Access**

This matches the infrastructure already used for Squashberry's other app and keeps the public site, API and admin boundary in one ecosystem.

### Workers

Use Workers as the public API edge.

Responsibilities:

- /config — remote app/version/welcome/announcement configuration
- /presence/heartbeat — lightweight device presence
- /activation/* — future entitlement/license checks if required
- /intelligence/lookup — authenticated caller-name lookup
- /intelligence/contribute — opt-in contact-name contribution
- /reports/* — moderation/reporting
- /admin/* — admin API protected by Cloudflare Access
- /backup/* — backup metadata/status only; Google Drive credentials should not be exposed to the public site

### D1

Use D1 for the first production data model.

Suggested core tables:

- app_config
- announcement_versions
- users
- devices
- number_identity_candidates
- identity_votes
- number_reports
- audit_log

The contact-intelligence layer should not be a raw copy of everyone's address book.

A better model is:

1. Normalize a phone number to E.164.
2. Generate a server-side keyed HMAC lookup key.
3. Store candidate-name contributions separately from the user/device record.
4. Aggregate multiple contributions into confidence-weighted candidates.
5. Keep contribution source details to the minimum needed for abuse prevention and deletion.
6. Give users a clear opt-in and an opt-out/delete path.

Do not put a phone-number hashing secret in the mobile app.

### R2

Use R2 for larger server-controlled objects that should not live in D1:

- exported moderation reports
- aggregate analytics snapshots
- future server-generated datasets
- optional encrypted server-side exports

R2 is not necessary for the basic Google Drive contact-backup flow.

## Google Drive backups

The planned user-owned backup flow can use the Google Drive API.

Google provides an appDataFolder location specifically for application-specific files. It is hidden from normal Drive browsing and is only accessible to the app that created it. The Drive API documents the drive.appdata scope for this use case.

Recommended WHO flow:

1. User taps Back up to Google Drive.
2. User authenticates their Google account.
3. WHO creates an encrypted backup package locally.
4. The app uploads the package to the user's Drive app-data area.
5. WHO stores only the minimal backup metadata needed to locate/version the backup.
6. Restore downloads the latest backup and decrypts it locally.

A future advanced option can offer an explicit user-visible WHO Backups folder, but the private app-data location is a simpler default.

Do not make Google Drive a server-side storage proxy for every user.

## Remote version/update contract

The app can consume a JSON object shaped like the public config.example.json in this repository.

The mobile client should fail open for existing users when this endpoint is unavailable, except for a locally known forced-update state that was already delivered. A temporary API outage must not brick the app.

## Admin control center

The public repository contains /admin/ as a UI foundation only.

The production version should sit behind Cloudflare Access and use a server-only admin API.

Planned tabs:

- Overview
- Users
- Caller Intelligence
- Announcements
- Versions & Updates
- Contacts
- Backups
- Reports
- Security
- Settings

The admin UI should never contain a database password, Worker secret, service-account credential or Google refresh token.

## Why Cloudflare first

Cloudflare's current Workers Free plan includes limited Workers usage, D1 is available on Free, and D1 currently includes 5 million row reads/day, 100,000 row writes/day and 5 GB of stored data on Workers Free. R2 currently has a free allowance of 10 GB-month storage, 1 million Class A operations and 10 million Class B operations per month, with free egress.

That is a practical starting point for an early WHO beta.

## Where Supabase fits

Supabase is a strong alternative if the intelligence data model becomes heavily relational and you want managed Postgres/RLS/auth in one product. Its current Free plan includes 500 MB database storage, 1 GB file storage, 50,000 MAU and 500,000 Edge Function invocations, with two active free projects; inactive free projects can pause.

For WHO's current beta, Cloudflare keeps the architecture closer to the existing environment. Supabase remains a sensible future migration target if D1/Postgres flexibility becomes the bottleneck.

## Where Firebase fits

Firebase is strongest when the product wants deep Google/mobile-native services such as Analytics, Crashlytics, FCM and Google-managed authentication. Firestore also has a free tier.

For the specific WHO plan, Firebase is not required for Google Drive backup: Drive backup is a Google Drive API integration and can be implemented independently. Keeping the core data plane on Cloudflare avoids adding a second cloud platform solely for Drive.

## Security boundary

Public:

- product website
- release/download information
- store badges
- privacy documentation

Authenticated app API:

- version config
- caller lookup
- opt-in contribution
- reports
- user/device heartbeat

Admin-only:

- moderation
- global config writes
- version policy writes
- announcement writes
- analytics
- user support operations
- audit logs

Secrets:

- Cloudflare Worker environment secrets
- admin/API signing keys
- Google OAuth secrets when needed

Never store these in this public repository.
