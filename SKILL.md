---
name: schema-implementation
description: Use this skill when the user wants to implement schema.org structured data across a website. Covers the full workflow from page type analysis through developer-ready ticket creation (JIRA, Wrike, or other project management tools). Triggers include any mention of "schema.org", "structured data", "rich results", "JSON-LD", "JIRA tickets for schema", "Wrike tasks for schema", or requests to analyze website pages for SEO markup. Also use when the user references schema implementation methodology, ticket writing rules, or page type analysis workflows.
license: CC-BY-4.0
metadata:
  author: Eoghan Henn and contributors
  source: https://github.com/rebelytics/schema-implementation
  version: "1.0.1"
  core_max_lines: "420"
---

# Schema.org Structured Data Implementation — Full Workflow Skill

This skill provides an end-to-end methodology for analyzing websites, designing schema.org markup, and producing developer-ready tickets. It has been refined through real-world projects implementing structured data across multilingual, multi-regional websites.

The methodology is client-agnostic. It works for any website where the goal is comprehensive schema.org coverage with developer handoff via a project management tool (JIRA, Wrike, or similar).

**Licence:** This skill is released under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
licence. You are free to share and adapt this skill for any purpose, provided
you give appropriate credit to the original author.

**Related audits:** this skill's Phase 5 verifies the live output against the spec designed in Phase 2. When the task is instead verifying a *reported fix* across locales ("fixed for all languages", "deployed everywhere"), load the published companion `jsonld-validation-audit` (https://github.com/rebelytics/jsonld-validation-audit), which holds the claim-verification disciplines this skill does not: the cache-busting gate, population enumeration from the sitemap index / live hreflang set, and the positive control. The audit end of breadcrumb work is the published companion `breadcrumb-navigation-audit` (https://github.com/rebelytics/breadcrumb-navigation-audit); this skill handles its ticket end.

---

## How This Skill Works

This is a project skill, typically completed in 1–3 hours across 1–2 sessions for most websites. Complex multilingual sites with many page types may take longer. The skill defines five sequential delivery phases: Phase 0 (Setup), Phase 1 (Discovery), Phase 2 (Schema Design), Phase 3 (Ticket Creation), and Phase 4 (Regeneration & Delivery) — followed by two post-deployment phases that run once the developer has shipped: Phase 5 (Post-Deployment Verification) and Phase 6 (Measuring Impact). Claude should identify which phase the user is in and guide them through it.

**The user provides:** Project context, feedback on analysis and ticket drafts, project-specific rules, resolution of open questions.
**Claude provides:** Page type discovery, HTML analysis, schema design, ticket drafts (parent + child), and a final regenerated set of tickets.

**Terminology note:** This skill uses "ticket" as a generic term for the developer-facing deliverable. The actual format depends on the project management platform chosen in Phase 0 (JIRA tickets, Wrike tasks, etc.).

---

## Reference files — load on demand

Phases 1 to 6 live in reference files, one file per phase. Each pointer names
its trigger. These loads are mandatory steps: when a phase starts, load its
file before doing the work — never improvise a phase from this core file. If
you notice a phase was handled without its file loaded, say so, load it, and
re-check the work done so far against it.

- `references/discovery.md` — Phase 1: page type catalog, triage, Google
  guidance check, per-page-type analysis (existing schema in all three
  formats, conditional schema triage, locale delivery), saving real data,
  consolidation, classification. **Load before building the page type
  catalog and before analysing any page type's HTML.**
- `references/schema-design.md` — Phase 2, sections 2.1–2.7: global and
  page-specific schema, `@id` cross-references, deprecation vs removal,
  format-migration dispositions, property selection rules, multilingual
  property values, verifying assumptions, project rules. **Load before
  choosing schema types or properties for any page type.**
- `references/ticket-writing.md` — Phase 3: pre-flight, the ticket template,
  subagent guidelines, the numbered ticket-writing rules (1–46), the parent
  ticket, the pre-delivery checklist, the feedback workflow, and Common
  Schema Decisions (§3.7). **Load before drafting, revising or checking any
  ticket, before briefing a subagent on tickets, and when a §3.7 decision
  comes up in design.**
- `references/delivery.md` — Phase 4: regeneration plan, platform-specific
  formatting (Wrike, JIRA, rich-text editors, WAF-protected tools), the
  delivery-format decision tree, editorial pass, cross-ticket consistency
  checks, packaging, Wrike XLS + HTML paste. **Load before the final
  regeneration and before formatting tickets for the target tool.**
- `references/post-deployment-verification.md` — Phase 5: the
  full-population sweep triggers, the Phase 5 pre-flight checklist,
  cross-language verification and fetch techniques, the four finding
  categories and report skeleton, follow-up comments, URL and source-value
  verification, GSC error reconciliation and diagnosis, single-fetch false
  greens, coverage audits from crawl exports. **Load before verifying any
  deployed schema, before confirming a ticket as done, before replying to a
  client-reported discrepancy, and before diagnosing a Search Console
  structured-data error.**
- `references/measuring-impact.md` — Phase 6: the Search appearance
  breakdown, the expectation statement, the measurable channels, data
  retention. **Load when the client asks what the structured data achieved,
  before any impact numbers are pulled.**

Section numbers (§1.2, §5.4) and rule numbers (Rule 35, anti-pattern 21) are
the same in every file, so a cross-reference resolves through this map.

---

## Phase 0: Setup & Tooling

**Goal:** Assess available tools and access, set up the most efficient workflow, and gather project-level preferences before starting content work.

### 0.1 Assess Available Tools

Before any analysis work begins, determine what capabilities are available:

- **Browser access** (e.g., Claude in Chrome extension, Cowork browser tools) — Enables Claude to navigate pages directly, capture rendered HTML, extract data, and verify facts autonomously. This is the most efficient workflow.
- **Web search** — Enables Claude to map site structure, find sitemaps, and discover page types without browser access.
- **File system access** — Enables saving analysis documents and ticket drafts incrementally.
- **Direct site access** — Some sites block automated access (403 errors). If web fetching fails, browser access or manual HTML upload are fallbacks.

### 0.2 Guide the User Toward Browser Access

Browser-assisted analysis is the **primary recommended workflow** for this skill. It enables Claude to:
- Discover and map page types autonomously
- Capture rendered HTML from any page without manual copy-paste
- Extract detection signals, data sources, and template structures in minutes
- Verify facts (URL patterns, language variants, field availability) before writing tickets

If browser access is not yet available, explain the benefits and guide the user through setup. The user's role shifts from "HTML gatherer" (tedious, low-value) to "reviewer and decision-maker" (high-value, where their domain expertise matters).

If browser access is unavailable or the site blocks automated browsing entirely, fall back to the manual workflow described in Phase 1.2.

**Cost awareness:** Browser access enables the most thorough analysis but
is significantly more expensive in token usage — Chrome tool calls return
large payloads that compound quadratically as the conversation grows.
Reserve browser workflows for initial discovery and verification phases
where real-time page inspection adds genuine value. For tasks where the
page type inventory is already established (e.g., writing tickets from a
completed audit), non-browser workflows are equally effective and much
cheaper. When browser is used, prefer targeted calls (`get_page_text`,
`find`, `javascript_tool` for surgical extraction) over heavy calls
(`read_page` for full DOM, `read_network_requests` for full traffic).

### 0.3 Gather Project Preferences

Before starting content work, ask the user:

1. **Project management platform** — "Which system will these tickets be created in? (JIRA, Wrike, Asana, etc.)" This determines terminology (tickets vs tasks vs items), formatting constraints (e.g., Wrike doesn't render nested code blocks inside list items — see §4.4 Platform-Specific Formatting Pass in `references/delivery.md`), and the naming convention for deliverable files.
2. **Ticket structure** — "Does your organisation have an established ticket structure or template for developer tasks? If so, please share a template or ticket creation guidelines. I'll adapt the schema tickets to match your team's conventions." If no template exists, use the skill's default structure.
3. **Greeting and tone** — e.g., "Hi team," with peer-to-peer tone, or formal.
4. **Project-specific rules** — Privacy constraints, terminology, brand rules, etc. (These can also be gathered during Phase 2.)

---

## Phase 1: Discovery & Page Type Catalog

**Goal:** Build a complete inventory of all page types on the website, with detection signals, existing schema coverage, and saved real data for ticket code examples.

**Load `references/discovery.md` before starting this phase.** It holds the whole phase:

- 1.1 Gather the Page Type List
- 1.1b Priority Assessment / Triage
- 1.1c Verify Google's Current Guidance
- 1.2 Analyze Each Page Type
- 1.3 Save Real Data for Ticket Code Examples
- 1.4 Document Findings
- 1.5 Identify Consolidation Opportunities
- 1.6 Classify Each Page Type

---

## Phase 2: Schema Design

**Goal:** Decide which schema types and properties to use for each page type, and how schemas relate to each other across the site.

**Load `references/schema-design.md` before choosing schema types or properties.** It holds sections 2.1–2.7:

- 2.1 Design the Global/Shared Schema
- 2.2 Design Page-Specific Schemas
- 2.3 Design Cross-References
- 2.4 Deprecation vs Removal
- 2.4b Format Migration Tickets — Disposition of the Existing Format
- 2.5 Apply Property Selection Rules
- 2.5b Multilingual Property Values
- 2.6 Verify Assumptions Before Finalizing
- 2.7 Project-Specific Rules

§2.8 stays in this core file because every ticket and every verification applies it:

### 2.8 Universal Technical Requirements

These apply to every project:
- **Schema must be server-side rendered** — Present in the initial HTML returned from the server, not dynamically injected by JavaScript. This is an SEO best practice, not a project-specific preference.
- **JSON-LD format** — Always implement as `<script type="application/ld+json">` in the `<head>` section.
- **Separate script tags** — Each schema type gets its own `<script>` tag.
- **@context format** — Always use `https://schema.org` (no trailing slash). Enforce
  this consistently across all tickets.
- **Emit via a real JSON serializer** — Never hand-assemble JSON-LD by string
  concatenation/templating of CMS values. Hand-built JSON-LD is prone to a whole
  class of breakage, broader than unescaped characters alone: unescaped quotes in
  interpolated values, smart/curly-quote substitution (U+201C/U+201D/U+201E where a
  straight `"` belongs), and structural errors such as stray or duplicated braces
  (e.g., an `@id` value wrapped in an extra set of braces). A serializer eliminates
  the entire class; escaping rules alone do not.
- **Decode HTML entities in text values** — A text property carries the
  decoded character (`'`, `&`), never the HTML entity (`&#039;`, `&amp;`) that a
  CMS field or the page source stores. Where the source field holds encoded
  text, the template decodes it before serialising.
- **Dates in ISO 8601, date-times with a timezone offset** — Every date and
  date-time property (`datePublished`, `dateModified`, `startDate`,
  `validFrom` …) uses ISO 8601; a date-time carries its offset
  (`YYYY-MM-DDThh:mm:ss+hh:mm`), never a bare local time or a display format
  copied from the page.

---

## Phase 3: Ticket Creation

**Goal:** Produce developer-ready tickets — one parent ticket for shared schemas, one child ticket per page type (or consolidated group).

**Load `references/ticket-writing.md` before drafting, revising or checking any ticket.** It holds the whole phase:

- 3.1 Pre-Flight Check
- 3.2 Ticket Structure Template
- 3.2b Subagent Usage Guidelines
- 3.3 Ticket-Writing Rules
- 3.4 Writing the Parent Ticket
- 3.5 Pre-Flight Checklist (Before Delivery)
- 3.6 Feedback & Iteration Workflow
- 3.7 Common Schema Decisions

---

## Phase 4: Final Regeneration & Delivery

**Goal:** Regenerate all tickets in a single pass, applying every accumulated rule, decision, and correction for cross-ticket consistency.

**Load `references/delivery.md` before the final regeneration pass.** It holds the whole phase:

- 4.1 Build a Regeneration Plan
- 4.2 Resolve All Open Questions First
- 4.3 Regenerate in Sequence
- 4.4 Platform-Specific Formatting Pass
- Delivery Format Decision Tree
- 4.5 Editorial Pass
- 4.6 Cross-Ticket Consistency Checks
- 4.7 Package for Delivery
- 4.8 Wrike Delivery — XLS Skeleton + HTML Paste

---

## Phase 5: Post-Deployment Verification

**Goal:** After the developer has deployed the schema, confirm the live output matches the spec — with particular attention to multilingual coverage and numeric-type correctness.

**Load `references/post-deployment-verification.md` before verifying deployed schema, confirming a ticket as done, or diagnosing a Search Console error.** It holds the whole phase:

- Phase preamble — scope note (reported-fix verification goes to `jsonld-validation-audit`), the two triggers for a full-population sweep, and the sitewide-ticket template inventory
- Phase 5 Pre-Flight Checklist (Before Sending the Report)
- 5.1 Cross-Language Verification Pattern
- 5.2 Audit Finding Categories
- 5.3 When to Re-Open the Spec
- 5.4 URL Verification — Pre-Publish Procedure
- 5.5 Source-Value Verification — Mirror Audits Aren't Enough
- 5.6 Reconciling GSC Error Counts with a Live Crawl
- 5.7 Single-Fetch False Greens on Dynamically-Rendered Fields
- 5.8 Diagnosing GSC "Unparsable Structured Data / Incorrect Value Type" Across Same-Page-Type URLs
- 5.9 Coverage Audit from Crawl Exports

---

## Phase 6: Measuring Impact

**Goal:** When the client asks what the structured data achieved, establish
*which channel the effect could appear in* before measuring anything — and
write that expectation into the impact ticket before the analysis starts.

**Load `references/measuring-impact.md` before any impact analysis is planned.** It holds the whole phase:

- 6.1 First Call — the Search Appearance Breakdown
- 6.2 State the Expectation Before the Analysis
- 6.3 The Measurable Channels That Remain
- 6.4 Data Retention — Pull the Export Now

---

## Anti-Patterns — What to Avoid

These mistakes have been identified across projects. Avoid them:

1. **Drafting tickets before all input is available** — The biggest time sink. Leads to fictional data and rework. Capture real data in Phase 1 instead.
2. **Regenerating after each feedback item** — Collect all feedback first, regenerate once.
3. **Including "why" explanations in tickets** — Developers need "what." Reasoning belongs in analysis documents, not tickets.
4. **Coverage matrices and language tables** — Scope creep. Stick to implementation-relevant information.
5. **Over-commenting code examples** — Only comment dynamic values and non-obvious references. Self-explanatory fields need no comments.
6. **Including data that requires NLP parsing** — If extracting a property requires parsing unstructured text, omit it. Stick to structured data sources.
7. **Duplicating shared schemas in child tickets** — Use `@id` references to the parent ticket's entities.
8. **Using "optional" for conditional properties** — Say "Add (when [condition]):" to make it clear the property is required when the condition is met.
9. **Mixing analysis and implementation** — Keep analysis documents separate from tickets. Tickets are pure implementation instructions.
10. **Delegating decisions to developers** — Don't offer alternatives ("if you prefer X or Y"). Choose one during analysis and put it in the ticket.
11. **Engineering workarounds for broken HTML** — If a page has a missing H1, that's a content bug, not a schema edge case. Tickets should assume correct markup.
12. **Dropping page types before checking templates** — A page type with low schema value may share a template with a high-value page type. Check templates before dropping anything.
13. **Using placeholder data** — Every URL, image path, and text value in code examples must be from a real page. No `[path-to-image]` or `"Example description here"`.
14. **Letting subagents guess at data** — Subagents are skilled writers but unreliable
    data sources. Pass complete raw data as input. Verify their output against source
    data before delivery.
15. **Reconstructing Current Implementation from memory** — Current Implementation
    blocks must be verbatim extractions, not reconstructions from summaries. Selective
    omission creates invisible gaps.
16. **Navigating to a "representative" page instead of the exact URL** — When verifying
    a ticket example, go to the URL in the example, not a page you think is similar.
17. **Including internal version numbers in client-facing deliverables** — Internal
    iteration counts (v1, v2, v3...) are for file naming conventions and internal
    tracking only. Client-facing documents should show only a date, not a version
    number. The client doesn't need to know how many internal drafts preceded the
    document.
18. **Pre-filtering source extraction by "expected" patterns during audits** —
    When comparing a schema property against the source container it mirrors
    (footer block, CMS field, breadcrumb UI, etc.), do not filter the source
    extraction at comparison time by the patterns you expect to find. Extract
    every value in the container, compare against the schema, THEN group or
    categorise the matches in the report. Comparison-time filtering encodes the
    audit's assumption into the comparison and silently misses anything
    unexpected — which is usually the thing the audit was meant to find. See
    §5.1 for the explicit rule.
19. **Proposing replacement URLs in audits without verifying them live** — Every
    URL in an audit recommendation is a claim ("this URL is the correct value").
    Unverified URL claims propagate as errors: 404s, wrong canonical forms,
    misattributed profiles. The cost of one browser load per URL is far smaller
    than the cost of shipping a recommendation whose URLs are broken or
    misattributed. Use the §5.4 three-part procedure (HTTP status / canonical
    form / destination identity) for every URL before delivery.
20. **Mirror-only auditing of schema pulled from a source** — Confirming that
    "schema correctly mirrors source" is half the audit. Source values
    themselves must also be verified: every URL resolves to the claimed entity,
    in canonical form, with no broken or misattributed cases. A clean mirror
    of wrong source data is still a Failed finding (it just routes to an
    editorial fix rather than a dev fix). See §5.5.
21. **Putting `@type` on a cross-page reference object** — A reference to an
    entity defined on another page is a bare `{"@id": "..."}`. Adding
    `@type` makes the engine parse it as an incomplete entity declaration
    that fails on missing required fields (e.g. GSC "missing field name").
    References are id-only; declarations carry the properties. See §2.3.
22. **Treating a brand-name title match as `sameAs` identity confirmation** —
    A page title containing the brand name does not confirm the URL points at
    the client's entity; common names have namesakes. Report name-only matches
    as "unconfirmed", never "probably correct". See §5.4.
23. **Concluding "can't reproduce" when a GSC error count exceeds a live
    crawl** — A much higher GSC count than a live crawl reproduces usually
    means a state-dependent field omission (out-of-stock dropping `offers`),
    not stale GSC data. Find the conditional branch. See §5.6.
24. **Classifying a spec-vs-implementation source divergence as Failed
    without triage** — "Wrong source" is a hypothesis, not a conclusion.
    Identify what the implementation IS using, sample same-template pages,
    and test the spec's source on them; if the implementation's source is
    structurally better (more available, more descriptive, more
    consistent), the spec is wrong, not the implementation. See §5.2.
25. **Reporting shared-code defects per-template** — Serialiser-level bugs
    propagate by code path, not by template. Per-template write-ups guide
    the dev team toward N per-template fixes when one shared-code fix is
    what's wanted. Broaden the sample (5–10 page types on the same path)
    and state the scope as the affected code path. See §5.2.
26. **Hard character truncation with a literal `"..."` suffix in content
    properties** — A common CMS serialiser default. Mid-word cuts read as
    corrupted data to both rich-results parsers and LLM consumers. Spec
    explicit length rules (full `articleBody`, full first paragraph for
    `description`) so the default never applies. See §3.7.
27. **Inferring a fact from a missing field** — "No location in the
    dataLayer, so the event is online" reads an absence as if it encoded
    the value the rule wants. An empty field means unknown; the fact, if
    the estate distinguishes it at all, lives somewhere the source does
    not carry (a rendered label, a separate flag). Find a live case where
    the field is empty and look at the page before keying a rule to it.
    See Rule 39c.
28. **Writing a drop / filter / exclude rule without enumerating what it
    removes** — A filter that reads as obviously safe ("only absolute
    http(s) URLs") is a claim about the distribution of live values, and
    the most common bad value is often repairable rather than junk. Run
    the rule against live data and confirm every removal is intended
    before it goes in the ticket. See Rule 39c.

---

## Session Management

This project may span multiple sessions. Claude should proactively ensure work isn't lost between sessions and guide the user on how to maintain continuity using whatever tools, features, and capabilities are available at the time.

Core principles:
- **Save progress incrementally** — Don't let large amounts of work exist only in conversation context.
- **Anticipate context limitations** — If the conversation is growing large or approaching limits, proactively guide the user on how to preserve progress and continue effectively.
- **Session summaries** — At the end of each session, create a summary that captures: what was accomplished, critical issues discovered, accumulated changes for final regeneration, the complete rules list, next session action items, and open questions.
- **Single source of truth** — Project files should serve as the authoritative record between sessions. Claude should read relevant project files at the start of a new session before doing any work.

---

## Required User Input — Checklist

For a successful project, the user must provide:

### Phase 0: Setup
- [ ] Browser access enabled (recommended) or willingness to provide rendered HTML manually
- [ ] Ticket structure template (if the organisation has one)
- [ ] Preferred greeting and tone for tickets

### Phase 1: Discovery
- [ ] Review and confirm the page type catalog (Claude provides the draft when browser access is available)
- [ ] For page types where browser access is unavailable: rendered HTML files (Inspect → copy `<html>` element → save as `.txt` or `.html`)
- [ ] If applicable: access to staging environments for unreleased page types

### Phase 2: Schema Design
- [ ] Project-specific rules (privacy, terminology, brand, other constraints)
- [ ] Confirmation of organizational hierarchy (e.g., global brand → national entities → local offices)
- [ ] Approval of proposed schema types for each page type

### Phase 3: Ticket Creation
- [ ] Feedback on each ticket draft (collected in batches, not one-at-a-time)
- [ ] Resolution of open questions before final regeneration
- [ ] For bug/inconsistency tickets: confirmation of whether the issue is already known

### Phase 4: Regeneration
- [ ] Approval of regeneration plan
- [ ] Resolution of all open questions
- [ ] Final review of regenerated tickets

### Phase 6: Measuring Impact (if commissioned)
- [ ] Search Console access (Performance report with Search appearance breakdown; BigQuery bulk export status)
- [ ] Deployment date per page type (from the developer or the ticket history)
- [ ] Sign-off on the §6.2 expectation statement before the analysis starts
- [ ] If AI-visibility data is to be used: confirmation that tracking predates deployment

---

## Feedback on this skill

If the user has methodology feedback, offer to draft a report for
https://github.com/rebelytics/schema-implementation; before drafting, search
that repository's open and closed issues and pull requests for the same
problem, and read its CONTRIBUTING file for how the maintainer takes reports.
If the problem is the agent not following the skill's rules, acknowledge and
correct it instead.

Schema Implementation: Eoghan Henn and contributors | CC BY 4.0
