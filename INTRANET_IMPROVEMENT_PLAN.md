# Where Do I Find It?

An "ask layer" over the files that already exist, plus a small set of
real **workflows** pulled out of the current Google Site — so a
confused person has a place to start, and a specific person has a place
to ask. Real documents stay on the office Synology, Paylocity stays the
HR system of record, BuilderTrend stays the jobs system of record. The
ZGX Nano's existing local AI stack answers "where do I find X" by
indexing and pointing into them, reusing the same ingestion pattern
already proven on the in-progress Phase 8 media-search pipeline.

Rendered version with diagrams: https://claude.ai/code/artifact/98aadf3e-9cab-4b8a-9240-5a52a0f2cbd8

## A0 — The scope line

Unchanged: everything on the job-and-trade side stays in BuilderTrend.
Everything on the company-and-people side is what this system covers.
Test: *does this describe how we build a house, or how we run the
company that builds houses?* BuilderTrend access itself (getting an
account, clocking in/out) is IT's job and stays in scope — using
BuilderTrend once you're in it doesn't.

## A1 — Ask, not publish

The current site already tries to do this — its homepage says "there
are several questions that this site attempts to answer," then lists
them. That instinct was right; it just had to be maintained by hand,
page by page, forever. Leave real files where they already live
(Synology, Paylocity, BuilderTrend) and build a thin layer that knows
where everything is and answers in plain language, re-reading the real
folders on a schedule instead of holding a second copy of the truth.

The ZGX Nano already runs the pipeline this needs: pull files from the
office Synology over a restricted account, extract and embed content,
store it in pgvector, surface it through Open WebUI — proven on the
in-progress Phase 8 media pipeline. Pointing it at documents is the same
build, not a new one.

## A2 — Three tiers

- **Source** (systems of record, untouched): office Synology (docs,
  forms), Paylocity (HR self-service), BuilderTrend (jobs, field), plus
  `_routing-index.md` and `_start-here.md`.
- **Index** (nightly, via n8n): restricted SFTP pull → extract & embed
  text → upsert into a `company_docs` pgvector table — same pattern
  already running for Phase 8.
- **Ask** (Open WebUI + the Google Site's homepage): answers a specific
  question, walks a workflow step by step, or hands over the Start Here
  menu when a question is too vague to match anything.

## A3 — Policy vs. workflow: two kinds of content

The current site already mixes two very different things under one
roof, and that's a real source of "where do I even start." Look at
what's actually sitting under **Company Policies** today: the Team
Member Handbook (a real policy — safety rules, social media policy) sits
right next to "Backing up your Hard Drive" and the referral program —
neither of which is a policy. One is a fact you look up once. The other
is a sequence of steps you follow in order, sometimes with a decision in
the middle ("only when pre-approved by your supervisor").

**Reference — look it up, done:** BuilderTrend login, server address,
holiday/safety/social media policy, logo files, signature line template.

**Workflow — steps, in order, sometimes a decision:** parking a call on
the desk phone (4 steps), backing up a laptop (install → point at
`jdbsrv` → sign in → set Continuous Backup), booking the conference room
(add the calendar → use it as your meeting location), requesting
work-from-home (supervisor pre-approval → checklist form), the Jenkins
Finder's referral program (get contact info → send to Sales *before*
they reach out → $1,000 gift card).

This matters for the AI too: a reference fact is safe to retrieve and
quote. A workflow is **not safe to paraphrase** — a model summarizing
"backing up your hard drive" could quietly drop the Continuous Backup
step. Workflow files get tagged as a distinct type (a `WORKFLOW-`
filename prefix is enough) and the assistant is instructed to return
them as literal numbered steps, never a summary.

## A4 — The folder plan

Same eight-section taxonomy as before, now with real files in it —
reference and workflow content side by side, tagged, not split into
different systems:

```
/Company/
  01-New-Hire-Hub/
  02-HR/
    POLICY-team-member-handbook.pdf   (exported from the Google Doc)
    WORKFLOW-request-time-off.md      (points to Paylocity)
  03-IT/
    WORKFLOW-phone-system.md          (Zoom Phone, parking a call, Polycom steps)
    WORKFLOW-backup-setup.md          (Synology Drive client, Continuous Backup)
    WORKFLOW-book-conference-room.md
    REFERENCE-buildertrend-access.md
    WORKFLOW-using-the-assistant.md   (exists elsewhere — import, don't rewrite)
  04-Administrative/                  (logo files, signature line, brand assets)
  05-Company-Procedures/
    WORKFLOW-request-wfh.md
    WORKFLOW-referral-program.md      (Jenkins Finder's Program)
  06-Directory-Org-Chart/
  07-Forms/
  _routing-index.md                   ("if it's not here, it's in Paylocity / BuilderTrend / ...")
  _start-here.md                      (the menu from A5)
```

## A5 — Start Here: the workflow menu

The direct answer to "some people won't know where to start to ask
questions." The homepage already half-built this — a short bulleted
list of the questions the site answers. Formalize that instinct into a
menu organized by real-world trigger, not by department, because a
confused person knows "I need to book a room," not "which department
owns room booking":

| If this is you... | Go here |
|---|---|
| Starting a new job | New Hire Hub |
| Requesting time off or sick time | Paylocity |
| Need to work from home | Workflow: request-wfh |
| Booking the conference room | Workflow: book-conference-room |
| Something's wrong with my phone/computer | Workflow: phone-system / backup-setup |
| Want to refer a friend | Workflow: referral-program |
| Have a policy question | Ask the assistant, or open the Handbook |
| **Don't know how to get to or use the assistant itself** | Workflow: using-the-assistant |
| Anything else | Ask the assistant |

**Note:** that last row needs its own workflow — assume nobody knows how
to reach or use the ZGX/Open WebUI on their own. Instructions for this
already exist elsewhere; this plan doesn't duplicate them, just flags
that they need a home in `03-IT/` and a line in this menu once folded
in.

One file, two homes: it's `_start-here.md` on the Synology (indexed like
everything else, so the assistant can open with it when a question is
too vague to match), **and** it's the Google Site's homepage — kept, not
retired, because it's already what everyone has bookmarked. The Google
Site's whole remaining job becomes this one menu.

## A6 — How it actually answers

**Ingestion (reuse, don't rebuild):** restricted DSM account, same
pattern as the existing `aiuser1`, scoped by folder ACL to exactly the
A4 folders. Nightly n8n job: SFTP pull → extract text → embed → upsert
into `company_docs`, keyed by file path. The retrieval prompt
distinguishes `WORKFLOW-` files (return steps verbatim) from everything
else (answer and cite).

**What never gets indexed:** comp, disciplinary, medical, and legal
material — excluded at the file-permission level, the same way
`aiuser1` can't see outside `photo/` and `video/` today. If HR wants
that searchable later, it's a separate Open WebUI Knowledge collection
gated by RBAC to HR only — a deliberate second build.

## A7 — Who maintains it

| Area | Owner | Job |
|---|---|---|
| HR / IT / Admin folders | HR lead · IT lead · admin manager | Keep files current; turn a process change into an updated `WORKFLOW` file, not a paragraph edit buried in a policy doc |
| `_start-here.md` | ZGX build owner + HR lead | Keep the menu to ~8 items — it's a menu, not an index of everything |
| `_routing-index.md` | ZGX build owner | Update when a system of record changes |
| Index & sync job | ZGX build owner | Watch the nightly n8n run, same discipline as the Bills/media pipelines |
| Query gaps | ZGX build owner + HR lead | Review monthly what people ask and don't get a good answer to |

## A8 — Build schedule (Phase 8B)

1. **Harvest what already exists** — pull the ~6 real workflows off the
   current site (phone/Zoom handoff, backup setup, conference room, WFH
   checklist, referral program, BuilderTrend access) and rewrite each as
   its own numbered `WORKFLOW-` file. Export the Team Member Handbook
   off Google Docs onto the Synology as the canonical file. Fold in the
   existing "how to reach and use the ZGX assistant" instructions as
   `WORKFLOW-using-the-assistant.md` — that material already exists,
   just needs to land here rather than being rewritten.
2. **Organize the folders** — build the A4 tree on the office Synology,
   assign owners, write each README.
3. **Lock down the account** — new restricted DSM account, folder ACLs
   limited to the A4 folders; verify by trying to browse outside them
   and failing.
4. **Adapt the Phase 8 pipeline** — same SFTP + pgvector shape, swap
   image-captioning for text extraction, add the workflow-vs-reference
   tagging from A3/A6.
5. **Rebuild the homepage as Start Here** — replace the current
   homepage's Q&A prose with the A5 menu; can ship independently of the
   AI work.
6. **Pilot, then open it up** — real questions from a few people, check
   that workflow answers come back as full steps, then announce
   company-wide.

## A9 — Front door & punch list

- **Don't know where to start** — Google Site homepage → the A5 Start
  Here menu. No login, no typing a question, just pick your situation.
- **Know exactly what to ask** — Open WebUI, at the office (kiosk or
  shared screen) or at a desk via bookmark.
- **Off-site / jobsite** — the existing WireGuard VPN (Phase 6) already
  covers this.

Punch list:
- Every A4 folder has a README and a named owner
- All 6 existing workflows are harvested into `WORKFLOW-` files, checked
  against the original site content for dropped steps
- The restricted account can reach only the A4 folders — tested, not assumed
- Comp/disciplinary/medical/legal folders are unreachable by that account
- The nightly sync runs clean for a full week before go-live
- The assistant returns a workflow as its literal steps, not a
  paraphrase — checked on at least 2 real workflow files
- A real new hire finds an answer starting from the homepage menu *and*
  by asking directly, without asking a person

## A10 — Staying evergreen

A workflow is evergreen only if changing it is easier than leaving it
wrong.

**Three things should trigger a change** — and only one of them is
"someone remembered":

1. **The system changes** — a vendor swap, a new tool, a new approval
   chain. This conversation's own Paylocity correction is exactly the
   kind of drift this has to catch on purpose, not by luck — a payroll
   switch should trigger a workflow update as part of the rollout, not
   an afterthought someone notices months later.
2. **Someone flags it** — a query comes back wrong or empty (caught by
   the A7 monthly query-gap review), or a person just tells their
   department owner "this step is wrong now."
3. **Scheduled review** — the A7 quarterly folder review catches the
   slow drift nobody happened to notice.

**The loop — no separate "publish" step:**

1. Trigger noticed (any of the three above)
2. Owner edits the file directly on the Synology, using the template
   below — no CMS, no ticket, no approval queue for a routine update
3. The nightly reindex (the same n8n job from A6) picks it up
   automatically — the owner doesn't do anything extra to "publish" it
4. Live everywhere — in the assistant within a day, and in
   `_start-here.md` too if it's common enough to earn a menu line

**Every `WORKFLOW-` file, same shape:**
- **Trigger** — the real-world situation that sends someone here
- **Steps** — numbered, in order, decisions called out explicitly
- **Who to ask if stuck** — a name, not just "IT"
- **Owner · Last updated** — same footer discipline as A7

**Retiring a workflow:**
- Move the file to `_archive/`, don't delete it — Synology's own Drive
  versioning (already in use today) is the real safety net
- An archived file is excluded from the index — it stops being retrieved
- Scrub any pointer to it out of `_start-here.md` and
  `_routing-index.md` in the same edit

**One rule keeps Start Here from becoming the wiki again:** it stays
capped at ~8 items. Adding a 9th means retiring or merging another —
gatekept by the same two people who own the file (A7), not a
free-for-all.

| Who | Can do |
|---|---|
| Anyone | Flag a workflow that's wrong, missing, or confusing — to their department owner, informally |
| Department owner (HR / IT / Admin) | Write and edit any `WORKFLOW-` file in their own folder, on the template above |
| ZGX build owner + HR lead | The only two who touch `_start-here.md` and `_routing-index.md` — small surface, kept deliberately narrow |
