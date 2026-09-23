# ZGX Ask-Layer — Phase 1 Scoped

**Project 2 of 2.** A local-AI "where do I find X" layer over the office
file server, built on the ZGX Nano's existing infrastructure.

**Status: Phase 1 built and working, Sept 23, 2026.** Paused Sept
14 because the server is too messy for full indexing; unblocked Sept 15
with the write-gate idea (P6); scoped down Sept 22 into something much
smaller and more achievable first — see P7. In your own words, on why
the original plan stalled: "I haven't figured out how to get the other
piece of this to work because we have so many folders on the server,
and they're such a mess that it would be very hard for AI to look at it
and figure out how to organize it for us." P7 sidesteps that blocker
rather than solving it head-on. **Built end-to-end the very next day —
see P9 for the full build log, real bugs found via live testing, and
what's still open.**

Rendered version with diagrams: https://claude.ai/code/artifact/d2da5afc-f715-4045-8d9b-c60cb6ad1df6

Sibling project (active): `INTRANET_IMPROVEMENT_PLAN.md` — the team
intranet site itself, unaffected by this pause.

## P0 — What this project is

Let employees ask "where do I find X" in plain language and get pointed
to the real file or system — instead of publishing and maintaining a
second copy of every procedure somewhere new. The ZGX Nano already runs
the infrastructure this needs (Ollama, Open WebUI, Postgres/pgvector),
proven end-to-end on the in-progress Phase 8 media-search pipeline (SFTP
pull from the office Synology → extract/embed → pgvector → surfaced in
Open WebUI). This project is that same pattern, pointed at documents
instead of photos.

It's a superset of the intranet site's job — the site (Project 1) only
ever needed to cover pages that live *on the site*; this was going to
reach into the wider file server too.

## P1 — Architecture already settled

- **Source** (systems of record, untouched): office Synology (docs,
  forms), Paylocity (HR self-service), BuilderTrend (jobs, field), plus
  a hand-written `_routing-index.md`.
- **Index** (nightly, via n8n): restricted SFTP pull → extract & embed
  text → upsert into a `company_docs` pgvector table — same pattern
  already running for Phase 8.
- **Ask** (Open WebUI): answers a specific question, walks a workflow
  step by step, or refuses rather than guesses when the retrieved text
  doesn't say.

Ingestion reuses the exact pattern already proven on Phase 8: a
restricted DSM account (same shape as the existing `aiuser1`), scoped by
folder ACL. Sensitive HR material (comp, disciplinary, medical, legal)
gets excluded at the file-permission level, not just prompted around.

## P2 — What's already true on the real server

The top level of `jdbsrv` is already split into `01 Job-Related` and
`02 Not-Job-Specific` — which almost matches the intranet's own
BuilderTrend-vs-company scope line. As a first cut, the restricted
ingestion account would simply never get access to `01 Job-Related` at
all.

**Confirmed in scope** (`02 Not-Job-Specific/`):
```
07 HR (Human Resources)
08 IT
03 Accounting Procedures
09 Insurance - GL & Risk
10 Client Forms & Books
11 Interoffice Forms & Forms
12 Management Reports
99 Best Practices
```

**Not yet decided:**
```
01.1 Estimating & Purchasing
01.2 Schedule Management
02 Selections and Interior
05 Designers
06 Construction Management
16 Renderings
17 Project Managers
15 Vendor Catalogues
04 Marketing Shared
14 Jenkins Standards
```

You chose "mixed — I'll go folder by folder" rather than an in/out
default for the second list. "Not tied to one job" isn't the same test
as "not construction process" — several of these read like design/build
workflow even though they sit outside `01 Job-Related`. This decision is
unresolved — see P5.

## P3 — What the proof-of-concept found

Couldn't reach the real Synology from that session (no network path, no
reason to hold credentials there). Rebuilt 9 real files from real site
content instead, and tested retrieval at three description depths
against 13 realistic questions with TF-IDF keyword search, as a
lightweight stand-in for the real pgvector embedding search.

| Depth indexed | Score |
|---|---|
| Folder-level library summary only | 10/13 |
| One-line-per-file description | 9/13 |
| Full workflow file content | 11/13 |

**Conclusion:** folder-level descriptions alone aren't enough — and the
10/13 flatters the approach, since it only had 3 folders to guess
between; a real deployment has far more. Full file content has to be
indexed, not just a label describing the folder.

**Two design fixes to carry forward:**
1. Keep each file scoped to one job — two files that legitimately share
   vocabulary (a backup workflow and a server-access reference both
   mentioning `jdbsrv`) can out-compete each other.
2. The assistant must refuse rather than guess when the retrieved text
   doesn't literally contain the answer — a real test question ("how
   many sick days do I get") scored a plausible-looking match against a
   file that didn't actually have the number.

## P4 — Everything else already decided

- **Content types:** tag files `WORKFLOW-` (return steps verbatim, never
  paraphrase) vs. reference/policy (answer and cite)
- **Retirement:** move a file to `_archive/`, don't delete — Synology's
  own Drive versioning is the safety net; scrub any pointer to it from
  the routing file in the same edit
- **Change triggers:** a system-of-record change (a vendor swap), a
  flagged bad answer, or scheduled review — same three as the intranet
  site's own governance
- **No manual "publish" step:** an owner edits a file on the Synology,
  the nightly n8n job picks it up automatically, it's live within a day
- **Front door:** a shared screen/kiosk at the office, a bookmark at
  each desk, and the existing WireGuard VPN (Phase 6) for off-site/
  jobsite access

## P6 — The long-term fix: automation as the only writer

**The core idea (Sept 15):** stop trying to clean up a moving target.
People dragging files onto the share by hand is what made it messy in
the first place, and will keep making it messy no matter how good the
index gets. The durable fix is upstream of indexing: **make an
automation the only way a file lands on the server**, so every file
already has a consistent name and a correct project folder the moment it
exists. Retrieval gets reliable because *ingestion* got reliable — not
because the AI got better at guessing.

**Is there a free AI tool that already does this?** Not quite, and it's
worth knowing why before reaching for one:

- **[LlamaFS](https://github.com/iyaja/llama-fs)** is the closest match
  in spirit — free, open-source, uses an LLM to rename and refile
  documents by content, and has an "incognito mode" that routes through
  Ollama instead of a cloud API, so it could run fully local against the
  ZGX's existing Ollama server. But it's a small, hackathon-grade
  project, not something with a track record at this scale, and its
  image/audio components (Moondream, Whisper) have unverified ARM64
  support on GB10-class hardware — the same "official containers just
  won't start on ARM64+CUDA" problem that's already bitten other tools
  on this exact chip family. More importantly: it invents its own
  organizing scheme from a batch of files. It has no way to know your
  177 real BuilderTrend job names and will not reliably reuse the exact
  same project name twice on its own.
- **[Paperless-ngx](https://www.layer3labs.io/guides/open-source-ai-document-management-software),
  Docspell, Mayan EDMS, Papermerge** are mature, genuinely free,
  well-supported self-hosted document archives with OCR and
  auto-tagging. They're built for a *scanned-paperwork inbox* (invoices,
  letters, insurance forms) with their own tag/correspondent database —
  a strong fit for the Accounting / Insurance / Client Forms slice of
  `02 Not-Job-Specific` specifically, but not a general gatekeeper for
  the whole project-file server or its job-name conventions.

**The actual recommendation: extend what's already running, don't bolt
on a new product.** The hard part was never "can an LLM read a
document" — it's "will it always produce the exact same project name."
You already have the answer to that sitting in Postgres: the `jobs`
table in `jdb_costs` holds 177 real, canonical project names pulled
straight from BuilderTrend (Phase 3/7). A generic file organizer has to
guess a name from the document's content; your pipeline can look the
real name up and refuse to invent one when it isn't sure.

Concretely, this is Phase 8's read pipeline, extended to also write:

1. **One locked inbox** (a single upload folder, an email-to-folder
   address, or a simple form) becomes the only writable path onto the
   server. Normal write access to the real project/department folders
   gets turned off for people.
2. **n8n watches the inbox** and sends each new file to Ollama for
   classification: document type, and any project or job it mentions.
3. **The mentioned project gets matched against the real `jobs` table**
   (fuzzy/trigram match), not trusted from the model's own phrasing. No
   confident match → routed to a human review queue instead of a guess.
4. **A fixed naming template** (e.g.
   `<JobCode>_<DocType>_<YYYY-MM-DD>_<slug>.pdf`) gets applied the same
   way every time — a template, not the model's mood that day.
5. **The file is written via SFTP** to the correct destination, and
   every decision is logged so a human can undo one in seconds.

**The honest catch — this is a process change, not just a script.** It
only works if people actually stop dragging files straight onto the
share, which means locking down write permissions and getting real
buy-in, not just standing up an n8n workflow. That adoption piece is
probably the harder of the two problems.

**It doesn't retroactively fix the existing mess**, either. What it
does give you is a way to freeze the bleeding immediately — every new
file from today forward is named and filed correctly — while the
`01 Job-Related` backlog gets cleaned up separately, on a slower
timeline, possibly semi-automated later using the newly-consistent
files as good examples of what "correct" looks like.

## P7 — Phase 1, scoped down: Find It / File It

**The idea (Sept 22):** instead of the full content-indexing plan (P1–P4)
or the enforced write-gate (P6), ship something much smaller first — one
tool, two questions, both answered by Llama through Ollama, right there
in Open WebUI. No server lockdown. No full-server cleanup required
first. This becomes the real Phase 1; P6 becomes an optional Phase 2 for
later, once this one has earned trust.

**On "teaching a llama" — what that actually means here:** not
fine-tuning. Fine-tuning is already Phase 10 on your own roadmap
(~18 months out) and needs a pile of labeled examples that don't exist
yet. What this needs instead is **prompting + lookup** — giving the
model the right context at question time (the real job list, the folder
categories, a few examples), not training it on anything. Faster to
build, costs nothing extra, and more reliable for a structured task like
this than fine-tuning would be at this stage.

**Question 1 — "Where could I find this file?"**
A natural-language search over a lightweight index of *file paths and
folder names* — not full document content. This is a directory crawl,
not an OCR/content pipeline, which is why it's achievable without first
solving the P2/P5 mess: it indexes what's *called*, not what's *inside*.
Llama does semantic matching over that path/filename text (plus folder
READMEs, where they exist), and **the answer includes a clickable link
straight to the file's folder**, not just a typed-out path — nobody
should have to hand-copy a path into File Station. Two ways to build the
link, in order of how much they're worth verifying before committing to
one:
- **An `smb://jdbsrv/<path>` link** — opens directly in Finder or
  Explorer on any machine with the share mounted. No DSM API involved,
  works the same way the "Jenkins Server" link already does today, and
  is the lower-effort, more reliable default for Phase 1.
- **A Synology File Station deep link** — nicer (opens in the browser,
  works for someone without the share mounted), but the exact URL
  scheme needs to be confirmed against your specific DSM version before
  relying on it — worth a quick spike, not an assumption.

**Question 2 — "Where should this file go?"**
A person describes the file (or uploads it); Llama classifies the
document type and pulls out any project/job it mentions; that name gets
matched against the real `jobs` table in `jdb_costs` (177 real
BuilderTrend names) instead of invented; the tool proposes a destination
folder and a consistently-formatted filename and **asks: "Would you like
me to place it there?"** On yes, the tool performs the write itself over
SFTP and confirms it's done — no manual drag-and-drop, no chance of a
typo'd folder name. This is confirm-then-execute, not advisory-only: it
mirrors the human-approval-before-any-write pattern you already built
and trust for the BuilderTrend push-back integration (Phase 3) — same
shape, applied to file placement instead of BuilderTrend data.

**What confirm-then-execute changes about the write account:** it now
needs real write access, which is a bigger deal than the read-only
`aiuser1` pattern used everywhere else in this project. Scope it tightly:
- **Create-only** — never grants delete or overwrite, so a bad match
  can't destroy or silently replace an existing file
- **Limited to the confirmed-in-scope destination folders** from P2,
  the same boundary already used for reads
- **Every placement logged** (source file, matched job, destination,
  timestamp) so a wrong match is a one-click undo, not a cleanup project

**Reuses what's already running:** same shape as the Phase 7 Vanna
cost-query tool — a small API Llama calls, surfaced as an Open WebUI
Tool — plus one new lightweight piece: a path crawler (list every file
path in the confirmed-in-scope folders from P2, store path + folder text
in a small table). Substantially smaller than the original P1–P4
content-pipeline build.

**Two honest expectations, set now:**
- "Find it" will work noticeably better in the confirmed-clean folders
  (07 HR, 08 IT, 99 Best Practices) than in the messy `01 Job-Related`
  archive — a path-only index can search bad naming, it can't fix it.
- "File it" is the stronger half out of the gate, precisely because it's
  anchored to a real, clean list of job names rather than needing the
  messy folders to already make sense.

**The shape this gives the whole project:** because Phase 1 already
executes writes (on a per-file human "yes"), it's closer to P6's goal
than originally scoped — what P6 adds on top, as an optional later
phase, isn't automated writing itself (Phase 1 already does that), it's
**removing the manual alternative**: locking down direct write access so
the tool becomes the *only* path, once its suggestions have earned
enough trust that the per-file confirmation starts to feel redundant.
Phase 3, also optional, layers real content-level indexing (the original
P1–P4 plan) onto whichever folders turn out to be worth the investment.

## P8 — Documenting the filing logic, so Llama can follow it

**Format matters more than content here.** An LLM follows explicit,
structured rules far more reliably than prose describing a filing
philosophy — same lesson as A2 in the sibling intranet project (policy
vs. workflow content reads differently), applied to rules instead of
procedures.

**One file: `_filing-rules.md`.** Same naming convention as
`_routing-index.md` and `_start-here.md`, but different in one important
way: it gets **injected directly into every "File It" prompt**, not
retrieved by search. Everything else in this project (WORKFLOW-,
REFERENCE- files) is found by semantic search when relevant — this
document is foundational instruction, not something to search *among*.
If retrieval happens to miss it on one call, Llama files something with
no rules at all. At a realistic size (15–20 categories), it comfortably
fits in every call's context, so there's no reason to risk that.

**One entry per category, same shape every time:**
```
### 07 HR (Human Resources)

What belongs here: personnel records, benefits paperwork, policy
documents that apply company-wide, not to one job.

Examples:
- Employee handbook, W-4s, benefits enrollment forms
- Company-wide holiday/PTO policy updates

Not this — see instead: a certificate of insurance naming a specific
employee goes in Client Forms & Books if it's for a client-facing job,
not here.
```

**Four things worth doing deliberately:**
1. **A fixed, enumerated category list — never free text.** Same
   principle as the `jobs` table lookup: the category names and real
   folder paths must be the exact same list every call, or "consistent
   filing" stops meaning anything.
2. **Write the disambiguation notes for pairs you already know are
   confusable, not just the clean cases.** The highest-leverage section.
   P3's proof-of-concept found real near-miss failures between related
   content sharing vocabulary (a backup workflow vs. a server-access
   reference) — the same thing will happen between real folders like
   Insurance vs. Client Forms, or HR vs. Interoffice Forms. Every
   category needs at least a placeholder "not this — see instead" line.
3. **State the fallback explicitly: no confident match means ask, don't
   guess.** Mirrors P7's confirm-then-execute flow, but needs to live in
   the rules doc as a written rule too, not an assumption.
4. **Keep a running table of real examples as a second section.** Every
   real filename-to-destination pair is both a few-shot example (models
   pattern-match from concrete cases far better than from abstract rules
   alone) and a regression test you can rerun after any edit to the
   rules, to catch a change that broke something that used to work.

**How it stays accurate:** log every time someone overrides Llama's
suggested destination — that override is a free, real signal a rule is
missing or ambiguous. Review overrides on the same cadence as the
monthly query-gap review already used elsewhere in this project, and
turn recurring ones into a new disambiguation note or example row —
same evergreen discipline as the rest of the project, pointed at filing
decisions instead of workflow content.

**What's still yours to fill in:** the actual "what belongs here" text
per category needs real business judgment, not a guess from outside the
company — this document only specifies the template and the discipline
around it.

**Update Sept 23 — a real first draft now exists:** `_filing-rules.md`
is written, built from Bethany's actual dictated description of the
`jdbsrv` folder structure plus real screenshots (`01 Job-Related`,
`02 Not-Job-Specific`, and the canonical top-level `04 Marketing
Shared`). It covers the access-restricted folders, the folders that are
never a filing destination, and the two-part routing job-related files
need (job name + status bucket).

**Update, same day, second pass:** Bethany corrected and extended it
with real permission detail:
- Full HR access breakdown by person (Garrett/Clyde/Skyler North:
  read+write, no delete; Sarah: read+write+delete; Bethany/Shan/Truitt:
  full).
- A new read-open/write-restricted category: `photo`/`photo_1` (write:
  Adaline + admins only) and `04 Marketing Shared` (write: PJ Madiera,
  Adaline, admins only).
- **A correction, not just an addition:** `01 Job-Related/Marketing
  Collateral` was flagged as a misplacement in the first draft — wrong.
  It's a deliberate workflow: Designers don't have write access to
  `04 Marketing Shared`, so this is their real drop-off point.
- The duplicate nested `04 Marketing Shared` has been deleted from the
  server — resolved, not just documented as an error.
- **Two new open items surfaced, not yet built:** (1) there's no
  mapping yet between BuilderTrend's job names and the server's actual
  job folder names — needed for reliable filing *and* for
  cross-referencing file-server content against BuilderTrend/cost data
  more broadly; (2) a proposed (not previously recorded, so explicitly
  unconfirmed) rule for when the tool should also suggest a BuilderTrend
  upload, separate from the server destination — grounded in the
  existing human-approved BuilderTrend push-back pattern (Phase 3)
  rather than a new risk pattern.

**Update, third pass:** Bethany confirmed `video`/`Video to Share` are
unused (permanently excluded now) and confirmed the proposed
BuilderTrend-upload rule — then extended it into a two-way rule that
changes the shape of P7/P8 slightly:

- **Server → BuilderTrend** (as proposed): a job-related document in a
  BuilderTrend-tracked category should prompt the tool to ask before
  also pushing it to BuilderTrend.
- **BuilderTrend → server** (new): the reverse holds too, and matters
  more — "if something goes on BuilderTrend, it probably also needs to
  be on the server, especially plans or plan documents... we want to
  own our own data." Same motivation as running the ZGX locally in the
  first place: BuilderTrend shouldn't become the only copy of anything
  that matters.
- **A real gap, named rather than glossed over:** a chat-time "ask when
  filing" rule only fires when a person brings a file through File It.
  It can't see a plan uploaded straight into BuilderTrend by someone who
  never touched the local tool — which is exactly the case the
  "own our own data" principle most needs covered. Closing that
  requires extending the *existing* nightly BuilderTrend pull (Phase 3,
  Bills/Jobs data today) to also catch new plans/attachments and sync
  or flag them — a pull mechanism, not something a prompt alone can
  guarantee.

## P5 — Resuming this: start here

1. **Build the P7 Phase 1 tool.** Doesn't require the existing mess
   cleaned up, doesn't require a permissions change, and starts
   producing consistently-named new files immediately.
2. **Make the folder-by-folder call** from P2's "not yet decided" list —
   in scope for the tool, or treated like BuilderTrend SOPs (out of
   scope) — needed either way, since P7's "find it" side has to know
   what it's allowed to search.
3. **Write `_filing-rules.md`** (P8) for the confirmed-in-scope
   categories, including disambiguation notes for the pairs most likely
   to be confused.
4. **Once P7 has run for a while, decide whether to tighten it into
   P6's enforced write-gate**, using real usage as the evidence rather
   than guessing upfront whether people will actually follow a
   suggestion.
5. **Content-level indexing (the original P1–P4 plan) is now optional,
   not required** — only worth doing for folders where path/filename
   search (P7) genuinely isn't cutting it.
6. **The `01 Job-Related` backlog stays a separate, later cleanup**, on
   its own timeline, not a blocker to any of the above.

## P9 — Sept 23, 2026: Phase 1 built end-to-end, live-tested, deployed

Built on-site, physically on the office LAN. Both Find It and File It are
live, tested against the real server, and running as persistent systemd
services on the ZGX. This section is the build log — what shipped, the
real bugs live testing found (per Component 2 vs File It vs the tool
integration, matching the pattern of every other phase in this project:
report a wrong answer, dig for the real cause, fix it, verify against
the real service), and what's still open.

### Component 1 — job mapping: loaded

`job_folder_xref.csv` (142 rows) loaded into a new `server_folder_map`
table in `jdb_costs` (Postgres). The 86 high-confidence single-job
matches were also mirrored into the existing `project_aliases` table
(`alias_type='server_folder'`), so Cost Data/Media Search cross-referencing
can use them immediately. Medium/low-confidence rows are loaded but not
auto-aliased — still need a human look before anything trusts them.

### Component 2 — Find It: built, tested, running

- New DSM account `aiuser2` (read-only, SFTP-only via the Applications
  tab, scoped to the confirmed-in-scope folders, explicitly excluding
  `07 HR` and `09 Accounting`) — see "Real bugs" below for what it took
  to actually get this account working.
- `path_crawler.py`: SFTP walk of the 5 confirmed-in-scope roots
  (`01 Job-Related`, `02 Not-Job-Specific` minus HR, `04 Marketing
  Shared`, `photo`, `photo_1`) into a new `file_index` table.
  **347,709 files indexed.** Scheduled nightly at 2am via a new n8n
  workflow (`JDB Find It - Nightly Path Crawler`, SSH node, same
  import-a-JSON-file pattern as the existing weekly report workflow).
- `find_it_app.py`: a Flask API (port 8086, `find-it.service`) that
  loads `_filing-rules.md` in full on every call, classifies
  job-specific vs. department vs. ambiguous (asks rather than guesses
  when unclear — a hard requirement, not just a nice-to-have), does a
  `pg_trgm` similarity search over `file_index`, and has Ollama pick the
  best match or the closest parent folder as a fallback — always citing
  the policy reasoning, never a bare link.
- Hard-coded (not left to the model) restricted-folder guard: even
  though `file_index` never contains HR/Accounting rows at all, the
  fallback step reasons over folder *names* in the rules text and could
  still name one — caught and blocked in code.
- A "known redirects" mechanism for things that aren't on the file
  server at all (e.g. the employee handbook actually lives on the team
  intranet site) — implemented as a deterministic keyword match parsed
  straight out of `_filing-rules.md`, not an LLM judgment call, after
  the LLM version missed the exact query it was built for.
- Wired into Open WebUI as a Tool, then merged into a single combined
  Tool alongside File It per Bethany's request (a user might want to
  both look something up and file something in the same conversation).

### Component 3 — File It: built, tested, running

- New DSM account `aiuser3` (read/write on the confirmed-in-scope
  folders, explicitly no access to `07 HR`/`09 Accounting`/
  `Accounting-Backup-ONLY` — Component 4's DSM-level hard boundary).
- `file_it_app.py`: a Flask API (port 8087, `file-it.service`) with two
  endpoints matching the propose-then-confirm design:
  - `GET /propose` — classify, match the job (via `server_folder_map`,
    never inventing one), check for HR/Accounting-flavored content and
    refuse with named contacts rather than silently misfiling it, find
    the best real *existing* subfolder via `file_index` (not just the
    bare category root), and return a destination + filename + a
    confirm question.
  - `POST /place` — the actual SFTP write, over `aiuser3`. Checks for an
    existing file with the same name; on a confirmed replace, archives
    the old one into `_archive/` (renamed with a timestamp) rather than
    overwriting, then writes the new file. Every placement logged to a
    new `placements_log` table (source, matched job, destination,
    timestamp, whether it was a replacement).
- `_archive/` retention: a 90-day auto-purge is the agreed design
  (delete-scoped-to-`_archive/`-only permission, `placements_log` keeps
  the permanent record after the bytes are gone) — **documented in
  `_filing-rules.md`, not yet built as a running job.**
- Filenames use the real original filename (slugified) plus the
  resolved job code and date — deliberately *not* an LLM-invented
  document-type label or a slug of the free-text description, after
  live testing showed both were unreliable (see below).

### Real bugs live testing found (same discipline as every other phase — dig, don't reassure)

1. **DSM SFTP subsystem refused both new accounts with `EOF during
   negotiation`** even though SSH auth succeeded and every permission
   setting (Applications tab, folder ACLs, group permissions) looked
   identical to the working `aiuser1`. Root cause, found only after
   checking `/etc/passwd` and the home directories directly: DSM's
   forced-password-change-at-first-login flag blocks the SFTP subsystem
   at the DSM layer even though OS-level password auth doesn't care
   about it. Fixed by actually logging in once through DSM's own UI.
2. **Open WebUI's built-in tools kept winning over the custom Tool.**
   With "File Context" (native RAG-over-attachments) on, attached files
   never reached the Tool at all. With it off, the model reached for
   built-in `ask_user`, then `list_memory_paths`, then
   `search_knowledge_files` (the last one hallucinated as raw text, not
   even a real call) before ever calling `file_document`. Root cause,
   found only after all of that: **the custom tool was never enabled
   for the new "JDB Assistant" model preset at all** — a brand-new
   model starts with its own empty Tools selection, it doesn't inherit
   from whatever model you'd enabled it on before. Once enabled, it
   worked immediately. Disabling the unrelated built-in tool categories
   (Ask User, Memory, Notes, Knowledge Base, etc.) for this model was
   still worth doing afterward, so nothing else competes for the same
   intent going forward.
3. **The model invented a fake job name ("Become Legendary") and later
   a fake document-type label ("handbook" for an interior design
   document)** when constructing the tool's own `description` argument
   or when Ollama classified `doc_type` server-side. Neither was
   grounded in anything the user actually said. Fixed two ways: the
   tool's docstring now explicitly forbids inventing job/project names
   or details not stated, and the actual filename template was changed
   to use the real, human-chosen original filename instead of an
   LLM-generated label — removing the specific piece that kept getting
   hallucinated, rather than just asking the model more firmly not to.
4. **A file got placed on the server without the user ever seeing or
   answering "would you like me to place it there?"** — after the user
   answered an unrelated clarifying question ("department wide"), the
   calling model set `confirmed=True` on its own initiative and the
   write executed. A boolean the model sets by its own judgment isn't a
   strong enough gate for something that actually writes to disk. Fixed
   by replacing the boolean with a `user_latest_message` string
   parameter and a **code-level** affirmative-language check (not an
   LLM judgment call) — the write only happens if the user's own literal
   words contain real affirmative language. Also collapsed what was a
   two-round confirm-then-confirm-replace flow into one: `/propose` now
   checks for a same-name collision live and discloses it up front in
   the single confirm question, so one verified "yes" covers the whole
   operation.
5. **A real document (`Interior Analysis.docx`, Interior Design
   department content) got proposed for `11 Interoffice Forms &
   Procedures` instead of the correct `02 Selections and Interiors`,**
   which already has a real `Interiors/INTERIOR ANALYSIS` subfolder for
   exactly this. Two compounding causes: the model invented a
   plausible-sounding "Travis County"-style subfolder on its own
   initiative in a *different* case rather than only returning the bare
   category (fixed by tightening the prompt to forbid it - subfolder
   selection is a code-level job now, via `file_index`), and separately,
   real folder names on the server sometimes have irregular double/
   triple spaces that don't match `_filing-rules.md`'s clean documented
   names, which silently broke the subfolder-matching SQL's prefix match
   (fixed by normalizing whitespace on both sides of the comparison).
   `_filing-rules.md`'s Selections and Interiors entry was also
   strengthened with this real example and a disambiguation note.
6. **A firewall gap, twice** — Docker containers (n8n, Open WebUI)
   reaching the host's own LAN IP for SSH (port 22) and the two new
   Flask services (8086, 8087) got silently dropped by `ufw`, same
   underlying pattern as the already-documented Ollama/port-11434
   regression from August: a "LAN only" rule doesn't automatically cover
   Docker's bridge subnet (`172.17.0.0/16`) as a source. Fixed by adding
   explicit `ufw allow from 172.17.0.0/16` rules for each port as the
   need came up.

### Also done this session

- **User Guide updated** (`JDB_AI_Assistant_User_Guide.md`/`.docx`,
  local copies now kept in `ai-for-jdb/` alongside the onboarding/admin
  guides) — added a "why we built this" opening, a Media Search section
  (was still missing despite being an open item since Sep 15), and a
  Find It/File It section, plus an updated decision-guide table.
  **Still needs to be pasted into the actual Google Doc** — the
  available Drive connector is tied to a personal Gmail account, not
  `bjenkins@newhousebuilder.com`, so it can read the shared doc but
  can't write to it.
- **Security note, unrelated to this build, flagged for later:** the
  DSM's Log Center showed continuous SSH brute-force attempts (generic
  bot usernames like `root`/`pi`/`ubuntu`, multiple per minute) against
  whatever port is internet-exposed for DSM SSH — been happening about
  a week per Bethany. Nothing indicates a successful breach, but worth
  addressing (fail2ban / IP allowlisting / key-only auth) once Phase 1
  work settles.

### Open items, going into next session

1. Build the `_archive/` 90-day retention job (design is written,
   nightly job itself isn't running yet).
2. Resolve the remaining medium/low-confidence rows in
   `server_folder_map` (Component 1) — still not auto-aliased.
3. Paste the updated User Guide content into the real Google Doc.
4. Medium/low-confidence job-folder rows aside, everything else in P7's
   original scope (subfolder matching, restricted-content handling,
   confirm-then-execute, replace/archive) is done and live.
5. `20 Inactive Projects`, drone-footage summaries, BuilderTrend sync
   (Component 5) — all still explicitly deferred, not forgotten, same
   as before.
