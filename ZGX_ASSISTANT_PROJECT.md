# ZGX Ask-Layer — Phase 1 Scoped

**Project 2 of 2.** A local-AI "where do I find X" layer over the office
file server, built on the ZGX Nano's existing infrastructure.

**Status: Phase 1 defined Sept 22, 2026 — ready to build.** Paused Sept
14 because the server is too messy for full indexing; unblocked Sept 15
with the write-gate idea (P6); scoped down Sept 22 into something much
smaller and more achievable first — see P7. In your own words, on why
the original plan stalled: "I haven't figured out how to get the other
piece of this to work because we have so many folders on the server,
and they're such a mess that it would be very hard for AI to look at it
and figure out how to organize it for us." P7 sidesteps that blocker
rather than solving it head-on.

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

## P5 — Resuming this: start here

1. **Build the P7 Phase 1 tool.** Doesn't require the existing mess
   cleaned up, doesn't require a permissions change, and starts
   producing consistently-named new files immediately.
2. **Make the folder-by-folder call** from P2's "not yet decided" list —
   in scope for the tool, or treated like BuilderTrend SOPs (out of
   scope) — needed either way, since P7's "find it" side has to know
   what it's allowed to search.
3. **Once P7 has run for a while, decide whether to tighten it into
   P6's enforced write-gate**, using real usage as the evidence rather
   than guessing upfront whether people will actually follow a
   suggestion.
4. **Content-level indexing (the original P1–P4 plan) is now optional,
   not required** — only worth doing for folders where path/filename
   search (P7) genuinely isn't cutting it.
5. **The `01 Job-Related` backlog stays a separate, later cleanup**, on
   its own timeline, not a blocker to any of the above.
