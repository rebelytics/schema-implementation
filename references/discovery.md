# Phase 1: Discovery & Page Type Catalog — reference for the schema-implementation skill

Load triggers: building or reviewing the page type catalog; analysing a page type's HTML and existing schema; saving real data for ticket code examples

Reference file for the `schema-implementation` skill (Eoghan Henn and contributors, CC BY 4.0). The core `SKILL.md` — the phase overview, §2.8 Universal Technical Requirements, the Anti-Patterns list and the Required User Input checklist — applies alongside this file. Section numbers (§1.2, §5.4) and rule numbers (Rule 35) are the same in every file; the core's section map says which file holds each phase.

## Contents

- Phase 1: Discovery & Page Type Catalog
  - 1.1 Gather the Page Type List
  - 1.1b Priority Assessment / Triage
  - 1.1c Verify Google's Current Guidance
  - 1.2 Analyze Each Page Type
  - 1.3 Save Real Data for Ticket Code Examples
  - 1.4 Document Findings
  - 1.5 Identify Consolidation Opportunities
  - 1.6 Classify Each Page Type

## Phase 1: Discovery & Page Type Catalog

**Goal:** Build a complete inventory of all page types on the website, with detection signals, existing schema coverage, and saved real data for ticket code examples.

### 1.1 Gather the Page Type List

**Primary approach (with browser access or web search):** Claude proactively offers to browse the site or use web search to produce a first-pass page type catalog. This includes reviewing the site's navigation, sitemap, URL structure, and any discoverable page types. Present the draft catalog to the user for review and correction.

**Fallback approach:** Ask the user to provide a list of all distinct page types (e.g., home page, team member profile, article, event, job posting, service page, etc.). If the user doesn't have a complete list, work with them to build one by reviewing the site's navigation, sitemap, or CMS structure.

For both approaches, also ask the user if there are page types planned but not yet live, and whether they want to include these.

**Collaborative framing:** Let the user know what you can do independently and what would help you do a better job — e.g., "Here's what I found. Could you review this list and add any page types I may have missed? If you can also share sitemap access or a CMS export, that would help me be thorough."

**Re-enumerate "all entities on the page" — don't inherit a prior count.** When
a schema must include "all X visible on the page" (all people in a leadership
section, all offices, all cards) and that schema is being moved or migrated
from a previous ticket, treat the prior ticket's entity count as an **anchor,
not ground truth** — re-enumerate the full set from the live page every time. A
migrated schema inherits its predecessor's stale assumptions, including a stale
entity count. Two traps to avoid when enumerating repeated entities:

- **Do not enumerate via heading tags alone.** Sub-section labels and entries
  are often non-heading elements (a person's name in a card, a role label in a
  `<div>`), so an `h2`/`h3` sweep undercounts. Count the repeated card/person
  structures in the container directly, then cross-check with a role/label
  occurrence sweep (e.g. `html.count('Director')`) before claiming the set
  is complete.
- **Watch for dynamically-hydrated sections.** Sections hydrated client-side sit
  behind a "Loading…" placeholder in the server HTML, so a raw-source count
  misses them entirely. Confirm whether the container is server-rendered before
  trusting a source-based count.

Principle: completeness must be re-established against the live DOM via repeated
structures plus a label sweep — never via headings alone, and never carried
over from the source ticket.

### 1.1b Priority Assessment / Triage

Before diving into detailed analysis of every page type, assess schema potential and classify each one:

| Tier | Description | Action |
|------|-------------|--------|
| **Implement** | High schema potential, clear `@type`, rich structured data available | Full analysis (3–5 examples) |
| **Optional** | Moderate potential, unclear value | Analyze 1–2 examples to decide |
| **Check template** | Low standalone schema value, but may share a template with an Implement page | Analyze 1 example to check template sharing |

**Critical rule:** Do not drop any page type before checking its HTML template. A page type with low standalone schema value but a shared template with an Implement page is a free win — it can be folded into the same ticket at near-zero additional development cost. The actual drop decision only happens after Phase 1.2 analysis, when template sharing is visible.

**Honest granularity rule:** The page type catalog's granularity must reflect the actual depth of research. If a page type was only checked with one example, don't split it into multiple rows that imply individual analysis. Group honestly (e.g., "Other pages (customer service, informational, etc.)") rather than creating separate categories based on assumptions. False precision in the catalog cascades into false precision in tickets.

### 1.1c Verify Google's Current Guidance

For each schema type in the implementation plan, check Google's Search Developer documentation and Search Gallery to confirm:
- Recommended scope (all pages vs. specific page types vs. homepage only)
- Current feature support status (active, deprecated, beta)
- Any deprecations or changes in the last 12 months

Document findings: "Google guidance for [SchemaType]: [scope], [features], [notes on recent changes]."

If user-provided scope differs from Google guidance, flag it: "Your requirement specifies [SchemaType] on all pages, but Google recommends [scope]. Proceeding with [recommendation] — confirm if you need a different approach."

This step catches outdated assumptions early. For example, the sitelinks search box feature for WebSite schema was deprecated in November 2024, and Google recommends WebSite schema on the homepage only.

**Cross-check against the dated changelog, not just the docs page (mandatory).** When verifying current guidance, also check the Search Central documentation updates changelog (https://developers.google.com/search/updates). Fetched docs pages are cache snapshots: `web_fetch` has been observed serving a week-stale merchant listing page that lacked a newly added property (`Product.category`) which a raw desktop-UA sandbox fetch of the same URL did show. The changelog is dated and authoritative — it catches both stale fetches and changes since this skill was last updated. Corollary: never deliver a "this change/property doesn't exist" claim about recently updated documentation from a single fetch channel. Treat channel staleness as the first hypothesis and re-fetch via a second channel (sandbox raw fetch with a desktop UA, or browser) before concluding the change doesn't exist. The same rule applies to any vendor's guidance: prefer the vendor's dated changelog/announcements channel as the second source.

### 1.2 Analyze Each Page Type

**Primary workflow (with browser access):** Claude navigates to example pages and captures rendered HTML directly using JavaScript (e.g., `document.documentElement.outerHTML`). The user confirms which pages to analyze. This is massively faster than manual HTML upload.

**Fallback workflow (without browser access):** Ask the user to provide rendered HTML for each page type. For each example page, open the browser's developer tools (Inspect), right-click the `<html>` element, select "Copy element" (or "Copy outer HTML"), paste into a `.txt` or `.html` file, and upload it. This captures the full rendered HTML including any dynamically loaded content.

For each page type, extract and document:

**Detection signals (how to identify this page type programmatically):**
- Body class (e.g., `content-article`, `content-event`)
- `dataLayer.pageType` or equivalent analytics layer value
- URL pattern (e.g., `/[lang]/[country]/articles/[slug]`)
- Hero/template CSS classes or other distinguishing HTML patterns

**Existing schema coverage (mandatory multi-format check):**

Before scoping any ticket, check for existing structured data in **ALL THREE
formats**. Missing existing markup leads to incorrect scoping (greenfield vs
migration) and costly rework.

1. **JSON-LD** — Search for `<script type="application/ld+json">` in the page source.
2. **Microdata** — Search for `itemscope`, `itemtype`, and `itemprop` attributes in
   the HTML. Many CMS platforms (Shopware, WordPress, etc.) emit Microdata by default,
   even when no JSON-LD has been implemented. Common default Microdata includes
   BreadcrumbList on navigation pages and Product with Brand/Offer on product pages.
3. **RDFa** — Search for `typeof`, `property`, and `vocab` attributes.

For each format found, document: what `@type` or schema type is present, which
properties are populated, and on which page types. If existing markup is found in
any format, frame the ticket as a **migration** (to JSON-LD) with explicit removal
instructions for the old format — not as a greenfield implementation.

- Always recommend JSON-LD as the target implementation format, regardless of what currently exists.
- **Extract existing schema verbatim.** When inspecting current schema, extract the
  `@context`, `@type`, and all properties exactly as they appear — do not assume or
  "correct" values. Record `@context` as-is (e.g., if the site already uses
  `https://schema.org`, don't flag it for an http→https update).
- For Organization schema: verify all `sameAs` URLs by navigating to each one. Check: (a) URL still resolves (no 404), (b) platform rebrands (e.g., twitter.com → x.com), (c) URL normalization (www vs non-www). Also check the site footer for social links not present in the current schema.
- **PostalAddress geographic consistency check.** For any PostalAddress
  entity, verify that `addressLocality`, `addressRegion`, and `addressCountry`
  are geographically consistent (e.g., the locality is actually in the
  stated region). This catches errors that purely on-page verification
  misses — `addressRegion` in particular is easy to miss because it's not
  visually verifiable on the page itself, it requires geographic knowledge
  or a quick lookup. Example: a ticket kept
  `addressRegion: "Hamburg"` for a Lüneburg-based company despite
  Lüneburg being in Lower Saxony. Data accuracy checks must cover
  internal consistency between related fields, not just whether each
  field individually matches a page element.

**Conditional schema triage.** Some schema types (VideoObject, HowTo, Review,
Event, FAQPage) appear only on pages matching specific content criteria —
e.g., VideoObject on pages with embedded video, FAQPage on pages with a
Q&A block. These conditional schemas are easy to note in passing during
audit without actually auditing their correctness or coverage, which
creates an "orphaned mention" pattern where the schema appears across
multiple tickets but no one owns it. For each conditional schema
encountered, make an explicit triage decision and record it:

- **(a) Audit it now** — inspect structure, validate against current Google
  guidance, check coverage across the page types where it could appear.
  Treat the result as a first-class audit finding.
- **(b) Defer to Phase 2** — flag the schema as "not audited" with the
  page types where it was spotted, and revisit during schema design.
  **A defer disposition creates its carrier at the moment it is written:**
  one line in the project's own backlog naming the schema type, the page
  types and what the follow-up checks. A deferral written only into a
  deliverable is addressed to the client's developers, not to the author,
  so "follow-up" there means nobody — the failure shape is a deferred audit
  that never runs while the deferred schema contradicts the architecture
  the same ticket set introduced.
- **(c) Declare out of scope** — explicitly note that the schema will not
  be addressed in this engagement, with a one-line rationale.

"Noted in passing" is not a disposition; every schema type encountered
during the audit needs an explicit audit / defer / out-of-scope decision.
Without this, conditional schemas drift into multiple child tickets as
casual mentions and the parent ticket can't say whether they need work.

**Cheapest check for a deferred conditional schema.** When a ticket
introduces an `@id` architecture, grep every other JSON-LD block on the
sampled pages for inline entities of the same type — a `publisher`,
`brand` or `seller` that defines its own Organization instead of
referencing the `@id`. The conflict is visible in one pass, before the
deferred audit runs.

**Available data sources:**
- dataLayer contents (all relevant fields)
- HTML elements with structured, easily extractable data (headings, metadata, cards, lists)
- `<meta>` tags (description, canonical URL, hreflang)
- `<title>` tag content

**Content structure:**
- What sections appear on the page (hero, key contacts, description, related content, etc.)
- Which sections are always present vs. conditional
- Author/contributor patterns

**Template grouping:** Identify which page types share HTML templates. This informs consolidation in Phase 1.5.

**Locale delivery mechanism.** For multilingual or multi-regional sites,
explicitly identify how locales are delivered: ccTLDs (e.g.,
`example.de`, `example.fr`), subdomains (e.g., `de.example.com`),
subdirectory paths (e.g., `example.com/de/`, `example.com/uk/`), or a
mix. Don't assume one mechanism — check the URL patterns and the
breadcrumb/nav structure on a few representative pages.

The mechanism shapes how many "instances" of each schema entity exist:

- **ccTLDs** — naturally get their own `@id` (different domain). One
  Organization, BreadcrumbList root, WebSite, etc., per ccTLD.
- **Subdomains** — typically treated like ccTLDs (different host).
- **Subdirectory locales** — same domain, path-based routing. They
  only get their own `@id` if you explicitly decide they should.
  When subdirectory locales function as independent storefronts —
  own homepage, own breadcrumb root, own currency, own nav — they
  warrant the same per-locale treatment as ccTLDs (own Organization
  `@id`, own WebSite, own BreadcrumbList root). Document this
  decision in the parent ticket Notes (see Phase 3.4).

Subdirectory locales on the same domain are the easiest case to miss
because the URL structure looks like a path inside a single site, but
behaviourally they're separate storefronts. Identify them in Phase 1
so they're not handled as an afterthought in Phase 3.

### 1.3 Save Real Data for Ticket Code Examples

During analysis, extract and save concrete data from each page type's best example — the page with the most conditional sections populated. This data will be used directly in ticket code examples during Phase 3.

Save for each page type:
- Page title, H1, meta description, canonical URL
- dataLayer contents
- Names, URLs, and other structured data from key sections (contacts, categories, related items, etc.)
- Hero image URLs (real, full paths — never placeholders)
- Any existing schema code blocks — **verbatim, complete copies** of the live JSON-LD
- **Image and media dimensions** — for any image, video, or media field, verify the
  actual dimensions of the live asset by navigating to the URL and checking
  `element.naturalWidth` and `element.naturalHeight` via JavaScript. Record as
  "Image: [URL], dimensions [W]x[H] (verified)." This catches errors that exist
  in the current schema (e.g., swapped width/height). When the workflow is
  sandbox-based rather than browser-based, verify dimensions browserlessly by
  fetching the asset bytes and reading them from the file header — see the
  browserless snippet under Phase 5. Match the verification method to the
  channel the rest of the work uses: a browser-only technique becomes a skipped
  check in a sandbox workflow.

This eliminates the need to re-request example pages later and ensures tickets always use real data.

**Browser content filter — fallback for blocked extractions.** The Claude
in Chrome browser tools have a content filter that may block extraction of
structured data containing personal or location information (street
addresses, phone numbers, geo coordinates). This most often affects
LocalBusiness schema on retail or service-location pages. All of the
typical extraction approaches — full JSON stringify, individual field
extraction, key-value pairs — can be blocked as a single `[BLOCKED]`
response.

When this happens:
1. Note which fields are blocked (usually address, phone, coordinates).
2. Attempt extraction of the non-blocked fields individually (e.g.,
   `name`, `branchCode`, `openingHoursSpecification` day/time blocks if
   present without the address, `@type`, `url`).
3. For enhancement tickets whose only change is structural (adding a
   property, updating `@context`, adding an `@id` reference), a
   "keep existing values" approach is acceptable — the ticket can
   describe the structural change without reproducing every current
   value. Mark the Current Implementation block with a note:
   `// Values not reproduced — blocked by browser content filter during extraction. Keep existing values unless specified below.`
4. For greenfield tickets that need the real data, ask the user to
   paste the raw schema block from their HTML or from devtools.

Partial extraction is acceptable. The quality bar is "the developer can
implement without guessing," not "every value is reproduced in the
ticket."

### 1.4 Document Findings

Create a summary document for each page type. Structure:

```markdown
# [Page Type] Analysis

**Date:** [date]
**Examples Analyzed:** [count]
**Status:** Documented

## Detection Signals
- Body class: `[class]`
- dataLayer pageType: `'[value]'`
- URL pattern: `[pattern]`

## Existing Schema
[Current implementation or "None"]

## Enhancement/Creation Opportunities
[What should be added or changed]

## DataLayer Content
[Relevant dataLayer fields]

## Key Patterns
[Variations across examples, conditional sections, etc.]

## Saved Real Data (for ticket code examples)
[Extracted data from the most complete example page]
```

### 1.5 Identify Consolidation Opportunities

After analyzing all page types, look for page types that can share a single ticket. Consolidation criteria:
- Same schema `@type` (e.g., multiple page types all using Article)
- Same properties needed
- Same enhancement approach

**Key principle:** Page types that need the same schema `@type` and similar properties should be consolidated into a single ticket, even if they use different HTML templates. The ticket should document detection logic for each template variant, but the schema implementation is shared. Consolidation is driven by schema output, not template input. If the JSON-LD is structurally identical, it's one ticket — regardless of how many templates need to produce it.

Example: A project may find that blog posts, knowledge center articles, newsroom articles, and focus area pages all use identical Article schema — these can be consolidated into a single ticket covering multiple page types with different detection rules.

**Explicit merge / keep-separate criteria.** When evaluating whether to
merge multiple page types into a single ticket:

**Merge when:**
- Target `@type` is identical (e.g., Article on both `cms_page` and
  `blog_post` templates).
- The Complete Implementation is structurally identical — same
  properties, same `@id` references, same multilingual treatment.
- Differences are limited to current-state bugs (different existing
  markup per template) and detection (different body classes, different
  template names).

**Keep separate when:**
- Target structures genuinely differ (e.g., different subset of
  properties, different author types that can't be unified, different
  `mainEntity` structure).
- Migration paths differ enough that the ticket would need two parallel
  Required Changes sections rather than a unified one.

**Merged ticket structure.** When merging, the ticket has:
- A **Detection** section listing each template's distinguishing
  signals (body classes, dataLayer values, URL patterns) so the
  developer can implement the same target schema across all variants.
- A **Template-Specific Notes** section documenting per-template bugs
  or migration steps where they differ.
- A single Complete Implementation block — because the target is the
  same.

A ticket's primary job is to describe the target state. If the target is
identical, separate tickets create redundancy and force the developer to
cross-reference two documents that describe the same end state.
Implementation differences (different templates, different bugs) are
notes within the merged ticket, not reasons for a separate ticket.

### 1.6 Classify Each Page Type

Assign each page type to one of three categories:

| Category | Description | Ticket Type |
|----------|-------------|-------------|
| **No changes needed** | Schema is complete and correct | No ticket (note in parent) |
| **Enhancement** | Schema exists but needs additions/fixes | Enhancement ticket |
| **Creation** | No schema exists, needs full implementation | Creation ticket |
