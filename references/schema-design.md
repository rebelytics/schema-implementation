# Phase 2: Schema Design — reference for the schema-implementation skill

Load triggers: choosing schema types and properties for a page type; designing @id cross-references; deciding the disposition of deprecated or existing-format markup

Reference file for the `schema-implementation` skill (Eoghan Henn and contributors, CC BY 4.0). The core `SKILL.md` — the phase overview, §2.8 Universal Technical Requirements, the Anti-Patterns list and the Required User Input checklist — applies alongside this file. Section numbers (§1.2, §5.4) and rule numbers (Rule 35) are the same in every file; the core's section map says which file holds each phase.

## Phase 2: Schema Design

**Goal:** Decide which schema types and properties to use for each page type, and how schemas relate to each other across the site.

### 2.1 Design the Global/Shared Schema

Identify schemas that are **identical on every page** of the website. These go into a parent ticket. Typical candidates:

- **Organization** — The brand/company entity, with `@id` for cross-referencing
- **BreadcrumbList** — Breadcrumb navigation, present on all pages with visible breadcrumbs

If the home page has no page-specific schema beyond WebSite + SearchAction (which is a global site-level entity), fold WebSite into the parent ticket rather than creating a separate child ticket. A child ticket for the home page is only needed if it requires additional page-specific schema (e.g., ItemList of featured content).

Parent ticket criteria: The schema must be (a) structurally identical on every page, (b) present on all pages (or a clearly defined subset like "all pages with breadcrumbs"), and (c) a standalone entity that other schemas reference.

### 2.2 Design Page-Specific Schemas

For each page type, choose the appropriate schema `@type` and properties. Decision framework:

**Content pages:** NewsArticle, BlogPosting, Article
**People/profile pages:** ProfilePage with Person as mainEntity
**Event pages:** Event
**Job listings:** JobPosting
**Service pages:** Service
**Office/location pages:** LegalService, LocalBusiness, or Organization
**Product pages:** Product, ProductGroup
**Category/listing pages:** CollectionPage, ItemList
**Hub/landing pages:** WebPage, CollectionPage
**Home pages:** WebSite with SearchAction
**FAQ pages:** FAQPage
**How-to/tutorial pages:** HowTo
**Course pages:** Course
**Recipe pages:** Recipe
**Review pages:** Review
**Informational overview pages:** WebPage

This list is not exhaustive — choose the most specific applicable type from the schema.org vocabulary.

### 2.3 Design Cross-References

Schema entities should reference each other using `@id`. Any entity that has an `@id` should be referenced by `@id` only — never by repeating the full entity. Map out the reference graph:

```
Organization (@id: "https://example.com/#organization")
  ↑ referenced by:
  ├── LocalBusiness.parentOrganization
  ├── Person.worksFor → LocalBusiness.parentOrganization
  ├── NewsArticle.publisher
  ├── Event.organizer
  ├── Service.provider
  └── WebPage.provider
```

This applies to any entity with `@id` — Organization, LocalBusiness, Place, etc.

**A reference is `@id`-only — never `@type` + `@id`.** A cross-page reference
object must be a **bare `{"@id": "..."}`** with **no `@type`**. When you
write `{"@type": "Product", "@id": "<product-url>"}` to reference a Product
defined on a *different* page, the search engine cannot resolve the `@id`
(the entity lives on another URL) and parses the object as an *incomplete
standalone declaration* of that type — which then fails validation on the
type's required properties (e.g. GSC "Product snippets — missing field
name"). `@type` + `@id` only = a malformed entity, not a reference;
declarations carry the properties, references carry just the id. Audit
`mentions`, `about`, `isPartOf`, `seller`, `publisher`, and any other
cross-entity reference array for this pattern — it is most common in
Article/MedicalWebPage `mentions` arrays that point at products.

### 2.4 Deprecation vs Removal

When a schema type or property has been deprecated by a search engine for
rich results (e.g., SearchAction / Sitelinks Search Box deprecated by Google
in November 2024), the default recommendation should be to **KEEP** the
existing markup, not remove it.

**Rationale:**
- Other consumers (Bing, LLM crawlers, future Google features) may still
  parse it
- Removing working schema has a measurable cost (dev time, regression risk)
  with zero upside
- Google occasionally reintroduces or repurposes deprecated signals

**When to recommend removal:** Only when the markup is actively causing a
problem — validation errors generating noise, penalty risk, or contradicting
current page content.

**How to document deprecations in audits:** Frame as context ("this schema
no longer produces a sitelinks search box in Google results as of November
2024"), not as a removal rationale. Location-hygiene recommendations
(e.g., "WebSite on homepage only, not every page") are separate, valid
cleanup items and should not be conflated with deprecation notes.

**Principle:** In audits, "deprecated" is informational, not actionable.
The bar for recommending removal of existing working schema is higher than
the bar for recommending addition of new schema.

### 2.4b Format Migration Tickets — Disposition of the Existing Format

When a migration ticket replaces one structured-data format with another
on the same page (e.g., Microdata → JSON-LD, RDFa → JSON-LD), the
ticket must specify the **disposition of the existing format** when
that format has known search-engine validation errors visible to the
client (Google Search Console "Items with issues" reports, Bing
Webmaster Tools structured-data warnings, Rich Results Test errors,
etc.).

**Rule:** "Removal is optional" is **not a viable disposition** for
existing-format markup that has visible validation errors. Search
engines validate each structured-data format independently — adding a
valid JSON-LD BreadcrumbList alongside a broken Microdata
BreadcrumbList does not silence the Microdata errors. If the dev team
interprets "optional" literally and leaves the broken format in place,
the client continues to see the same GSC warnings indefinitely after
the migration ships, and the work that was just delivered visibly
fails to resolve the problem that motivated it.

**Two viable dispositions for existing-format markup with errors:**

1. **Remove once JSON-LD ships** — the cleanest path. The migration
   ticket explicitly removes the existing format in the same release
   that adds the new format. Use this when there's no business reason
   to keep both (no dependency on the existing format from other
   consumers, no risk of regression).
2. **Fix in place to match the JSON-LD structure** — keep both
   formats but bring the existing format to validity by mirroring the
   JSON-LD. Use this when removal is risky (e.g., the existing format
   is consumed by an internal tool, a legacy integration, or another
   search engine that performs better with the original format).

**What the ticket must say:** state the chosen disposition explicitly
in Required Changes, with the same level of specificity as the JSON-LD
addition itself. For removal: "Remove the existing
`<div itemtype='https://schema.org/BreadcrumbList'>` block on
[page types]." For fix-in-place: show the corrected Microdata/RDFa
markup alongside the new JSON-LD.

**Anti-pattern.** A migration ticket that adds correct JSON-LD,
mentions the existing broken format in Notes ("the existing Microdata
BreadcrumbList is missing the `position` property"), and leaves the
disposition implicit or marked optional. The client will continue to
see the same GSC errors after the release; the ticket appears to
fix the issue but doesn't.

**Where this applies in the workflow:**

- **Phase 1 discovery:** when cataloguing existing schema, also note
  whether the page type's existing format has visible validation
  errors. This becomes an input to the disposition decision.
- **Phase 2 design:** for any migration page type, decide the
  disposition during design, not at ticket-writing time.
- **Phase 3 ticket creation:** the disposition appears as a concrete
  Required Changes item with exact-before→exact-after specificity (per
  rule 19b).

**Principle:** Adding correct structured data in a new format does not
silence search-engine errors on incorrect structured data in the
original format. Migrations driven by existing validation errors must
make the disposition of the original format explicit; the "keep both,
removing the old one is optional" pattern is fine only for clean
migrations where the original format has no errors.

### 2.5 Apply Property Selection Rules

For each property on each schema type, apply these filters:

1. **Data must be visible on the page** — Never include data that isn't shown to users.
2. **Data must be structured or easily extractable** — Available in dataLayer, meta tags, or clearly structured HTML elements. Avoid properties that require natural language parsing of unstructured text.
3. **Include entities with partial data** — If an entity (e.g., Person) has some easily extractable properties (name, url) and some hard-to-extract ones (image from a complex carousel), include the entity with only the easy properties. Don't exclude the entire entity just because one property is hard to get.
4. **Data can come from related sources** — Properties don't have to come from the current page only. Developers may have access to related pages or backend data at render time.
5. **Sensible defaults are acceptable** — e.g., `eventStatus: EventScheduled` for upcoming events.
6. **Omit rather than leave empty** — Never include empty strings, empty arrays, or null values. Remove properties entirely when data is unavailable.
7. **Numeric properties must be JSON numbers, not strings.** Every numeric-typed property in schema.org must be emitted as a JSON number (e.g., `"price": 29.99`), never as a string (`"price": "29.99"`). This rule applies to **all** numeric properties, not only the obvious ones. When auditing or designing schema, enumerate numeric properties explicitly rather than spot-checking the familiar ones (ratings). The commonly-missed ones include:

   - `price`, `lowPrice`, `highPrice` on Offer / AggregateOffer
   - `offerCount` on AggregateOffer
   - `priceSpecification.price`
   - `value` inside QuantitativeValue (`minValue`, `maxValue`, `value`)
   - `ratingValue`, `reviewCount`, `ratingCount`, `bestRating`, `worstRating` on AggregateRating / Rating
   - `numberOfEmployees.value`
   - `weight`, `height`, `width`, `depth` values

   An audit that only checks `aggregateRating` for numeric-vs-string will silently miss stringified `price` fields, which Google ignores entirely. During Phase 2 design and Phase 3.5 verification, list every numeric-typed property in the Current Implementation and confirm its JSON type.

### 2.5b Multilingual Property Values

For multilingual sites, **every property in every schema entity is a spec decision point**: is this value canonical across languages, localized per language, or derived from a CMS field that already resolves per language at render time? Leaving this implicit in the ticket pushes a research task onto the developer and causes post-deployment audits to discover inconsistencies the spec could have prevented.

Annotate each property with one of three tags during Phase 2 design:

- **canonical** — Same value on every language version. Example: `@id` for Organization, `sameAs` URLs, product SKU.
- **localized** — Different value per language version, produced by translation. Example: `description`, `alternateName`, `name` when language-specific, `keywords`.
- **CMS-derived (per-language)** — Different value per language version, already resolved by the CMS / template layer for the page being rendered. Example: `headline` (from the H1), `datePublished` (from the post metadata), `image` (from the hero selector). The developer does not need to do anything special — the existing render path produces the right value for the current language.

**Where the annotation lives in the ticket.** Add a one-line "Multilingual" note for each entity in the Complete Implementation section, or as inline `// canonical` / `// localized` / `// per-language` comments next to individual properties. For properties where the same data source naturally differs per language (H1, meta description), the inline comment can be brief: `// identical to H1 (per-language)`.

**Anti-pattern.** Saying "the schema should be translated" without specifying per property. Translation is a per-property decision — some properties (`name` of a category) should be translated, others (`sku`, `@id`, `sameAs`) must not. An audit that finds a localized `@id` or an untranslated `description` has usually been caused by a spec that skipped the per-property call.

**Verification link.** The Phase 5 post-deployment cross-language verification pattern (see below) depends on this annotation — a boolean presence-and-type audit across language versions can only classify a value as "correctly localized" or "incorrectly canonical" if the spec said which one it should be.

### 2.6 Verify Assumptions Before Finalizing

Before moving to ticket creation, verify every assumption that can be checked:
- Does a URL pattern adapt per language? Check by visiting multiple language versions.
- Does a field exist on all pages of a type, or only some? Check multiple examples.
- Is a date field consistently formatted? Check several pages.

State verified findings as facts in the tickets. Never write vague conditionals ("if X, then Y") for things that can be observed. If something genuinely cannot be verified (e.g., it depends on server-side logic not visible in the rendered HTML), ask the user.

### 2.7 Project-Specific Rules

Document and enforce any project-specific constraints. Common examples:
- Privacy rules (e.g., "no phone/email in schema")
- Terminology rules (e.g., "use 'lawyer' not 'attorney'")
- Brand rules (e.g., "always use the parent brand name, not subsidiary names")


§2.8 Universal Technical Requirements is in the core `SKILL.md`; it applies to every page type designed here.
