# Where Do I Find It?

**Project 1 of 2.** The team intranet, scoped to one job: a starting
place anyone can land on and find HR, IT, and administrative
information — without needing to already know where to look. Design and
construction procedures stay in BuilderTrend.

> This used to include a plan to make the ZGX Nano's local AI answer
> questions over the whole file server. That's now a separate, **paused**
> project — see `ZGX_ASSISTANT_PROJECT.md` for where it left off and why.

Rendered version with diagrams: https://claude.ai/code/artifact/98aadf3e-9cab-4b8a-9240-5a52a0f2cbd8

## A0 — The scope line

Everything on the job-and-trade side stays in BuilderTrend. Everything
on the company-and-people side is what this site covers. Test: *does
this describe how we build a house, or how we run the company that
builds houses?*

**Stays in BuilderTrend:** job scheduling & phase sequencing, trade
partner scopes of work, material selections & change orders, permitting,
inspections, punch lists, warranty.

**On the intranet:** IT, HR, and admin policy & procedure; new hire
onboarding, forms, directory; "where is the file for..." / "who do I
ask about..."

## A1 — Audit the existing site first

Before writing anything new, inventory every page on the current site
and mark each: keep / rewrite / merge / kill / move to BuilderTrend.
Also check for: orphan pages, duplicates of content that already lives
in BuilderTrend/Paylocity/the server, pages with no owner or
last-updated date, broken navigation paths (count clicks from Home), and
permissions on anything HR-sensitive.

## A2 — Policy vs. workflow: two kinds of content

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

This isn't just a naming exercise — it changes how a page reads. A
reference page can be short and dense. A workflow page needs numbered
steps a person can follow without re-reading, and should say up front
who to ask if a step doesn't work. Use one template for every workflow
page (Trigger → Steps → Who to ask if stuck → Owner → Last updated) so
people stop re-learning the layout every time.

## A3 — The site plan

Eight top-level sections, everything else nests under one of them:

- **Home** — the Start Here menu (see A4)
- **New Hire Hub** — pre-day-one, Day 1, Week 1, 30/60/90, who's who
- **HR** — *Policy:* handbook, holidays, conduct · *Workflow:* request
  time off (→ Paylocity), offboarding
- **IT** — *Workflow:* phone system, backup setup, book conference room,
  send a fax · *Reference:* BuilderTrend access, server access
- **Administrative** — expenses, purchasing authority, vehicles, travel,
  facilities, brand assets
- **Company Procedures** — *Workflow:* request WFH, referral program
- **Directory & Org Chart**
- **Forms & Documents Library**

Every page carries a footer: Owner / Last reviewed / Next review due. A
page overdue for review gets flagged on Home's "needs attention" list,
not silently left stale.

## A4 — Start Here: the homepage menu

The direct answer to "some people won't know where to start." The
current homepage already half-built this — it literally says "there are
several questions that this site attempts to answer," then lists them.
Formalize that instinct into a menu organized by real-world trigger, not
by department, because a confused person knows "I need to book a room,"
not "which department owns room booking":

| If this is you... | Go here |
|---|---|
| Starting a new job | New Hire Hub |
| Requesting time off or sick time | Paylocity |
| Need to work from home | Workflow: request WFH |
| Booking the conference room | Workflow: book conference room |
| Something's wrong with my phone/computer | Workflow: phone system / backup setup |
| Want to refer a friend | Workflow: referral program |
| Have a policy question | Open the Handbook, or ask HR |
| Anything else | Ask HR (bjenkins@newhousebuilder.com) |

**One rule keeps this from becoming a second wiki:** it stays capped at
~8 items. Adding a 9th means retiring or merging another — see A6.

## A5 — New hire path

- **Day one** — desk/login ready, buddy assigned, handbook
  acknowledgment, and a walkthrough of the Start Here menu — three real
  questions tried live: "where's the handbook," "how do I request time
  off," "who's my IT contact."
- **Week one** — required trainings marked complete on the New Hire Hub;
  role-specific reading assigned by manager.
- **30 / 90 days** — manager check-in, benefits enrollment reminder (→
  Paylocity), access review, formal review closes the onboarding record.

## A6 — Who maintains it — and how it stays evergreen

| Section | Owner | Review cadence |
|---|---|---|
| HR | HR lead | Quarterly, or on any policy change |
| IT | IT lead | Quarterly, or on any tool/vendor change |
| Administrative | Office / admin manager | Quarterly |
| New Hire Hub | HR lead + hiring manager | Every hire; formally every 2 quarters |
| Home / Start Here menu | HR lead | One-in-one-out — never grows past ~8 items |
| Structure & template | Intranet curator (you) | Approves new top-level pages |

**Three things should trigger an update** — and only one of them is
"someone remembered": (1) the system behind a page changes — a vendor
swap, a new tool (this session's own Gusto → Paylocity correction is
exactly that kind of drift); (2) someone flags it — a new hire hits a
dead end and tells their department owner; (3) the scheduled quarterly
review catches the rest.

**Retiring a page:** unpublish, don't delete — Google Sites keeps
version history as the safety net. Pull its link out of the Start Here
menu and any other page that points to it in the same edit.

## A7 — Platform call

**Recommendation: stay on Google Sites for this rebuild.** The company
is already on Google Workspace — Sites is free, integrates with Drive
and Calendar, and the real problem right now isn't the platform, it's
that the site has no structure, no owners, and no template. Fix those
first.

**Where Sites falls short:** no content approval workflow, no real
version history on a page, coarse permissions (whole page/section, not
field-level), search is weak past ~40–50 pages.

**When to revisit this:** headcount pushes past roughly 40–50 people,
you need sign-off workflows on policy changes, or search/findability
complaints persist after A3–A5 are done — then look at Confluence,
Notion, or SharePoint, not before.

## A8 — Build schedule

1. **Harvest what already exists** — pull the ~6 real workflows off the
   current site (phone/Zoom handoff, backup setup, conference room, WFH
   checklist, referral program, BuilderTrend access) and rewrite each on
   the A2 template. No new writing from scratch where content already
   exists.
2. **Foundation** — run the A1 audit, lock the A3 site plan, assign A6
   owners.
3. **Framing** — build the nav shell, stub every page from A3, set
   section-level permissions (HR-sensitive pages restricted).
4. **Rough-in** — write/migrate HR, IT, and Admin content into the
   harvested pages; build the New Hire Hub.
5. **Finish-out** — rebuild Home as the A4 Start Here menu, directory &
   org chart, consistent labeling.
6. **Walkthrough** — pilot with one real new hire or one volunteer per
   department, run the A9 punch list, then announce company-wide.

## A9 — Punch list before occupancy

- Every top-level section has a named owner and a review date in its footer
- All ~6 existing workflows are harvested onto the A2 template, checked
  against the original site content for dropped steps
- Nothing on the intranet duplicates a BuilderTrend SOP — check the A0
  boundary again
- A brand-new hire can complete IT setup and week-one HR tasks using
  only the New Hire Hub and Start Here menu, without asking a person
- Every nav item resolves in two clicks or fewer from Home
- HR-sensitive pages (comp, disciplinary, medical) are
  permission-restricted, not just unlinked
- Old/duplicate pages from the A1 audit are unpublished, not left live
  alongside the new structure
- Home page has a visible "needs attention" list for overdue reviews
