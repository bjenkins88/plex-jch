# Team Hub Site Plan

A phased plan for rebuilding the New House Builder team intranet
(sites.google.com/newhousebuilder.com/team) so it covers company-level HR,
IT, and administrative procedures, while design and construction SOPs
stay in BuilderTrend.

Rendered version with diagrams: https://claude.ai/code/artifact/98aadf3e-9cab-4b8a-9240-5a52a0f2cbd8

## A0 — The scope line

Everything on the job-and-trade side stays in BuilderTrend. Everything on
the company-and-people side moves to the intranet. Test: *does this
describe how we build a house, or how we run the company that builds
houses?*

**Stays in BuilderTrend:** job scheduling & phase sequencing, trade
partner scopes of work, material selections & spec sheets, change order
procedure, permitting & inspection checklists (per job), punch list /
final walkthrough procedure, warranty & callback procedure, daily log /
field reporting standard.

**Moves to the intranet:** IT (accounts, equipment, security, helpdesk),
HR (handbook, benefits, PTO, reviews, conduct), Admin (expenses,
purchasing, vehicles, travel), new hire onboarding, culture, org chart,
company-wide meeting & comms norms, office-level safety & emergency
contacts.

Exception: office-level safety (fire exits, emergency contacts, workers'
comp reporting) sits on the intranet; jobsite safety (PPE by trade, OSHA
logs tied to a project) stays in BuilderTrend.

## A1 — Audit the existing site first

Before writing anything new, inventory every page on the current site and
mark each: keep / rewrite / merge / kill / move to BuilderTrend. Also
check for: orphan pages, duplicates of content that already lives in
BuilderTrend/Gusto/Drive, pages with no owner or last-updated date, broken
navigation paths (count clicks from Home), and permissions on anything
HR-sensitive.

## A2 — The site plan

Eight top-level sections, everything else nests under one of them:

- **Home** — announcements, quick links, search
- **New Hire Hub** — pre-day-one, Day 1, Week 1, 30/60/90, who's who
- **HR** — handbook, benefits (link to Gusto), PTO & holidays, reviews,
  conduct, offboarding
- **IT** — new account & hardware, helpdesk, approved software, security
  policy, remote access
- **Administrative** — expenses, purchasing authority, vehicles & fuel
  cards, travel, facilities
- **Company Procedures** — how to request time off, submit an expense,
  get IT help, propose a new SOP
- **Directory & Org Chart**
- **Forms & Documents Library**

## A3 — New hire path

A path, not a folder: pre-day-one (offer letter, I-9/W-4 in Gusto, IT
equipment ordered) → Day 1 (desk/login ready, buddy assigned, handbook
acknowledgment) → Week 1 (required trainings marked complete) → 30 days
(manager check-in, benefits enrollment reminder, access review) → 90 days
(formal review, hub sign-off).

## A4 — Who maintains it

| Section | Owner | Review cadence |
|---|---|---|
| HR | HR lead | Quarterly, or on any policy change |
| IT | IT lead / MSP contact | Quarterly, or on any tool/vendor change |
| Administrative | Office / admin manager | Quarterly |
| New Hire Hub | HR lead + hiring manager | Every hire; formally every 2 quarters |
| Structure & template | Intranet curator | Ongoing |

Every page carries a footer: Owner / Last reviewed / Next review due.
One SOP template reused everywhere: Purpose → Scope → Owner → Steps →
Related forms/links → Related BuilderTrend reference (if any).

## A5 — Build schedule

1. **Foundation** (weeks 1–2) — run the A1 audit, lock the A2 site plan,
   assign A4 owners, pick the SOP template.
2. **Framing** (weeks 3–4) — build the nav shell, stub every page, set
   section-level permissions.
3. **Rough-in** (weeks 4–7) — write/migrate HR, IT, Admin content; build
   the New Hire Hub; link out to Gusto/Drive instead of duplicating.
4. **Finish-out** (weeks 6–8) — directory & org chart, cross-links to
   BuilderTrend at the scope line, consistent labeling/search aids.
5. **Walkthrough** (weeks 8–9) — pilot with one real new hire or one
   volunteer per department, run the A7 punch list, then announce
   company-wide.

## A6 — Platform call

**Recommendation: stay on Google Sites for this rebuild.** The company is
already on Google Workspace; the current problem is structure and
ownership, not the platform. Revisit only if headcount pushes past ~40–50
people, you need policy sign-off workflows, or search/findability
complaints persist after A2–A5 are done — then look at Confluence,
Notion, or SharePoint.

## A7 — Punch list before occupancy

- Every top-level section has a named owner and a review date
- Nothing on the intranet duplicates a BuilderTrend SOP
- A new hire can complete IT setup and week-one HR tasks using only the
  New Hire Hub
- Every nav item resolves in two clicks or fewer from Home
- HR-sensitive pages are permission-restricted, not just unlinked
- Old/duplicate pages from the A1 audit are archived or redirected
- Home page has a visible "needs attention" list for overdue reviews
