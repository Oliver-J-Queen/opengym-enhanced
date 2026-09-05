---
title: openGym Enhanced — Current App Status
tags:
  - opengym
  - project-status
  - self-hosting
  - workout-tracking
created: 2026-09-05
source_version: 1.3.1
---

# openGym Enhanced — Current App Status

> [!summary]
> **openGym Enhanced** is a personalized fork of [openGym by Duarte Santos](https://gitlab.com/DuarteSantos8/opengym). It is a polished self-hosted workout tracker: plan workouts, follow them while training, record progress, and optionally sync private profiles through a small server. The checked-out source is **v1.3.1** (released 2026-09-04).

## What this fork currently changes

- GitHub home: `Oliver-J-Queen/opengym-enhanced`.
- The README gives clear credit to the upstream project and preserves its AGPL-3.0 license and notices.
- GitHub Actions retains only the automated test workflow. Automatic public demo deployment and container-image publishing have been removed.
- No product features have been changed yet; this is still the upstream feature set, ready for personal enhancements.

> [!important]
> The supplied Compose file still names prebuilt images from GitLab. To run **this fork's code**, use `docker compose up -d --build` rather than `docker compose pull`. The build uses the local checkout and does not need a published image.

## Product shape

There are two ways to use the app.

| Mode | What it provides | What it does not provide |
| --- | --- | --- |
| **Self-hosted web/PWA** | Browser app on a phone or computer; passkey sign-in; profiles; device sync; optional push alerts and admin controls. | Native-only installation without a server. |
| **Standalone Capacitor mobile app** | A locally stored Android app (and developer-built iOS app); workout-day reminders; local backup/export. | Account, browser/device sync, or server-side multi-user features unless it is paired to a self-hosted instance. |

For one person or a few friends, self-hosting is optional. A local-only mobile build is the least operational work. A small HTTPS instance is the sensible route when people want to use both phone and laptop or need separate profiles.

## Everyday training experience

### Plans and routines

- Build a weekly schedule with a routine on each weekday; the first day of the week is configurable as Monday or Sunday.
- Choose from a searchable library of about **1,324 exercises**, including body-part, equipment, instructions, and animated demonstrations. The library can be filtered to the equipment available at a home gym or commercial gym.
- Create custom exercises with a name, body part, and optional written description. Custom exercises participate in plans, history, statistics, exports, and imports like built-in exercises.
- Each routine holds exercises, working sets, reps or timed sets, target load, rest time, progression policy, warm-up configuration, notes, and optional superset grouping.
- Reorder exercises by drag/long press. Superset groups move together.
- Mark a routine as a **planned deload** so it remains in history but is excluded as a baseline for automatic progression afterward.
- Share a plan as a small file or a print-friendly PDF. Importing a shared plan merges it: existing routines are not overwritten.

### Starting and completing a workout

1. Home identifies today's planned session, including date-specific rescheduling. On a rest day it shows the next scheduled workout.
2. Starting a workout can ask for body weight, then creates set rows from the routine and pre-fills targets from previous training/progression rules.
3. During training, sets can record load, reps, time, optional RIR/RPE effort, warm-up status, and notes. It detects personal records and reports them at the end.
4. A rest countdown begins after sets; individual exercises can override the global rest time. Supersets run members back-to-back and rest once after a round.
5. Finish writes the completed workout to the profile history. The new history becomes the input for statistics and the next workout's targets.

The workout screen is intentionally flexible: add, remove, move, or swap exercises mid-session; turn a neighboring pair into a superset; switch between exercises by swipe; and use a freestyle session when no routine fits.

### Exercise-specific behaviours

| Training case | How it works |
| --- | --- |
| **Strength sets** | Standard weight × reps logging, target prefill, PR detection, estimated 1RM, and optional progression. |
| **Warm-up sets** | Marked separately, visible in the session, but excluded from estimated 1RM, progression calculations, volume, and fatigue. |
| **Supersets** | Can be planned or made during an active session. The app advances through members and rests once at group completion. A group shrinks away when only one member remains. |
| **Timed exercises** | Planks, hangs, wall sits, and carries use a work timer and record actual duration. They may still have added load. |
| **Bodyweight exercises** | No empty load field for ordinary push-ups/pull-ups/dips. Progression increases reps, then sets; adding a belt/vest turns load back on. |
| **Per-side work** | The user logs the overall total and sees the per-side split. Targets increment in twos so each side has a whole number of reps. |
| **Barbell work** | Bar/EZ/trap/Smith bar weight is configured per exercise; the screen calculates the plates needed per side while history stores total load. |
| **Cardio** | Records duration and speed rather than pretending cardio is a weight × reps lift. |
| **Past workouts** | History can backfill a workout with its original date, time, duration, routine/freestyle status, sets, and notes. If a session exists that day, the user chooses replacement, coexistence, or cancel. Backfilled sessions do not create misleading later PRs. |

## Progression and performance logic

The progression engine lives in tested, pure frontend functions. It calculates targets from the training history each time rather than storing fragile derived targets.

- **Linear progression:** add a configurable increment after successful work.
- **Greyskull LP:** AMRAP top set, double jumps, and percentage resets after stalls.
- **Double progression:** work within a visible, editable rep range; increase load after the range is achieved.
- **Timed progression:** grow target duration for timed work.
- A routine has a default rule, and individual exercises can override it.
- Missed reps do not advance weight. Stalls lead to a deload according to the active rule.
- Bodyweight movements progress in reps/sets until the configured ceiling suggests additional load or a harder variation.
- An explanation accompanies targets (for example, why a load advanced, held, or deloaded).

It also calculates an estimated one-rep max per exercise. It uses the best eligible set, identifies that source set, offers a calculator, and refuses high-rep estimates above 12 reps where the formula would be less meaningful.

## Statistics and feedback

- **Body-weight tracking:** chart, configurable goal line, and colour-coded movement toward/away from the goal.
- **Exercise history:** per-exercise performance/estimated-1RM progress, including bodyweight-aware displays.
- **Activity heatmap:** GitHub-style annual calendar, shaded by training time.
- **Muscle map:** front/back body diagrams with a selectable figure and three readings:
  - **Balance:** training volume by muscle over a selected period, including neglected muscles.
  - **Fatigue:** recovery estimate that weights hard/near-maximal work more heavily and decays smoothly over time.
  - **Strength:** time since each muscle was trained plus the exercises and estimated 1RMs associated with it.
- **Effort statistics:** optional RIR or RPE tracking, average/weekly distribution, and effort context on exercise trends. RIR/RPE is descriptive only; it does not silently change progression or 1RM calculations.
- Routine editing previews which muscles the routine will train; completing a workout shows what was trained.

## Data entry, import, export, and privacy

### Import and export

- One-tap JSON profile export/import for backup or migration.
- CSV import for **FitNotes**, **Strong**, **Hevy**, and simple custom columns.
- Direct Hevy import with a Hevy Pro API key; the key is used only for that import and is not saved.
- Apple Health body-weight export import.
- Recognized exercises map to the library. Unrecognized exercises become custom exercises so rows are not discarded.
- Existing workout days are protected during imports; imported routines are added rather than replacing current plans.

### Where self-hosted data lives

The server uses no database service. Its persisted files are in the host's `./data` directory:

- `db.json`: profiles and public passkey records.
- `state-<user>.json`: each profile's plans, workouts, body weight, and settings.
- `secret`: the session-cookie signing/encryption secret.
- `audit.log`: optional administrator activity events.
- `vapid.json`: automatically generated push-notification keys.

Backing up `./data` backs up the instance. Passkey private keys never go to the server: they remain on the person's device/password manager.

> [!warning]
> The app downloads about 140 MB of exercise images and GIFs from the upstream exercise dataset on first run. The metadata/instruction text is MIT-licensed, but the image/animation rights are separately unresolved/third-party. The project does not redistribute those files; see `NOTICE.md` before reusing the media outside normal app use.

## Accounts, sync, and friend-sized hosting

### Authentication and device sync

- Web users sign in with **passkeys** (WebAuthn), not passwords. Sessions last 90 days by default and can be ended everywhere from Settings.
- Profiles are isolated server-side and sync their state across signed-in browser devices.
- WebAuthn requires HTTPS for any non-localhost hostname. `http://localhost:8080` works on the computer running Docker; a phone cannot use a plain LAN-IP address for passkeys.
- The standalone mobile app pairs to a self-hosted account through a short, one-time pairing code created in an authenticated browser. It uses a bearer token in its WebView because WebAuthn is not usable there.
- Guest mode stores data only in that browser: no account, no sync, no server contact. An instance can disable it.

### Optional administrator controls

An ordinary personal instance needs none of these. If enabled by environment variables, an admin can:

- View users, recent activity, history, and body weight.
- Disable an account (which signs it out and prevents further access).
- Turn on invite-only registration and issue/revoke invite codes.
- Enable/disable the AI Coach instance-wide.
- Review or clear an activity log of authentication and admin events.

The activity log excludes IP addresses by default; network/full IP logging is opt-in. Guests never appear because they never contact the server.

### Notifications and phone usability

- Opt-in browser push alerts for a completed rest timer, including when the browser is not foregrounded.
- Optional workout-day reminder when a planned workout is still incomplete; Android schedules it by calendar date so completed/rescheduled days remain quiet.
- Optional visual screen flash when rest/work timers end.
- Optional wake lock prevents the screen sleeping during an active workout. It needs HTTPS or localhost and may be unavailable under iOS Low Power Mode.
- Light/dark theme and eight accent colours are profile settings.

## Languages and accessibility-oriented details

- Full interface translations: English, German, Spanish, French, Italian, Portuguese (Portugal), Portuguese (Brazil), Polish, Turkish, Russian, Chinese, Korean, Hindi, Thai, and Hungarian.
- Exercise instructions are localized in 12 languages; built-in exercise names have additional bilingual handling for Brazilian Portuguese and Hungarian.
- Locales are loaded on demand rather than all shipped upfront.
- The app supports keyboard use in areas such as the muscle map and handles mobile-specific navigation/gestures.

## Optional AI capabilities

### 1. Local read-only MCP server

`mcp/` is a separate, opt-in Model Context Protocol server. An MCP-capable client (for example, Claude Desktop or Cursor) starts it locally and grants it read-only access to the selected `./data` directory.

It can answer eight categories of question: routines, week plan, workout list/detail, body weight, estimated 1RM, and muscle balance. It uses the same pure calculation functions as the UI, so values should match the Stats screen. It adds no Compose service, makes no network requests, and does not expose passkeys, session secrets, or push keys. It cannot write data yet.

### 2. Server/device AI Coach

The AI Coach is **off by default** and is not needed for normal tracking. An administrator can enable it, choose a provider, then each user consents and completes a short training intake. It can draft a week of routines, chat about refinements, review training, and debrief a single workout.

Supported providers:

- Anthropic API, OpenAI API, Google Gemini API, or an OpenAI-compatible endpoint (Ollama, LM Studio, vLLM, OpenRouter, or a private gateway) on the default API image.
- Claude Agent SDK or Codex CLI in the larger `coach` API image.
- In the standalone mobile app, either pair with the user's own self-hosted instance or use a personally supplied API key stored in secure device storage.

Safety and privacy boundaries implemented in the source:

- A named-field allowlist controls what can leave the server: relevant plan, training, body-weight, intake, and preferences data. It deliberately excludes display name/user ID, passkeys, credentials, push subscriptions, invite data, and unrelated profile data.
- Training review data is bounded to 12 weeks or 60 sessions.
- The model cannot modify workouts, weigh-ins, account settings, or arbitrary data. Returned proposals must match a closed list of plan/week change types, reference valid exercises/routines, and cite evidence.
- Changes are applied locally only after user approval, with a plan snapshot for undo. A stale proposal cannot overwrite later plan edits.
- Optional anonymous comparison requires both an admin switch and per-user opt-in; it reports only coarse medians/ranks once at least three people share.

> [!caution]
> The Coach provides training suggestions, not medical advice. It should not be treated as a doctor, physiotherapist, or injury-diagnosis tool. It can also incur provider/API costs when enabled.

## Technical architecture

```text
Phone / laptop browser (PWA)
        │ HTTPS / same origin
        ▼
web container: nginx
  ├─ serves compiled React application
  ├─ serves locally downloaded exercise media
  └─ proxies /api
        │
        ▼
api container: Node.js + WebAuthn + web-push
  └─ JSON files mounted as ./data on the host
```

| Component | Implementation | Responsibility |
| --- | --- | --- |
| `frontend/` | React 19, Vite, React Router, Zustand | PWA UI, local/guest state, workout logic, statistics, sync client, Capacitor build. |
| `api/` | Node.js without a web framework; `@simplewebauthn/server`, `web-push` | Passkey ceremonies, signed sessions, state storage/sync, pairing, pushes, optional administration and Coach gateway. |
| `web/` | Multi-stage Docker build + nginx | Builds frontend and serves it/proxies API under one origin, required for passkeys. |
| `mcp/` | Node.js stdio MCP server | Optional local, read-only assistant integration. |
| `media/` | One-shot Compose download volume | Locally cached exercise images/GIFs after first startup. |

The frontend's training calculations are mostly standalone functions with colocated tests. This is valuable for customization: progression, history, warm-up behavior, effort, muscle balance/recovery, import matching, and 1RM logic can be changed/tested without changing server persistence.

## API surface

The formal OpenAPI file is `api/openapi.yaml`. The main groups are:

- Health and configuration: `/api/health`, `/api/config`, `/api/me`.
- Passkey registration/login and session termination.
- Mobile pairing: `/api/pair/create` and `/api/pair/redeem`.
- Full profile-state read/write: `/api/data`.
- Push public key, subscription, test, rest-timer creation/cancellation, and activity updates.
- Admin user, invite, and audit-log operations.

The frontend remains the main product interface; the API is primarily an authenticated sync/service boundary, not a public third-party developer platform.

## Automation and quality checks in this fork

GitHub Actions currently runs `.github/workflows/test.yml` on pull requests and relevant pushes to `main`.

It checks:

- Frontend dependency install, unit tests, production build, locale/source-string checks, and a fatigue-model property probe.
- MCP dependency install, tests, and a plain-Node import/loadability check.
- API tests, generated Coach asset consistency, and core loadability.
- Docker builds for the default and Coach API targets, including boot/runtime assertions.

This repository deliberately no longer publishes Docker images or deploys a GitHub Pages demo. The upstream `.gitlab-ci.yml` remains in the repository but GitHub does not execute it.

## Known limits and deliberate omissions

- No managed cloud account/service; hosting and backups are the owner's responsibility.
- No database server; JSON-file storage keeps it simple but is not intended as a high-concurrency, large-gym platform.
- No native iOS distribution outside a developer-built app; the PWA is the supported self-hosted iPhone experience.
- MCP is read-only; assistant-driven logging/editing is planned but not shipped.
- AI Coach is optional and can have privacy/cost implications.
- Roadmap items not yet implemented include percentage/training-max programs (for example 5/3/1), additional starter plans, body measurements, per-exercise notes, and a built-in plate calculator (although per-exercise bar/plate math during workouts already exists).

## Practical recommendation for this fork

For a personal project shared with a small trusted group:

1. Build from this checkout with `docker compose up -d --build`; do not rely on upstream prebuilt images.
2. Use HTTPS and a stable hostname only if people need browser/PWA sync across devices. Set `RP_ID` and `ORIGIN` before registration because changing the hostname invalidates existing passkeys.
3. Leave admin, invite-only mode, notifications, MCP, and AI Coach off until a concrete need exists.
4. If inviting friends, enable `INVITE_ONLY=1` and `ALLOW_GUEST=0` after creating the first administrator profile.
5. Back up `./data` regularly. Treat it as private: it contains training history, public passkey records, session secret, and possibly encrypted Coach credentials/audit history.
6. Keep the lightweight GitHub test workflow; it is the right guardrail while personal features are added.

## Source references

- `README.md` — product feature list, configuration, data model, and quick start.
- `docs/SELF_HOSTING.md` — HTTPS, passkeys, multiple users, backup, notification, and deployment details.
- `docs/DATA_IMPORTS.md` — import formats and mappings.
- `docs/MOBILE.md` — Capacitor app build/distribution.
- `docs/AI_COACH.md` — provider setup, privacy boundaries, validation, and isolation.
- `mcp/README.md` — local read-only MCP interface.
- `api/openapi.yaml` — HTTP API contract.
- `CHANGELOG.md` — release-by-release detail; current source release is v1.3.1.
