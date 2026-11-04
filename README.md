# ES Startup Competition 2026 — Digital Judging

CAO / Startup Competition operations. Filtration: 17 November 2026, 17:00–20:00 Africa/Cairo. Five rooms, approximately 50 startups, two or three active judges per room. The finalist target is a configurable 6 or 8 and remains a leadership decision.

## Architecture and deployment

React 19, TypeScript and Next-compatible Vinext on Cloudflare Workers, Tailwind/CSS, Cloudflare D1 (SQLite) and platform-managed Sign in with ChatGPT. This deployable stack uses the connected Sites hosting/database/authentication instead of requiring unconnected Supabase and Vercel credentials. It is not a Supabase/PostgreSQL deployment. Source is kept in a managed Git repository; a separate GitHub publication may require the owner to create an empty repository because the connected GitHub tool does not offer repository creation.

Production hosting manifest: `.openai/hosting.json`. `DB` is a durable server-side database binding. Schema is in `db/schema.ts`, with an immutable migration under `drizzle/`. `app/api/platform/route.ts` is the authenticated, role-authorized API. `app/scoring.ts` is the shared scoring implementation. `app/page.tsx` contains administrator and judge interfaces. All writes are server-side; no privileged credentials are shipped to the browser.

## Access setup — required before event

1. Open the published site and sign in using **ismail.alabasy@gmail.com** or **ismailalabasy@aucegypt.edu**. These two verified owner emails have initial administrator access. No password is generated or stored by the app.
2. Sites starts owner-private. In the Site's sharing/access settings, invite the actual judges and additional administrators using their verified sign-in email. Do not make the app publicly writable. This hosting visitor policy and the app role list are separate gates; a person needs both.
3. In **Exports & setup → Administrator access**, the owner can add leadership administrators by name/email. Only the two owner accounts can grant/revoke administrator roles. Administrators see all scores and feedback, can reopen submissions and change settings. Revoke the hosting invitation as well if removing a visitor entirely.
4. In **Judges**, add each actual judge with the same email used for sign-in, room, organization, title and optional HTTPS LinkedIn profile. Their first successful sign-in binds the verified email to a stable Site-specific user identity. Later logins must match that identity. An unregistered or disabled visitor receives a denied-access response.
5. Judges need a ChatGPT account and must be invited through Site sharing. External-judge invitation, real identity-provider login and simultaneous use on their devices require a pre-event rehearsal; the automated API tests substitute trusted edge identity headers, not an actual OAuth login.

Authentication is owned by the hosting platform. Do not host the built Worker directly behind a proxy that permits visitors to forge `oai-authenticated-user-*` headers. If migrating to Vercel/Supabase, replace this identity boundary and SQL adapter before use.

## Rehearsal

From the empty Overview or Exports & setup, choose **Load fictional rehearsal data**: creates 50 clearly labeled fictional startups, five rooms and 13 fictional judges (three/two/three/two/three). No real judge contact information is invented. In Judges, **Rehearse as judge** lets an administrator test draft/submit using fictional judges only. This endpoint explicitly rejects impersonation of real judges. Download rehearsal backups if useful, then **Clear fictional data** before importing actual entries. Real records are preserved. Demo deletion requires the on-screen confirmation.

Production begins empty: the code does not silently seed competition data or create a public demo account.

## Rooms, startups, CSV imports

Create/edit up to five rooms first. Add 2–3 active judges and assign startups to each. Rooms with fewer than two active judges are flagged. The server rejects a fourth active judge. Judge/startup room assignments cannot change after evaluations exist; do assignment changes before judging. Records with evaluations cannot be deleted. Unassigned startups are visible to administrators but not judges.

Add/edit startup name, team, description, problem, solution, category and an optional HTTPS pitch-deck URL. Linked files must be shared separately with judges; the app stores a link and does not upload or change sharing of the linked document.

**Exports & setup → Import competition information** supplies UTF-8 CSV templates. Startup columns: `name,team,description,problem,solution,industry,room,deck`. Judge columns: `name,email,organization,title,linkedin,room`. Room can be the exact room name or ID. Imports create entries sequentially, stop on the first failure and report how many rows were saved. Avoid reimporting successful startup rows, since startup names are not unique IDs. Judge email duplicates are rejected. CSV quoting/newlines are supported. Imports are not atomic across the entire file.

## Judge workflow

Sign in → see your room, other judges' professional profiles and assigned startups → open startup → rate all seven criteria on the same page. Information buttons reproduce the finalized behavior anchors from `ES_Startup_2026_Filtration_Judging_Guide_BOXES.pdf`. Traction, a completed MVP, precise forecasts and a complete GTM are not required of idea-stage teams.

Ratings and comments autosave after a short pause. Wait for **All changes saved**. Every save includes an optimistic version; simultaneous tabs cannot silently overwrite each other. A conflict requires reloading after downloading any unsaved local copy. A failed save displays Retry and Download local copy. The app warns before leaving the browser with pending changes, and the in-app Back action waits for pending saves. There is no automatic offline replay; use printable scorecards if connectivity fails.

Submit requires seven integer ratings (1–5) and a separate sustainability Yes/No. Submission locks the evaluation server-side. Judges never receive other judges' scores or feedback through the API, including after submission. Administrators may reopen with a mandatory reason; this increments the version and records an audit entry. Judges should reopen the startup page after an administrative reopening.

## Scoring and review

| Criterion | Weight |
|---|---:|
| Problem Identification & Significance | 20 |
| Customer Understanding | 15 |
| Solution & Value Proposition | 20 |
| Validation & Evidence | 15 |
| Business Model & Feasibility | 15 |
| Market Opportunity & Growth Potential | 10 |
| Presentation & Communication | 5 |

Total = sum(rating / 5 × weight), maximum 100. A complete all-1 evaluation scores 20, not zero. A partially filled draft shows a clearly provisional preview and never contributes to the room average. Server responses, administrator summaries and CSV exports share the same scoring code. Sustainability is unweighted and is never added to this total.

Startup average = arithmetic mean of submitted scores from its currently assigned active judges. Missing submissions are excluded, not counted as zero. Required/submitted counts accompany every average. Fully evaluated requires 2–3 active assigned judges and all submissions, or at least one submission plus an explicit administrator completion exception with a reason. Exceptions are visible in results and audit history.

Overview refreshes every 10 seconds; manual Refresh is available. Rankings are by descending within-room average, with exact displayed scores for ties. Incomplete averages are provisional. The configurable disagreement flag uses max minus min submitted score and never changes a score. Open each startup to see every judge's ratings, comments, sustainability answer and submission status. Judge/room/startup/completion filters are provided.

In **Finalist review**, open startup details and manually select/remove finalists with the leadership panel's reason. No automatic finalist selection. Default target 6 is a working app default, not an event decision; change to 8 if confirmed. Exactly the configured count is required before final result locking. Final locking blocks evaluation and record changes; administrators can unlock, with an audit entry.

## Export, backups and recovery

Download startup results CSV, detailed evaluation CSV (including drafts, ratings, sustainability, comments and timestamps), and a full JSON backup from Exports & setup. CSV uses formula-prefix escaping. CSV opens in Excel; native XLSX generation is not implemented. Print is available; retain the approved one-page paper scorecard as the primary emergency fallback. Keep backups locally before judging, periodically during judging and after locking results. No automatic scheduled backups or recovery SLA is configured.

Full JSON backup contains all records, all drafts/submissions and full audit history. Audit UI shows only the latest 500 events. Draft/submit audit entries are labeled attempts; only the versioned evaluation row proves a successful save. The application audit is retained through demo cleanup, but it is not externally tamper-proof against database operators.

Operator recovery: stop judging, preserve a fresh backup of the target, run `node scripts/restore-backup.cjs BACKUP.json restore.sql`, inspect the generated SQL and apply it through the authorized database administration workflow to the intended D1 database in one transaction/batch. The script only generates SQL; it never connects to production. Test recovery in an isolated database first. Restore replaces competition tables and must never be run casually against a live event. Site version rollback changes code, not database contents.

## Local development and checks

Requires Node >=22.13.0, Git and the installed dependencies. Use `npm run dev`, `npm run build` and `npm run db:generate`. The checkout is configured for the managed Linux pnpm environment; use the platform's install/build helpers there. Apply the migration to local D1 using the starter's Wrangler local migration workflow. Local/production data are separate.

`node scripts/test-platform.cjs` compiles the actual API and scoring modules against real in-memory SQLite, using the actual migration and D1-shaped adapter. It covers 50 startups, 13 judges and five rooms, authorization, private payloads, drafts, validation, submission lock, optimistic concurrency, mean scores, missing submissions, exceptions, manual finalist selection, results lock, backups and demo cleanup. Authentication headers are mocked at the trusted edge boundary. It does not prove real identity-provider login or platform sharing.

Run `node node_modules/typescript/bin/tsc --noEmit` for type checking. Browser QA checks are reported separately; don't infer that unit tests prove mobile device or external judge login behavior.

## Pre-event operational checks

Invite actual judge emails; confirm they can sign in independently; verify room codes and startup data; choose 6 vs 8; rehearse all judges saving/submitting concurrently; confirm a backup download and isolated restore; print fallback scorecards; confirm Wi-Fi and charged devices. No in-app conflict declaration or recusal feature is implemented, as requested.

## Mobile judge interface and safe preview (4 Nov update)

Open `/judge-preview` on the published origin to try a fictional evaluation without adding competition records. On desktop it shows an interactive phone frame with 320/375/390/430px width options; on a phone it opens the full-width judge form. `/judge-preview?view=full` always opens the form directly. Site-level sign-in/access still applies. The preview uses the same `app/components/JudgeEvaluation.tsx` as the live judge portal, with a separate local-device-only save adapter and a prominent PREVIEW banner. It never calls the evaluation API. Reset clears only the preview's browser storage. Real judge scores continue to use authenticated, versioned database writes.

Mobile changes include a compact live judge header, 48px-high rating buttons, minimum 44px explanation/back controls, 16px comment input, collapsible startup profile, live save announcements, and a sticky weighted-score/submission bar with safe-area spacing. All seven criteria remain on one scrollable page. Saving is also attempted when the page becomes hidden, and evaluation fetches request keepalive; this is a best effort, not an offline or mobile-OS delivery guarantee.

Browser checks on 320, 375, 390 and 430px iframe viewports found no horizontal document overflow. Minimum rating target at the smallest tested viewport was approximately 44.6 × 48px. Preview checks covered rating changes, guide opening, all seven ratings, sustainability Yes, comments, local draft persistence after reload, 84/100 calculation for [5,4,4,4,4,4,4], submission, disabled controls and persisted lock. These are narrow-browser layout and preview checks, not physical iOS/Android device or real-judge authentication tests. Complete those with judges before the event.
