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

**Build:**
1. Add a `server_folder_path` column to `jobs` (or a separate
   `job_folder_map` table if a job ever has more than one folder).
2. Walk the real folders under the three status directories via SFTP.
3. Fuzzy-match each folder name against `jobs.job_name` (Postgres
   `pg_trgm` similarity is the natural fit, already Postgres-native).
4. Anything below a confidence threshold → a review list for Bethany,
   not an auto-match. This is a one-time reconciliation, not something
   to get cute about automating fully on the first pass.

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
6. On yes: a *separate* write-capable DSM account — create-only, no
   delete/overwrite, ACL-scoped to the same confirmed folders as
   Component 2 — performs the SFTP write.
7. Log every placement (source file, matched job, destination,
   timestamp) to a `placements_log` table. This is the undo mechanism —
   there is no other one.

---

## Component 4 — Permission enforcement: two layers, not one

**DSM-level (the write account itself):** no filesystem permission to
`07 HR`, `09 Accounting`, or `Accounting-Backup-ONLY` at all — this is
enforced by the OS, not by asking the model nicely. Defense in depth:
even a broken prompt or a jailbreak attempt can't write somewhere the
account has no access to.

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
