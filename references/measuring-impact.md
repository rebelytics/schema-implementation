# Phase 6: Measuring Impact — reference for the schema-implementation skill

Load triggers: the client asks what the structured data achieved; writing an impact ticket or analysis brief

Reference file for the `schema-implementation` skill (Eoghan Henn and contributors, CC BY 4.0). The core `SKILL.md` — the phase overview, §2.8 Universal Technical Requirements, the Anti-Patterns list and the Required User Input checklist — applies alongside this file. Section numbers (§1.2, §5.4) and rule numbers (Rule 35) are the same in every file; the core's section map says which file holds each phase.

## Phase 6: Measuring Impact

**Goal:** When the client asks what the structured data achieved, establish
*which channel the effect could appear in* before measuring anything — and
write that expectation into the impact ticket before the analysis starts.

Most schema types never produce a reportable SERP feature. An impact
analysis that starts from "did rich results go up?" without first checking
whether the implemented types can appear as rich results at all will
measure nothing, and the client should learn that from the plan rather than
from the result.

### 6.1 First Call — the Search Appearance Breakdown

Open the Search Console Performance report, group by **Search appearance**,
over the longest window available (16 months). This is a five-minute check
that decides the shape of the whole analysis. Then list, per implemented
type, whether it can produce a row there at all:

- **Can appear** (a Search appearance row is possible, so a before/after
  on that row is a legitimate channel): Event, JobPosting, Product
  (merchant listings / product snippets), Recipe, VideoObject, FAQPage
  (now heavily restricted by Google), Review snippets, HowTo (retired for
  most sites), plus whatever rows the report already shows.
- **Never appear** (no Search appearance row exists by design): Organization,
  BreadcrumbList, ProfilePage, WebPage, WebSite, the Article family
  (Article / NewsArticle / BlogPosting), LocalBusiness and its subtypes
  (LegalService, MedicalBusiness, …), Person, ItemList without a
  carousel-eligible child type. These types feed entity understanding,
  breadcrumb rendering and knowledge-panel data — none of which shows up
  as a search appearance.

A type in the first group can still be absent from the report: an
appearance row only exists once Google has actually served the feature.
Valid items of an eligible type can be live for weeks and still produce no
row for that type, while the only rows present belong to a type that predated
the project. Treat an empty
row as a finding in itself (eligibility is not display), not as a data
problem to be fixed by waiting.

### 6.2 State the Expectation Before the Analysis

Write the outcome of §6.1 into the impact ticket (or the analysis brief)
*before* any numbers are pulled: which types could have produced a
rich-result lift, which cannot by design, and therefore which channels the
analysis will use. This is §1.1c (verify Google's current feature
support) applied to measurement instead of implementation. A
client who expects a rich-result chart for their Organization or
BreadcrumbList markup should hear that it does not exist from the plan,
not from a report that shows no change.

### 6.3 The Measurable Channels That Remain

- **CTR by page type, before/after each deployment date.** Structured data
  ships per page type on different dates (Phase 4 delivery order), which
  gives a staggered rollout for free. For each page type, compare CTR in
  the weeks before and after its deployment date, *within position
  buckets* (1–3, 4–10, 11–20) or on **query–page pairs present in both
  periods** — never on raw aggregate CTR, which moves with ranking and
  query mix. Page types not yet deployed in the same window are the
  controls: if their CTR moved the same way, the change is seasonal or
  algorithmic, not the schema.
  Where no API or BigQuery access exists, the Performance report UI still
  yields a daily clicks / impressions / CTR / position table per page
  type: the report keeps its state in its URL (date window, a regex page
  filter for the page type, a query exclusion for brand terms, the date
  breakdown with all four metrics), so each series is one navigation plus
  a rows-per-page switch, and the daily table under a page filter
  reconciles with the filtered total where dimension tables do not. Pull
  one series per page type, with and without the brand exclusion, and
  reconcile each against its filtered total before using it.
  **A denominator move is not an uplift.** An aggregate CTR rise with flat
  clicks and falling impressions is a denominator effect — fewer
  low-position impressions, the same clicks — and reads as a rise only
  because CTR is a ratio. Report clicks and impressions beside every CTR
  figure, and claim an uplift only where clicks rose within a position
  bucket or on matched query–page pairs.
- **SERP sampling for breadcrumb rendering.** BreadcrumbList has no
  Performance row, but its effect is visible: sample live SERPs for the
  page types that received it and record whether the URL line renders as
  a breadcrumb path or a raw URL. Repeat per locale on multilingual sites.
- **AI-visibility trackers — correlational only.** Where an AI-visibility
  tracker (the `ai-visibility-tracking-strategy-builder` family, or the
  client's own) was already running before deployment, citation and
  mention rates for the affected page types before/after are worth
  reporting — labelled as correlation. Where tracking started at or after
  deployment there is no baseline, and the analysis must say so rather
  than present the post-period as evidence.
- **Search appearance rows, where §6.1 found any.** For eligible types
  that did produce a row: impressions and clicks on that row over time,
  plus the Rich Results / Enhancements report item counts as the supply
  side.

### 6.4 Data Retention — Pull the Export Now

GSC's 16-month window erodes the pre-deployment period a month at a time.
As soon as the analysis is agreed — not when it is scheduled — pull the
daily page-level export (page × date × query where volumes allow, page ×
date at minimum) for every page type in scope, and file it with the
project. Check whether the property has **BigQuery bulk export** enabled;
if it does, the retention problem disappears and the analysis can run on
unsampled data; if it does not, enabling it starts retention from that
day forward only, so the export above is still needed for the pre-period.

**Principle:** most schema types never produce a reportable SERP feature.
Establish the channel first, state it in the plan, then measure only what
can move — and secure the pre-period data before the window eats it.
