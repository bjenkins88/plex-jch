# Where Do I Find It?

An "ask layer" architecture for Jenkins Design Build company knowledge —
not a rebuilt intranet wiki. Real documents stay on the office Synology,
Paylocity stays the HR system of record, BuilderTrend stays the jobs
system of record. The ZGX Nano's existing local AI stack answers "where
do I find X" by indexing and pointing into them, reusing the same
ingestion pattern already proven on the in-progress Phase 8 media-search
pipeline.

Rendered version with diagrams: https://claude.ai/code/artifact/98aadf3e-9cab-4b8a-9240-5a52a0f2cbd8

## A0 — The scope line

Unchanged from the first draft: everything on the job-and-trade side
stays in BuilderTrend. Everything on the company-and-people side is what
the assistant can answer. Test: *does this describe how we build a
house, or how we run the company that builds houses?* The `jdb_costs`
data the ZGX already pulls from BuilderTrend (Phase 3/7) is a separate
concern — financial reporting, not "where do I find the procedure."

## A1 — Ask, not publish

The original plan proposed rebuilding the Google Site into a proper
wiki. The real problem is different: the procedures likely already exist
somewhere, but nobody can tell you which somewhere — and centralizing
them onto a new site just creates a second somewhere that also goes
stale. The fix: leave files where they already live (Synology, Paylocity,
BuilderTrend) and build a thin layer that knows where everything is and
answers in plain language. It doesn't go stale the way a wiki does
because it re-reads the real folders on a schedule instead of holding a
second copy of the truth.

Most of this already exists. The ZGX Nano runs Ollama + Open WebUI as a
private local chat interface today, and the in-progress Phase 8 media
pipeline already proved the exact mechanism needed: pull files from the
real office Synology over a restricted SFTP account, extract and embed
content, store it in Postgres/pgvector, surface it through Open WebUI —
currently aimed at photos and video. Pointing the same pipeline at
documents is the same build, not a new one.

## A2 — Three tiers

- **Source** (systems of record, untouched): office Synology (docs,
  forms), Paylocity (HR self-service), BuilderTrend (jobs, field), plus
  one hand-written `_routing-index.md` that maps non-file answers to the
  right system.
- **Index** (nightly, via n8n): restricted SFTP pull → extract & embed
  text → upsert into a new `company_docs` pgvector table — same pattern
  already running for Phase 8.
- **Ask** (Open WebUI, at the office and over the existing WireGuard
  VPN): answers "where's the expense form?" or "who approves a PO?" with
  the real file path, or a pointer to Paylocity/BuilderTrend.

Nothing in the index tier is a second copy of the truth — it's rebuilt
every night, so it's never more than a day stale even if no one touches
it.

## A3 — The folder plan

Same eight-section taxonomy as the first draft, now describing Synology
folders instead of web pages:

```
/Company/
  01-New-Hire-Hub/       — pre-day-one, Day 1, Week 1, 30/60/90, who's who
  02-HR/                 — handbook, PTO & holiday policy, reviews, conduct, offboarding
  03-IT/                 — accounts, equipment, security policy, helpdesk, remote access
  04-Administrative/     — expenses, purchasing, vehicles, travel, facilities
  05-Company-Procedures/ — how to request time off, submit an expense, get IT help
  06-Directory-Org-Chart/
  07-Forms/
  _routing-index.md      — "if it's not here, it's in Paylocity / BuilderTrend / ..."
```

Each top-level folder gets a short `README.md` at its root — readable by
a person browsing File Station, and embedded by the pipeline as a strong
topic summary that measurably improves retrieval over raw files with no
orientation.

## A4 — How it actually answers

**Ingestion (reuse, don't rebuild):** a restricted DSM account (same
pattern as the existing `aiuser1`), scoped by folder ACL to exactly the
seven A3 folders. Nightly n8n job: SFTP pull changed files → extract
text (Word/PDF/Excel) → embed → upsert into `company_docs`, keyed by
file path. `_routing-index.md` gets embedded the same pass, which is how
the assistant answers questions that aren't a file at all.

**What never gets indexed:** comp, disciplinary, medical, and legal
material — excluded at the file-permission level, not just "the model
won't mention it," the same way `aiuser1` can't see anything outside
`photo/` and `video/` today. If HR wants that material searchable later,
that's a deliberate second build: its own Open WebUI Knowledge
collection gated by Open WebUI's existing RBAC to the HR role only.

## A5 — New hire path

Same shape as before, but the action at each step is "ask," not
"browse": Day 1 includes trying three starter questions live ("where's
the handbook," "how do I request time off," "who's my IT contact"); Week
1 trainings live in `01-New-Hire-Hub/`; 30/90-day check-ins and benefits
reminders point to Paylocity rather than handling enrollment directly.

## A6 — Who maintains it

| Area | Owner | Job |
|---|---|---|
| HR / IT / Admin folders | HR lead · IT lead · admin manager | Keep their Synology folder current — file hygiene, not page editing |
| `_routing-index.md` | ZGX build owner | Update when a system of record changes |
| Index & sync job | ZGX build owner | Watch the nightly n8n run, same discipline as the Bills/media pipelines |
| Query gaps | ZGX build owner + HR lead | Review monthly what people ask and don't get a good answer to |

## A7 — Build schedule (Phase 8B)

Framed as a sibling of the in-progress Phase 8 media pipeline, reusing
its proven pattern against a different corpus:

1. **Organize the folders** — build the A3 tree on the office Synology,
   assign owners, write each README. No ZGX work needed; can start today.
2. **Lock down the account** — new restricted DSM account, folder ACLs
   limited to the seven A3 folders; verify by trying to browse outside
   them and failing.
3. **Adapt the Phase 8 pipeline** — same SFTP + pgvector shape, swap
   image-captioning for text extraction; new `company_docs` table,
   separate from `media_assets` and `jdb_costs`.
4. **Write the routing doc** — pure content work, parallel to steps 2–3.
5. **Pilot, then open it up** — real questions from a few people, fix
   retrieval gaps, then announce company-wide.

## A8 — Front door & punch list

- **At the office** — a shared screen/kiosk with Open WebUI open, logged
  into a low-privilege company account, somewhere central.
- **At a desk** — a bookmark to the office Open WebUI URL (a friendlier
  local hostname than the raw IP if easy to set up).
- **Off-site / jobsite** — the existing WireGuard VPN (Phase 6) already
  covers this.

The Google Site's job shrinks to almost nothing — retire it, or keep one
landing page with the assistant link and a couple of raw folder
shortcuts for people who'd rather browse than ask.

Punch list:
- Every A3 folder has a README and a named owner
- The restricted account can reach only the seven folders — tested, not assumed
- Comp/disciplinary/medical/legal folders are unreachable by that account
- The nightly sync runs clean for a full week before go-live
- `_routing-index.md` covers every system named in A0/A2
- A real new hire finds HR/IT/admin answers by asking, without asking a person
- Query log reviewed weekly for the first month to catch retrieval gaps
