# LFC-tickets — Project Notes

## What this is
A single-file web app (`index.html`) for a small group who share two Liverpool
season-ticket seats: it lists home fixtures and lets the group record who's using
each seat for each match.

## Architecture
- No build step, no framework — everything lives in `index.html`. Vanilla JS
  module + Tailwind (via CDN) for styling, Firebase Firestore for storage and
  live sync.
- `render()` rebuilds the whole `#app` DOM from a `currentFixtures` array on every
  state change (filter click, seat edit, or an `onSnapshot` update from another
  client).
- Firestore doc `liverpool-tickets/data` → `{ fixtures: [...], lastUpdated, version }`.
  Each fixture: `id, competition, opponent, date, kickoff, status, score, ticket1, ticket2`.
  `DATA_VERSION` gates a migration path in `init()` — bump it if the fixture shape
  ever changes, and see the merge logic there for how it avoids clobbering fixtures
  a newer client already saved.
- `scripts/backup-data.mjs` / `restore-data.mjs` (untracked, local-only) read/write
  the live Firestore doc directly over REST for manual snapshots — not part of the
  app itself.

## Standing rules for this project
These were set explicitly by Ian and should be re-confirmed (not assumed) at the
start of any future work session that touches this app:
- **Never touch the database** (Firestore, or Supabase/etc. if this ever migrates)
  as part of a code change — no schema/data changes, no live writes during
  implementation or testing. Read-only inspection of DB-related code is fine;
  actually connecting to live Firestore (even read-only) during implementation is
  not, unless Ian says otherwise.
- **Redesign or risky work happens on a separate branch**, never directly on
  `main`, and is **never merged into `main` without explicit approval**.
- Preserve existing functionality, APIs, persistence, and data contracts unless a
  functional change is explicitly approved — a redesign is UI/UX only by default.
- Before any destructive git operation (discarding a branch, resetting), confirm:
  no unpushed/unique commits exist, `main` matches `origin/main`, and untracked
  directories (`backups/`, `scripts/`) are unaffected either way.

## V2 redesign — status: abandoned, will restart from a revised design
A Claude Design project ("Fixture list redesign proposal",
`claude.ai/design/p/a8f720d7-ca4c-423e-8543-2de14930487f`) was ported into
`index.html` on a `tickets-v2-design` branch. That branch has since been deleted
— it never diverged from `main` beyond uncommitted working-tree edits, so nothing
was lost. **Why abandoned:** Ian found things in the Claude Design itself he wants
to change before re-attempting the port — not a code-quality problem, a deliberate
reset to fix the source design first.

What the V2 port did (for reference, in case the revised design keeps the same
shape): red header band with derived "Upcoming"/"Seats claimed" stats, sticky
filter tabs with competition marks + live counts, fixtures grouped by month,
CSS-gradient kit swatches per club (no crests — trademark), and a seat-assignment
combobox (avatar, free-text input, roster dropdown, "Clear seat", and a
non-interactive locked variant for already-played fixtures).

**Open questions / gaps the previous attempt hit, worth resolving in the revised
design rather than re-guessing:**
- The design has no concept of a `postponed` fixture (only played vs. not, via
  score presence) — V2 treated postponed as "upcoming" styling with a "Postponed"
  tag in place of the kickoff time. Confirm this is still right.
- Locking seat-editing on played fixtures was a functional change beyond pure
  UI — Ian approved it explicitly last time; re-confirm rather than assume.
- The design's kit-colour/abbreviation table only covers 14 of the 24 real
  opponents, and 3 of the app's 4 competitions (no `FA Cup`). Extending it needs
  doing again for whatever the revised design covers.
- Mobile layout: a played fixture's "opponent name + score chip" row must not let
  the score's horizontal position depend on the opponent name's length — fixed
  last time by disabling wrap on that row (name truncates with an ellipsis
  instead) plus a `max-width: 640px`-only media query that makes the date/kit/name
  block fill its full line once the seat-pair has wrapped below it. Worth
  reapplying if the revised design has the same row structure.
- When verifying a layout fix, measure actual DOM geometry
  (`getBoundingClientRect`, computed styles) in an offline Firebase-stubbed
  preview rather than trusting a screenshot alone — a font-loading flash (FOUT)
  can look identical to a real box-model bug in a static image, and only DOM
  measurement tells them apart.

## Next phase
Waiting on Ian's revised Claude Design. When it's ready: re-run the safety
protocol above from scratch (git state check → propose a branch → wait for
approval → implement → verify against an offline/DB-free preview → report) rather
than assuming anything about scope or approach carries over from this attempt.
