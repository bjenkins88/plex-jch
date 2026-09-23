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

## Access control — read this before anything else

Three folders are restricted. **The general Find It / File It tool must
never search, suggest, or write into these** unless the person asking
has been specifically confirmed to have access. This isn't a filing
preference, it's a hard boundary — same principle already used for
sensitive HR content in the rest of this project.

| Folder | Restricted to |
|---|---|
| `02 Not-Job-Specific/07 HR (Human Resources)` | Specifically granted individuals only — exact list still to be defined (Bethany to specify) |
| `09 Accounting` *(top-level share, not the "03 Accounting Procedures" folder below)* | Peggy, Shan, Bethany, Truitt only |
| `Accounting-Backup-ONLY` | Same access as `09 Accounting` |

**Open item:** the exact HR access list needs to be written down before
this goes live — "specifically given it" isn't enough for the tool to
enforce.

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

**Purpose unconfirmed — exclude until checked:** `video`,
`Video to Share` (not described yet; confirm before including either
as a real destination)

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
- `Marketing Collateral` — **known misplacement.** Should be under the
  top-level `04 Marketing Shared` share instead. Don't file new items
  here even though old ones exist.
- `Temporary Items`, `Website` — legacy; `Website` in particular is
  flagged for archiving. Not filing destinations.

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
department-level material.

### 03 Accounting Procedures
**What belongs here:** how the accounting department does things —
procedures and process documents.
**Not this — see instead:** actual financial data/records go in the
restricted `09 Accounting` top-level share, not here — this folder is
about process, not the numbers themselves.

### 04 Marketing Shared *(nested copy — likely a duplicate)*
**Flagged by Bethany as probably a mistake.** The real, canonical
Marketing Shared is the top-level `04 Marketing Shared` share (shared
between Marketing and Sales), not this nested folder. Route new files
to the top-level share; don't perpetuate the duplicate.

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

Shared between the Marketing and Sales departments. Real subfolder list
below — **descriptions are inferred from the folder names, not yet
confirmed by Bethany; treat as a first draft, not settled rules.**

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

## Fallback rule

If the document doesn't clearly match a category above, or matches more
than one with similar confidence: **don't guess.** Ask a clarifying
question ("is this for a specific job, or a department-wide record?"),
or route to human review. This applies doubly to anything that might
touch a restricted folder — when in doubt, treat it as restricted.

## Open items to resolve before this goes live

1. **HR access list** — needs the actual names, not "specifically given it."
2. **`video` / `Video to Share`** — purpose unconfirmed; exclude until Bethany describes them.
3. **`05 Standard Operating Procedures`** is empty — worth deciding whether this becomes the real home for Project 1's harvested workflow content, or stays separate from `99 Best Practices`.
4. **`04 Marketing Shared` subfolder descriptions** are inferred from names only — need Bethany's confirmation or correction before the tool relies on them.
