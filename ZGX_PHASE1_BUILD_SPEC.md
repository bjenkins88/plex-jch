# ZGX Phase 1 Build Spec — Find It / File It

This pulls P1, P4, P6, P7, P8, and `_filing-rules.md` into one ordered,
buildable spec. It assumes whoever's building has SSH/DSM access this
session doesn't have.

## Readiness status

**Ready to build now:**
- The site plan/permission model (`_filing-rules.md`) — complete enough
  to start against, with known gaps flagged inline
- The scope boundary — `01 Job-Related` excluded from the general tool
  entirely; `02 Not-Job-Specific` categories all have real descriptions
  now, including the ones that were an open scope question in P2
  (Estimating & Purchasing, Schedule Management, Selections and
  Interiors, Designers, Construction Managers, Jenkins Standard
  Details, Vendor Catalogues, Renderings, Project Managers) — resolved
  by Bethany's dictation, not still pending
- The architecture (three tiers, confirm-then-execute, create-only
  write account) — settled since P1/P7

**One real blocker:** the BuilderTrend job-name ↔ server-folder mapping
doesn't exist yet (Component 1 below). Both "Find It" and "File It"
depend on it for reliable job matching — build this first, or in
parallel with Component 2, which doesn't depend on it.

**Deferred on purpose, not blocking:** BuilderTrend↔server document
sync (Component 5) — explicitly "can happen later" per Bethany.

---

## Component 1 — Job-name mapping (build first)

**Problem:** `jdb_costs.jobs` has 177 real BuilderTrend job names.
Nothing yet confirms those names match the actual folder names under
`01 Job-Related/05 Active Projects` / `10 Completed Projects` /
`20 Inactive Projects`.

**Status: a real first pass exists — `job_folder_xref.csv`.** Built by
exporting the real BuilderTrend job list (`Jobsites-4.xls`, 177 jobs,
read via `xlrd`) and matching it against 142 real server folder names
from `05 Active Projects` and `10 Completed Projects` (from screenshots;
`20 Inactive Projects` not captured yet). Matching logic: parse each
name into an alpha code + a year, match on the code, then check whether
the year agrees.

Results: **82 high-confidence exact matches** (code and year both
agree), **14 medium-confidence** (code matches, year differs by a
consistent-looking 1–6 years, or multiple BT sub-phase jobs share one
code), **15 low-confidence fuzzy matches** (no code match, closest
string match only — do not trust these without a look), **31 no-match**
(no reasonable BT counterpart found — expected, since the server
predates BuilderTrend and holds older completed jobs BuilderTrend never
tracked).

**Worth a specific look, not just "verify eventually":**
- The medium-confidence "year differs" group isn't noise — it's a
  consistent pattern (server year usually *earlier* than the matching
  BT year), which reads like the server folder was created at contract
  signing while BuilderTrend's year reflects a later re-entry or
  relabeling. Still worth a human glance, but this group is a good bet
  to be correct, not a coin flip.

**Four items confirmed by Bethany, resolved in `job_folder_xref.csv`:**
- `JDBO2023` — an internal code for an office remodel, not a client
  job. Correctly has **no** BuilderTrend record, and none should ever
  be expected — re-tagged `internal-no-bt` rather than left as a
  low-confidence guess.
- `SMTH 2017` — the completed job. Re-filed under `10 Completed
  Projects` as a clean high-confidence match; the copy that showed up
  in the `05 Active Projects` screenshot was a stale/misplaced folder,
  not a second real job.
- `PARADE 2008` — confirmed pre-BuilderTrend, the same "Parade Home 08"
  already referenced in the media-indexing work.
- **`20 Inactive Projects` doesn't need this exercise at all** — those
  jobs were never in BuilderTrend, full stop. Dropped from scope
  entirely rather than left as an open item to eventually capture.

**Still needed before this is complete:**
1. Someone needs to actually resolve the remaining medium/low-confidence
   rows and confirm the rest of the no-matches are genuinely
   pre-BuilderTrend rather than a naming mismatch the algorithm missed.
2. Once resolved, load `job_folder_xref.csv` into Postgres as the real
   `server_folder_path` column/table — this file is the draft, not the
   final schema.

---

## Component 2 — Path crawler + "Find It" (independent of Component 1)

1. New restricted DSM account (`aiuser`-pattern), read-only, ACL-scoped
   to exactly the confirmed folders in `_filing-rules.md` — explicitly
   excluding `07 HR`, `09 Accounting`, `Accounting-Backup-ONLY`, and all
   system/infrastructure folders listed there.
2. n8n job (nightly, same cron shape as Phase 8): SFTP walk → store
   `path`, `folder`, `filename` in a new table, e.g. `file_index`.
3. Semantic search over `file_index` via Ollama for "Find It" queries.
4. Response includes a clickable link: `smb://jdbsrv/<path>` by default
   (per P7 — reliable, no DSM API dependency). A File Station deep link
   is a later polish item, not a blocker.
5. **(Confirmed Sep 23)** If it's unclear which category/job a query
   means — especially whether it's job-specific at all — ask a
   clarifying question before answering, same fallback rule as File It.
   Once answered, briefly explain *why* using `_filing-rules.md`'s own
   language ("per policy, ..."), not just a bare link. If nothing
   matches, return the closest confident **parent folder** as a
   starting point instead of an empty refusal. See `_filing-rules.md`,
   "Find It — answer shape" section for the full spec.

---

## Component 3 — "File It": classify, match, confirm, write

1. Open WebUI Tool (Python), same shape as the Phase 7 Vanna tool.
2. **`_filing-rules.md` is loaded into the system prompt on every call
   — in full, not retrieved.** This is the one place in the whole
   project where that distinction (always-injected vs. searched) is
   load-bearing; get this wrong and every other rule in the file is
   decorative.
3. Ollama classifies: document type, and any job/project mentioned.
4. Match the mentioned job against `jobs` (using Component 1's mapping)
   — no confident match → ask, don't guess (per the fallback rule).
5. Propose a destination + filename
   (`<JobCode>_<DocType>_<YYYY-MM-DD>_<slug>.ext`), then ask: **"Would
   you like me to place it there?"**
5a. **(Confirmed Sep 23)** Before writing, check whether a file with
   that *exact* name already exists at the destination. If not, proceed
   as below. If it does, ask a second, separate question — *"A file
   named `<name>` already exists there. Replace it?"* — never overwrite
   silently.
6. On yes: a *separate* write-capable DSM account — create-only for new
   files, ACL-scoped to the same confirmed folders as Component 2 —
   performs the SFTP write. **(Confirmed Sep 23)** On a confirmed
   replace, the account first moves the existing file into a
   `_archive/` subfolder of the same destination (renamed
   `<name>__replaced-YYYYMMDD-HHMMSS.ext`), then writes the new file —
   never a true in-place overwrite yet. This is deliberately the
   starting point, not the end state: expected to move to a true
   overwrite later, once the tool has earned enough trust that the
   archive step feels redundant.
7. Log every placement (source file, matched job, destination,
   timestamp, and whether it was a fresh placement or a replacement) to
   a `placements_log` table. This is the undo mechanism — there is no
   other one, and it's what makes it safe for `_archive/` to eventually
   get cleaned up (see below) without losing the historical record.
8. **(Confirmed Sep 23) `_archive/` retention:** a scheduled job (same
   shape as the existing nightly/weekly crons) deletes anything inside
   any `_archive/` folder older than **90 days**. The write account
   needs delete rights for this, scoped *only* to `_archive/`
   subfolders via Synology's per-subfolder Advanced Permissions — the
   main content tree stays exactly as create-only as originally
   designed. See `_filing-rules.md`, "File It — replacing an existing
   file" for the full spec.

---

## Component 4 — Permission enforcement: two layers, not one

**DSM-level (the write account itself):** no filesystem permission to
`07 HR`, `09 Accounting`, or `Accounting-Backup-ONLY` at all — this is
enforced by the OS, not by asking the model nicely. Defense in depth:
even a broken prompt or a jailbreak attempt can't write somewhere the
account has no access to.

**(Confirmed Sep 23) The one place this account gets delete rights:**
`_archive/` subfolders only, via Synology's per-subfolder Advanced
Permissions, so the 90-day retention job (Component 3, step 8) can
actually clean up. Every other in-scope folder stays create/write
without delete — delete exists in exactly one narrowly-scoped place,
never on primary content.

**Tool-logic level (policy the account *could* technically do, but
shouldn't without asking):** the write account needs real write access
to `04 Marketing Shared` and `photo` to perform placements on behalf of
people who *are* authorized — Adaline, PJ Madiera, the admins. The tool
itself still has to enforce who's asking: a Designer filing something
marketing-bound gets routed to `Marketing Collateral`, never directly
into `Marketing Shared`, regardless of what the service account could
technically do. This is a policy check in the tool's own logic, not a
DSM ACL — get it wrong and the account's broader access becomes the
loophole.

---

## Component 5 — BuilderTrend sync (deferred, not v1)

Both directions specified in `_filing-rules.md` — server→BuilderTrend
ask, and the BuilderTrend→server pull extending the existing nightly
Bills/Jobs job (Phase 3). Explicitly out of scope for the first build;
noted here only so it isn't lost when this gets picked back up.

---

## Suggested build order

1. Component 1 (job mapping) and Component 2 (path crawler) — can
   happen in parallel, neither depends on the other
2. Component 3 (File It), which depends on Component 1
3. Component 4's DSM-level restrictions — set up *before* Component 3's
   write account goes live, not after
4. Run for a while against real files; correct `_filing-rules.md` based
   on real overrides, per the evergreen loop already specified in P8
5. Component 5, whenever it comes back off the deferred list
