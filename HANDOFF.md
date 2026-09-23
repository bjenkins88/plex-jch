# Handoff: ZGX Phase 1 build

**For:** whoever picks this up next — a local Claude Code session on
the office network (or over WireGuard), or Bethany directly.

**Why this exists:** everything so far was planned by a Claude session
running in an isolated cloud sandbox with no path to the office LAN —
no route to `192.168.1.35`, no SSH/SFTP credentials, by design. This
file is the cold-start entry point for a session that actually has
network access, so none of that planning has to be redone or
re-explained from scratch.

## The one-sentence goal

Build a tool where people at the office can ask Llama (via the ZGX
Nano's existing Ollama + Open WebUI) two questions — *"where could I
find this file?"* and *"where should this file go?"* — with the second
one actually performing the file placement, on confirmation.

## Read these, in this order

1. **`ZGX_PHASE1_BUILD_SPEC.md`** — start here. The actual ordered build
   plan: 5 components, what depends on what, current status of each.
2. **`_filing-rules.md`** — the rules Llama needs loaded into every
   call: folder-by-folder routing logic, access restrictions (hard
   boundaries — read this before writing any code that touches `07 HR`
   or `09 Accounting`), and the fallback "ask, don't guess" rule.
3. **`job_folder_xref.csv`** — the BuilderTrend-job-name ↔ server-folder
   mapping (Component 1 of the build spec). Real data, partially
   confirmed by Bethany, not yet loaded into Postgres.
4. **`ZGX_ASSISTANT_PROJECT.md`** — the full narrative history if you
   want the "why," including the retrieval proof-of-concept and the
   reasoning behind confirm-then-execute. Not required to start
   building, useful if something in the spec seems arbitrary.
5. **`buildertrend_jobs_raw.csv`** — the raw 177-row BuilderTrend export
   the xref was built from, in case anything needs re-deriving.

*(Sibling project, unrelated to this build: `INTRANET_IMPROVEMENT_PLAN.md`
— the team Google Site itself. Separate, active, no shared dependency.)*

## Status right now

- **`_filing-rules.md`**: usable first draft. Real open items are
  written into the file itself (search "Open items") — don't skip that
  section.
- **`job_folder_xref.csv`**: 142 server folders matched against 177 real
  BuilderTrend jobs. ~86 confirmed/high-confidence, the rest split
  between "needs a quick human look" and "confirmed no BT record." Not
  yet loaded anywhere — still a CSV, not a database table.
- **Nothing has been built yet.** No n8n workflow, no Open WebUI Tool,
  no new DSM accounts, no Postgres schema changes. This is a planning
  handoff, not a partially-built system.

## Start here, concretely

Per the build spec's suggested order:

1. **Component 1 + Component 2 in parallel** — neither depends on the
   other:
   - Component 1: load `job_folder_xref.csv` into Postgres
     (`jdb_costs.jobs`, new `server_folder_path` column or a separate
     mapping table), resolve the remaining medium/low-confidence rows
     with Bethany as needed.
   - Component 2: new restricted read-only DSM account, ACL-scoped per
     `_filing-rules.md`'s confirmed-in-scope list; nightly n8n crawl
     into a new `file_index` table; wire up semantic search + the
     `smb://jdbsrv/<path>` link format from the build spec.
2. **Component 4's DSM-level lockdown before Component 3 goes live** —
   set up the write account's restrictions first, not after.
3. **Component 3** (the actual "File It" Tool) once 1 is done.

## One thing worth doing before writing any code

Verify SSH/SFTP access to the ZGX (`192.168.1.35`) and `jdbsrv` actually
works from wherever this session is running, and confirm the existing
`aiuser1`-style restricted account pattern from Phase 8 is still the
right template to copy for the two new accounts this needs (one
read-only for Component 2, one write-restricted create-only for
Component 3).

## Keep the thread alive

Commit progress back to this same branch
(`claude/intranet-improvement-plan-64w1n2` in `bjenkins88/plex-jch`) as
things get built, the same way everything up to this point was tracked
— so whoever picks this up after *you* isn't starting cold either.
