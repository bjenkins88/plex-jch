# ZGX Ask-Layer — Paused

**Project 2 of 2.** A local-AI "where do I find X" layer over the office
file server, built on the ZGX Nano's existing infrastructure.

**Status: ON HOLD as of Sept 14, 2026,** with a promising unblock
identified Sept 15 — see P6. In your own words: "I haven't figured out
how to get the other piece of this to work because we have so many
folders on the server, and they're such a mess that it would be very
hard for AI to look at it and figure out how to organize it for us." The
blocker is the folder structure, not the AI — see P5/P6 for what that
means for resuming.

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

## P5 — Resuming this: start here

1. **Stand up the write-gate from P6 first.** It doesn't require the
   existing mess to be cleaned up before it can start, and every day it
   runs is one less day of new mess to eventually deal with.
2. **Make the folder-by-folder call** from P2's "not yet decided" list —
   in scope for the assistant, or treated like BuilderTrend SOPs (out of
   scope).
3. **Pull a handful of real files** from 2–3 confirmed folders (07 HR,
   08 IT, 99 Best Practices are good candidates) and re-run the P3
   proof-of-concept against real content instead of reconstructed
   samples.
4. **Then pick the read-side build schedule back up:** lock down the
   restricted DSM account, adapt the Phase 8 ingestion pipeline for text
   instead of images, wire it into Open WebUI.
5. **Deal with the `01 Job-Related` backlog separately**, on its own
   timeline — a manual/semi-automated cleanup pass, not a blocker to
   starting 1–4.
