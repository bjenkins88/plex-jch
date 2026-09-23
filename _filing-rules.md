# Filing Rules — JDB SRV

This file is loaded into every "Find It" and "File It" call. It is not
searched — it is always present, in full, so a routing decision never
depends on retrieval happening to surface the right rule.

Two shared folders exist at the top level of `jdbsrv`, and the first
decision on every file is which one it belongs to:

- **`01 Job-Related`** — anything tied to a specific job. A job is a
  project that's been designed, is being built, has been built, or a
  prospect currently being talked to. Everyone has access.
- **`02 Not-Job-Specific`** — everything else: department-level
  reference material, procedures, and general company information not
  tied to one job.

If a document names a specific address, lot, neighborhood, or client
whose home is being designed/built/discussed — it's job-related, full
stop, regardless of which department produced it.

---

## Known redirects — not on the file server at all (confirmed Sep 23)

A small number of things people ask "where do I find X" about live
somewhere other than `jdbsrv` entirely. Find It must answer with the real
location below, not search or guess a server folder for these.

| Asked about | Real location |
|---|---|
| Employee handbook, company policies | The team intranet site's Company Policies page: `https://sites.google.com/newhousebuilder.com/team/company-policies?authuser=1` — not a file on jdbsrv. |

---

## Access control — read this before anything else

Several folders have real access restrictions. **The general Find It /
File It tool must never search, suggest, or write into a
read-restricted folder, and must never write to a write-restricted one**
unless the person asking has been specifically confirmed to have that
level of access. This isn't a filing preference, it's a hard boundary —
same principle already used for sensitive content elsewhere in this
project.

**Read + write restricted (not searchable or writable by anyone else):**

| Folder | Access |
|---|---|
| `02 Not-Job-Specific/07 HR (Human Resources)` | Garrett, Clyde, Skyler North — read/write, **no delete**. Sarah — read/write/delete. Bethany, Shan, Truitt — full (administrators). No one else. |
| `09 Accounting` *(top-level share, not the "03 Accounting Procedures" folder below)* | Peggy, Shan, Bethany, Truitt only |
| `Accounting-Backup-ONLY` | Same access as `09 Accounting` |

**Read-open, write-restricted** (the tool can search/suggest these for
anyone, but must never perform a write unless the uploader is on the
list):

| Folder | Who can write |
|---|---|
| `photo` / `photo_1` | Adaline, plus administrators (Shan, Bethany, Truitt) only. Everyone can read/browse. |
| `04 Marketing Shared` *(top-level, canonical)* | PJ Madiera, Adaline, plus administrators (Shan, Bethany, Truitt) only — **not** the Designers (see `Marketing Collateral` below for how their contributions actually flow in). |

A delete-capable account is a bigger deal than a write-only one — if the
"File It" tool's own service account ever needs delete rights on any of
these (it shouldn't, per P7's create-only design), that's a decision
for Bethany/Shan/Truitt specifically, not a default.

---

## Folders that are never a filing destination

These exist on the server but aren't part of the human filing taxonomy.
The tool should never suggest them, never search them, and never treat
an empty search result here as meaningful.

**System/hidden (every share has these):** `.AppleDB`, `.AppleDesktop`,
`.AppleDouble`, `.recycle`, `.webaxs`, `#recycle`, `#snapshot`,
`.TemporaryItems`

**Infrastructure shares, not document storage:** `ActiveBackupforBusiness`,
`chat`, `docker`, `Guest`, `home`, `homes`, `music`, `NetBackup`, `Plex`,
`PlexMediaServer`, `surveillance`, `usbshare1`, `usbshare2`, `usbshare3`

**Confirmed unused:** `video`, `Video to Share` — Bethany confirmed
neither is in active use. Excluded the same as the folders above; not
candidates worth revisiting.

---

## `01 Job-Related` — routing is two-part, not one

A job-related file needs **two** decisions, not one: *which job*, and
*which status bucket that job currently sits in*. The status bucket can
change over the life of a job (Active → Completed), so a document filed
correctly six months ago can be in the technically-wrong bucket today —
that's a known, accepted drift, not a bug to chase.

1. Match the job name against the real `jobs` table (as already
   specified in P1/P7) — never invent one.
2. File under whichever status bucket that job is currently in:
   - `05 Active Projects` — the job is in progress
   - `10 Completed Projects` — the job is finished
   - `20 Inactive Projects` — started, then never completed
   - (In practice, jobs sometimes sit in the wrong bucket longer than
     they should — match the job, not the folder it happens to be in
     today, if the two disagree.)

**Open build item — the job-name mapping doesn't exist yet.** Step 1
above assumes a BuilderTrend job name reliably identifies a folder on
the server, but nothing has confirmed the two actually use the same
names. The real `jobs` table (`jdb_costs.jobs`, 177 BuilderTrend job
names) and the server's actual folder names under `05 Active
Projects` / `10 Completed Projects` / `20 Inactive Projects` need an
explicit cross-reference — a `server_folder_name` column added to
`jobs`, or a separate mapping table — built once, by walking the real
folders and matching each to its BuilderTrend record. Bethany flagged
this as important beyond filing, too: it's what makes cross-referencing
file-server content against BuilderTrend/cost data possible at all.
Until this mapping exists, treat any job-name match as **unconfirmed**
and route to human review rather than assume the strings line up.

**Non-job-bucketed folders under `01 Job-Related`:**

| Folder | What belongs here |
|---|---|
| `00 DISPLAY TVS` | Files queued to display on the office TVs for marketing |
| `00 Prospects` | Prospective clients being talked to, not yet a signed job |
| `01 HOA DOCUMENTS` *(per job)* | That project's homeowners' association documents |
| `03 COUNTY DOCUMENTS` *(per job)* | That project's county documents |
| `50 Concept Renderings-Digital Plan Book` | Concept renderings of plans currently in design |
| `A&M Career Fair` | Recruiting materials for the Texas A&M career fair |
| `Rendering Videos` | Videos of renderings from the design department |
| `ZY - Templates & Forms`, `ZZ -`/`ZZZ - Job Related Project Template(s)` | BuilderTrend-related templates — not general company forms |

**Legacy / do-not-file-here-going-forward** (real content, wrong home):

- `04 Photos and Video of Projects` — **superseded.** Professional
  project photos now belong in the top-level `photo` share (Synology
  Photos requires them there to work correctly). Only route here if
  told explicitly; default new photos to `photo` instead.
- `60 Buildtools Archive Data` — old data from a pre-BuilderTrend tool
  ("Buildtools"). Archive only — never a destination for a new file.
- `Temporary Items`, `Website` — legacy; `Website` in particular is
  flagged for archiving. Not filing destinations.

**Not a misplacement — a real, deliberate workflow:**

- `Marketing Collateral` — this exists *because* the Designers don't
  have write access to `04 Marketing Shared` (that folder is
  write-restricted to PJ Madiera, Adaline, and administrators — see
  Access control above). This is the Designers' legitimate drop-off
  point for anything marketing-bound. **Routing rule:** if a Designer is
  filing something for the marketing department, it goes here, not to
  `04 Marketing Shared` directly (they can't write there anyway). If
  PJ Madiera, Adaline, or an administrator is filing it, it goes
  straight to `04 Marketing Shared`.

---

## `02 Not-Job-Specific` — one entry per real category

### 01 CITY Permits HOA submittals BUILDING codes
**What belongs here:** general permitting/HOA/building-code reference
material — not tied to one job. Used to understand a jurisdiction's
requirements, not to store one project's actual permit.
**Not this — see instead:** a specific project's own HOA or county
paperwork goes under that job's `01 HOA DOCUMENTS` / `03 COUNTY
DOCUMENTS` in `01 Job-Related`, not here.

### 01.1 Estimating & Purchasing
**What belongs here:** the Estimating & Purchasing department's own
working documents.

### 01.2 SCHEDULE MANAGEMENT
**What belongs here:** the scheduling department's own documents.

### 02 Selections and Interiors
**What belongs here:** interior designers' and the selections team's
department-level material — including interior analyses, selections
records, and similar Interior Design department documents. Real existing
subfolder structure includes `Interiors/INTERIOR ANALYSIS` — prefer
placing a matching document there over the bare category root when a
fitting subfolder already exists (confirmed Sep 23, real example:
`Interior Analysis.docx`).
**Not this — see instead:** a company-wide administrative form or
procedure (e.g. a generic interoffice request form) goes in
`11 Interoffice Forms & Procedures` instead — that folder is for
company-wide process paperwork, not one department's own working
documents.

### 03 Accounting Procedures
**What belongs here:** how the accounting department does things —
procedures and process documents.
**Not this — see instead:** actual financial data/records go in the
restricted `09 Accounting` top-level share, not here — this folder is
about process, not the numbers themselves.

### 04 Marketing Shared *(nested copy — deleted)*
**Resolved.** This duplicate has been removed from the server. The real,
canonical Marketing Shared is the top-level `04 Marketing Shared` share
(see its own section below). If a folder with this name ever reappears
here, that's an error, not a valid destination.

### 05 Designers
**What belongs here:** architects/designers/interior designers'
department-level information — their own resources, not tied to a
specific job site.
**Not this — see instead:** a specific job's actual design files live
under that job in `01 Job-Related`.

### 06 Construction Managers
**What belongs here:** the construction management department's own
general information.

### 07 HR (Human Resources)
**Restricted — see Access control above.** Not a general filing
destination.

### 08 IT
**What belongs here:** IT department information.

### 09 Insurance - GL & Risk
**What belongs here:** general liability and risk management
department information.
**Not this — see instead:** a certificate of insurance tied to a
specific client or job goes in `10 Client Forms & Books`, not here.

### 10 Client Forms & Books
**What belongs here:** client-facing forms and books/paperwork
templates.

### 11 Interoffice Forms & Procedures
**What belongs here:** internal interoffice forms and procedures.

### 12 Management Reports
**What belongs here:** management-level reporting.

### 14 JENKINS STANDARD DETAILS
**What belongs here:** standard construction detail drawings/specs used
company-wide.

### 15 Vendor Catalogues
**What belongs here:** vendor catalog reference material.

### 16 RENDERINGS
**What belongs here:** resources for *producing* renderings in general
(software, technique references) — not actual project renderings.
**Not this — see instead:** an actual project's renderings belong under
that job in `01 Job-Related` (or `50 Concept Renderings-Digital Plan
Book` for concepts still in design).

### 17 Project Managers
**What belongs here:** project managers' department-level information.

### 99 Best Practices
**What belongs here:** general best-practice reference material — the
existing candidate home for harvested WORKFLOW- content per P4/P7.

---

## Other top-level shares (siblings of `01 Job-Related` / `02 Not-Job-Specific`)

| Share | What belongs here |
|---|---|
| `03 User Folders` | One folder per employee — intended for personal backups |
| `05 Standard Operating Pr[ocedures]` | **Currently empty** — designated future home for SOPs, not yet populated |
| `06 Google Workspace Backup` | Automated backup destination — not a browse/file target |
| `photo` / `photo_1` | Where Synology Photos stores professional project photos — the canonical home going forward for anything that used to land in `01 Job-Related/04 Photos and Video of Projects` |

---

## `04 Marketing Shared` *(top-level, canonical)*

Shared between the Marketing and Sales departments. **Write access is
restricted to PJ Madiera, Adaline, and administrators** (see Access
control above) — everyone else, Designers included, can read but not
write here. A Designer filing something marketing-bound goes to
`01 Job-Related/Marketing Collateral` instead (see that entry above).

Real subfolder list below — **descriptions are inferred from the folder
names, not yet confirmed by Bethany; treat as a first draft, not settled
rules.**

| Subfolder | Best guess at what belongs here (needs confirmation) |
|---|---|
| `Ads and Press` | Paid advertising and press/media placements |
| `Awards` | Award submissions and materials |
| `Brochures` | Printed/digital brochures |
| `ClientPacket` | The packet given to prospective or signed clients |
| `Collateral - Misc` | Marketing collateral that doesn't fit another category here |
| `Communities` | Material organized by neighborhood/community |
| `CONTENT` | Raw content library — copy, photos, video for reuse |
| `Marketing Agencies - Consultants` | Materials related to outside marketing agencies/consultants |
| `PLANS` | Floor plan marketing materials |
| `Realtors` | Materials for realtor/agent partners |
| `REPORTS` | Marketing performance reports |
| `SALES` | Sales department materials/collateral |
| `UPLOADS` | Inbound drop folder — likely a staging area, not a final destination |
| `VISUALS` | Renderings/photos/visual assets for marketing use |
| `Z-Archived Marketing and Sales` | Archived — not a filing destination for new files |

**Open item:** confirm or correct each row above before this section is
trusted for real filing decisions — right now it's a best guess from
folder names alone, exactly the kind of thing this document exists to
replace with your actual intent.

## Keeping BuilderTrend and the server in sync — both directions

**Confirmed by Bethany.** Two rules, not one — they run in opposite
directions and need to both hold:

**Server → BuilderTrend.** A document that's job-related and belongs to
a category BuilderTrend actually tracks — contracts, change orders,
permits/inspections, schedule-affecting documents, vendor/trade
paperwork — should prompt the tool to also ask **"should this go into
BuilderTrend too?"**, separately from asking where it goes on the file
server. If yes, that upload goes through the same human-approved
Browser-Use push-back pattern already built and trusted for BuilderTrend
data (Phase 3) — never a silent, unconfirmed write into BuilderTrend,
for the same reason File It's server writes are confirm-then-execute,
not automatic.

**BuilderTrend → server.** The reverse also holds, and matters just as
much: **if something goes on BuilderTrend, it probably also needs to be
on the server — especially plans or plan documents.** The reasoning is
explicit, not incidental: *"We want to keep copies of our company
documents on our company server... it goes back to us wanting to own
our own data."* This is the same motivation behind the whole ZGX
project running locally instead of on someone else's cloud — BuilderTrend
shouldn't become the only copy of anything that matters.

**A real gap this creates — worth naming, not glossing over:** the
"ask when filing" rule above only fires when someone brings a file
through File It in the first place. It has no visibility into a plan
uploaded directly into BuilderTrend by someone who never touched the
local tool — which is the exact case the "own our own data" principle
most needs to cover. Closing that gap needs a **pull**, not just a
prompt: extending the existing nightly BuilderTrend pull (already
running for Bills/Jobs data, Phase 3) to also check for new
plans/attachments and either sync them to the server automatically or
flag them for a human to bring in — not something a chat-time rule can
guarantee on its own.

**Not every job-related file qualifies for the server→BuilderTrend ask**
— a reference photo or an internal note doesn't need to go into
BuilderTrend just because it's job-related. This still needs its own
short category list (which document types trigger the question).

## Fallback rule

If the document doesn't clearly match a category above, or matches more
than one with similar confidence: **don't guess.** Ask a clarifying
question ("is this for a specific job, or a department-wide record?"),
or route to human review. This applies doubly to anything that might
touch a restricted folder — when in doubt, treat it as restricted.

**Confirmed (Sep 23): this rule applies to Find It, not just File It.**
The single highest-value place to ask rather than guess is the very
first branch decision — job-specific vs. department-wide — since a
wrong guess there sends someone looking in the wrong tree entirely.

## Find It — answer shape (confirmed Sep 23)

Every Find It answer, once a file or folder is identified, should
briefly explain *why* — citing the relevant rule from this document
("per company policy, HOA documents for a job live under that job's
`01 HOA DOCUMENTS` folder") — not just return a bare link. This is
what all the category descriptions and disambiguation notes above are
*for*; the answer should surface that reasoning, not just consult it
silently.

**If nothing matches:** don't return an empty-handed refusal alone.
Return the closest confident parent folder as a starting point (e.g.
"I couldn't find that specific file, but marketing materials like this
generally live under `04 Marketing Shared/PLANS` — here's a link to
that folder") so the person has somewhere real to look, without
inventing a specific file that isn't there.

## File It — replacing an existing file (confirmed Sep 23)

Before writing a new file, check whether a file of the **exact same
name** already exists at the destination.

- **No match found:** write normally, as already specified.
- **Exact name already exists:** ask first — *"A file named `<name>`
  already exists there. Replace it?"* Never overwrite silently.
- **On "yes":** don't delete the old file — move it into a `_archive/`
  subfolder inside the same destination folder, renamed
  `<original-filename>__replaced-YYYYMMDD-HHMMSS.ext`, *then* write the
  new file in its place. This is the same "retire, don't delete"
  discipline already used elsewhere in this project (P4), applied to
  replacements instead of retirements — and it's what keeps
  `placements_log`'s one-click-undo promise meaningful; a true
  overwrite would leave nothing to undo *to*.
- **Starting point, not the end state:** this project may move to a
  true overwrite later, once the tool has earned enough trust that the
  archive step feels redundant — same "confirm-then-execute now, tighten
  later" arc as the rest of Phase 1 (see P7's own framing of this in
  the build spec).

**Keeping `_archive/` from becoming its own mess — retention (confirmed
Sep 23, 90-day default):** a scheduled job (same shape as the existing
nightly/weekly crons already running) deletes anything inside any
`_archive/` folder older than **90 days**. `placements_log` keeps the
permanent record of the replacement (source file, matched job,
destination, timestamp, flagged as a replacement) forever — only the
old file's bytes get reclaimed after 90 days, never the fact that it
happened. The write account needs delete rights, but scoped *only* to
`_archive/` subfolders (Synology's per-subfolder Advanced Permissions
supports this) — the main content tree stays exactly as
create/write-restricted as originally designed; delete exists in
exactly one place, and only for files already marked temporary.

## Open items to resolve before this goes live

1. ~~**HR access list**~~ — **resolved.** See Access control above.
2. ~~**`video` / `Video to Share`**~~ — **resolved.** Confirmed unused; permanently excluded.
3. **`05 Standard Operating Procedures`** is empty — worth deciding whether this becomes the real home for Project 1's harvested workflow content, or stays separate from `99 Best Practices`.
4. **`04 Marketing Shared` subfolder descriptions** are inferred from names only — need Bethany's confirmation or correction before the tool relies on them. (Write-*access* to the folder itself is now confirmed — this item is only about the subfolder-by-subfolder content guesses.)
5. **BuilderTrend job name ↔ server folder mapping doesn't exist yet** — needs to be built (see the note under "01 Job-Related — routing is two-part"). Blocks reliable job matching for both filing and any cross-referencing against BuilderTrend/cost data.
6. ~~**The "also upload to BuilderTrend" rule is proposed, not confirmed**~~ — **confirmed**, and extended to run both directions (server→BuilderTrend and BuilderTrend→server). See "Keeping BuilderTrend and the server in sync" above.
7. **The document-type list that triggers a BuilderTrend-upload question** still needs defining — right now it's a plausible starting set (contracts, change orders, permits/inspections, schedule docs, vendor/trade paperwork), not a confirmed list.
8. **The BuilderTrend→server pull mechanism doesn't exist yet** — closing the "own our own data" gap for files that never touch File It needs an extension to the existing nightly BuilderTrend pull (Phase 3), not just a chat-time rule.
