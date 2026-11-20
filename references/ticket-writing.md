# Phase 3: Ticket Creation — reference for the schema-implementation skill

Load triggers: drafting, revising or checking a parent or child ticket; briefing a subagent to draft or verify tickets; a recurring schema decision (people properties, name and description sources, articleBody, Product category, additionalProperty, BreadcrumbList)

Reference file for the `schema-implementation` skill (Eoghan Henn and contributors, CC BY 4.0). The core `SKILL.md` — the phase overview, §2.8 Universal Technical Requirements, the Anti-Patterns list and the Required User Input checklist — applies alongside this file. Section numbers (§1.2, §5.4) and rule numbers (Rule 35) are the same in every file; the core's section map says which file holds each phase.

## Contents

- Phase 3: Ticket Creation
  - 3.1 Pre-Flight Check
  - 3.2 Ticket Structure Template
  - 3.2b Subagent Usage Guidelines
  - 3.3 Ticket-Writing Rules
    - Tone & Communication
    - Technical Requirements
    - Ticket Structure
    - Content Philosophy
    - Current Implementation Blocks
    - Notes Rules
    - Code Examples
    - Workflow
  - 3.4 Writing the Parent Ticket
  - 3.5 Pre-Flight Checklist (Before Delivery)
  - 3.6 Feedback & Iteration Workflow
  - 3.7 Common Schema Decisions

## Phase 3: Ticket Creation

**Goal:** Produce developer-ready tickets — one parent ticket for shared schemas, one child ticket per page type (or consolidated group).

### 3.1 Pre-Flight Check

Before creating any ticket draft, confirm that:
1. Real data was captured during Phase 1 analysis — ready for use in code examples
2. Project-specific rules are documented
3. Schema design is confirmed (which `@type` and properties for each page type)
4. Ticket structure preferences are known (user's template or skill default)
5. **For bug/inconsistency tickets:** Ask the user "Is this a known issue? Has the
   dev team investigated this before?" This is especially important for issues with
   multiple possible root causes (CMS, GTM, consent management, caching) where
   previous investigation may have already narrowed down the cause. The answer
   changes the ticket's framing, priority, and sometimes whether it should be
   written at all.

As a general principle: do not create drafts until all required user input is available, unless seeing a draft would help the user provide better feedback or input.

### 3.2 Ticket Structure Template

Every child ticket must follow this exact structure (unless the user provided their own template in Phase 0):

```markdown
# [Page Type Name] — [Schema Action] [Schema Type]

[Greeting]

[1-2 sentence intro — what this ticket covers.]

**Detection:** [Body class and/or dataLayer condition and/or URL pattern]

---

## Current Implementation

[Either "Not currently implemented." or a location statement (e.g., "Present on: product detail pages only. Not present on homepage, category pages, or content pages.") followed by existing schema code block]

---

## Required Changes

- **Keep:** [properties to preserve, if enhancement ticket]
- **Update:** [properties to modify]
- **Add:** [always-required new properties]
- **Add (when [condition]):** [conditional properties]

---

## Complete Implementation

[Full code example with <script> wrapper, real data, inline // comments.
The JSON object MUST open with `"@context": "https://schema.org"` as
the first property, then `"@type"`, then the rest. The Placement section
showing `@context` is not a substitute — developers may copy from this
section directly, so the structural requirement must be present here.]

---

## Notes

[Bulleted factual statements. DataLayer disclaimer when applicable.]

---

## Placement

[One sentence naming the location observed in View Source on the sampled
pages — never a default (Rule 8b). New block: "Place this schema in a
separate `<script type="application/ld+json">` tag in the `<head>` section,
alongside the Organization and BreadcrumbList schemas from the parent
ticket." — when that is where the parent's blocks were observed. Change to
an existing block: "Please keep it where it is — in its existing script tag
in the body, next to the video embed."]

**Critical requirement:** The schema must be present in the HTML returned
from the server (visible in "View Page Source"), not dynamically injected
by JavaScript.
```

### 3.2b Subagent Usage Guidelines

Subagents (Task tool) can dramatically speed up ticket creation by drafting
multiple tickets in parallel. The key principle: use subagents for speed, but
never let them guess at data values.

**Safe subagent tasks** (all required information can be passed in the prompt):
- Drafting tickets when complete raw data is provided as input
- Formatting validation and cross-ticket consistency checks
- Pre-flight checklists against rule lists

**Unsafe without safeguards** (require live data the subagent can't access):
- Any task requiring current prices, image dimensions, schema values, or page content
- Drafting Current Implementation blocks (must be verbatim extractions)
- Verifying schema against live pages

**When using subagents for ticket drafting:**
1. The main agent gathers ALL raw data from the live site first: existing schema
   JSON-LD (complete, verbatim), image dimensions (verified via naturalWidth/
   naturalHeight), prices, text content, URLs.
2. Pass the complete raw data to the subagent — not a summary, but the actual
   extracted values.
3. Subagents MUST NOT invent data values. Specifically, if a property value
   was not provided in the task prompt, the subagent must use one of:
   (a) `// keep existing value` for enhancement tickets where the value
   already exists on the page,
   (b) `// [VERIFY: value not confirmed]` for values that need checking,
   or (c) omit the property entirely.
   **NEVER write a plausible-looking value** (street address, phone number,
   geo coordinates, opening hours, additionalProperty names, etc.) that
   wasn't in the input data. Plausible fabrications are more dangerous than
   obvious placeholders — a `[VERIFY]` flag gets caught in review; a
   realistic-looking fake address passes through and reaches the developer
   as gospel. In one demo run, a retail-store subagent
   fabricated street addresses, phone numbers, geo coordinates, opening
   hours, and four `additionalProperty` names that were entirely invented
   and mixed with real values — this is the exact failure mode to prevent.
4. After subagent drafts are returned, run a verification pass checking every
   concrete value against the source data (see Phase 3.5).

**Provide subagents with:**
- The complete ticket structure template (Section 3.2)
- The full ticket-writing rules (Section 3.3)
- A filled example ticket showing exact formatting expectations, especially
  the Notes format
- All raw extracted data for the page type being ticketed

**Anti-placeholder instruction for the Complete Implementation section.**
Subagents default to treating the "Complete Implementation" code block as
a template rather than a concrete example of the structure filled with
real data. Include this in every subagent prompt when a Complete
Implementation section is required:

> The Complete Implementation section must use the EXACT values from the
> real data provided above. Mark dynamic values with `// dynamic per
> [page type]` COMMENTS, but the property VALUES must be real. A developer
> should be able to paste this JSON-LD on the corresponding real page and
> have it validate. Never replace a real URL with a pattern like
> `YYYY/MM/slug`, never replace a real name with `[Service Name]`, never
> replace a real image filename with `[image-hash]`.

The distinction this makes explicit is "show the structure" vs "show a
real instance of the structure" — the latter is always what's wanted.
Without this instruction, subagents will sometimes replace real NewsArticle
URLs with `"YYYY/MM/article-slug/"` patterns even when the full verbatim
source data was provided in the prompt.

**Concrete Notes transforms for subagent prompts.** Rule 36 ("Notes are
data records, not justifications") is too abstract for subagents on its
own — they compress "be factual" into "write what sounds reasonable."
Include 2–3 concrete transform examples in every subagent prompt alongside
Rule 36:

> WRONG: "This aligns with current best practices." → RIGHT: (delete,
> adds no information)
> WRONG: "Schema.org recommends HTTPS for improved security." → RIGHT:
> (delete, the `@context` change itself already says this)
> WRONG: "Improves compatibility with current and emerging search
> platforms." → RIGHT: (delete, evaluative and non-factual)

The transform examples make the distinction visceral in a way that "be
factual" doesn't. Subagents consistently interpret abstract rules more
loosely than concrete transforms; prefer the transforms.

**When running verification subagents**, instruct them to perform a **mechanical property-by-property comparison** between Current Implementation and Required Changes sections, rather than reading for general coherence. The most dangerous errors are silent omissions — properties that exist in the current schema but appear in neither Keep, Update, Add, nor Remove. A holistic "does this look right?" review consistently misses these; only a line-by-line enumeration catches them.

### 3.3 Ticket-Writing Rules

These rules have been refined through iterative feedback across multiple projects. Apply them to every ticket.

#### Tone & Communication
1. **Be polite** — Use "Please" in instructions to developers.
2. **Clear instructions** — The user and Claude handle the research, analysis, and decision-making so that developers can focus on implementation.
3. **No explanatory notes** — No "why not X" justifications or "we considered Y but rejected it."
4. **No vague cross-references** — Put specific instructions where they're needed, don't say "see other ticket."
5. **Neutral descriptors** — Say "present" not "correct" when describing current state.
6. **Direct requirements** — Section header is "Required Changes" not "Recommended Changes."
7. **Notes style** — Write statements of fact ("We do not include contact information — privacy requirement.") not negative instructions.

#### Technical Requirements
8. **Server-side rendering** — Schema must be in initial HTML, not JS-injected. Include this as a critical requirement in every Placement section.
8b. **Placement names the observed location** — The placement sentence is a
    factual claim about the live site, even when it reads like boilerplate:
    it names where the block sits in View Source on the sampled pages ("in
    the `<head>` section" / "in its existing script tag in the body, next to
    the video embed"), never a default copied from a template or from this
    skill. For a change to an existing block, write "please keep it where it
    is" plus the observed location. The failure shape: a publisher-only
    change to an existing video block carried a head-section sentence copied
    from the skill while the block had always been in the body — a developer
    following it literally moves the block, and a verifier checking against
    it flags a deviation that never existed.
9. **Separate script tags** — Each schema type gets its own `<script type="application/ld+json">` tag.
10. **Reference by `@id`** — When an entity has an `@id`, always reference it using `{"@id": "..."}`. Never repeat the full entity. This applies to Organization, LocalBusiness, Place, or any other entity with an `@id`.
11. **Simple terminology** — Say "all pages" when meaning every page on the site. Use "all page types" only when referring to the set of distinct page types.
12. **Language objects** — Always use both `name` (English language name) and `alternateName` (ISO code): `{"@type": "Language", "name": "German", "alternateName": "de"}`.
13. **DataLayer disclaimer** — Include in every ticket whose **code examples contain inline comments referencing `dataLayer.*` values** (e.g., `// identical to dataLayer.office.name`): "The dataLayer is mentioned here for reference only. Please do not make any changes to the current dataLayer implementation." Do not include the disclaimer when the ticket only mentions dataLayer in Notes or Detection sections — the disclaimer exists to prevent developers from misinterpreting code comments as instructions to modify the dataLayer, so it belongs alongside those comments, not elsewhere.
14. **Dual-property patterns** — When two properties serve different purposes with similar data, specify exactly what goes in each.
15. **@context presence and consistency** — Every per-schema Complete
    Implementation block MUST open with `"@context": "https://schema.org"`
    as the first property, even when the bottom Placement section already
    shows it. Tickets contain redundant copies of the same schema across
    the per-schema and Placement sections, and developers may copy from
    either — if the per-schema block omits `@context`, that omission can
    ship to production. The value must also be consistent across all
    tickets: always `https://schema.org` (https, no trailing slash). This
    is a presence rule first, a consistency rule second.

#### Ticket Structure
16. **Parent ticket criteria** — Only schemas that are identical on all pages and standalone entities belong in the parent ticket.
17. **No "Related Tickets" sections** — Each ticket is self-contained.
18. **Current state transparency** — Show exactly what schema exists today.
19. **Logical ordering in Required Changes** — Always: Keep → Update → Add → Add (conditional).
19b. **Exact before→after, not pattern descriptions** — When the audit has
   identified specific values that need changing, the Required Changes
   section must show the exact before→after for the property. Don't
   describe the change generically when the actual values are known.

   **Wrong:** "Update the Twitter URL from `twitter.com` to `x.com`."
   **Right:** "Update `sameAs` Twitter entry from
   `https://twitter.com/examplebrand` to
   `https://x.com/examplebrand`."

   Reserve pattern-level descriptions for cases where multiple dynamic
   values follow the same rule (e.g., "all `sameAs` URLs matching
   `twitter.com/*` should become `x.com/*`"), not as a substitute for
   showing the concrete values where they're already known. The more
   lookup work a ticket eliminates, the faster and more accurately it
   gets implemented.
20. **Focus on implementation** — Remove analysis sections, data source inventories, validation checklists.
21. **Optional vs conditional** — Split "Add:" (always required) from "Add (when [condition]):" (conditionally required). Never use "Add (optional)" — it sounds skippable.
22. **Remove reference material** — No lookup tables, language code mappings, or similar reference data in tickets.

#### Content Philosophy
23. **Spare developers the research** — Tickets provide clear implementation instructions so developers don't have to spend time researching schema.org decisions or best practices. That work has already been done.
24. **Specificity over generality** — Specific instructions in child tickets, not general principles.
24b. **Scope by defect class, not observed population** — For any bug or
    inconsistency ticket, describe the defect as a class ("the template
    emits the source-language slug wherever X applies — please check every
    place X applies") and report the observed instances separately as
    evidence, explicitly labelled non-exhaustive. A scoping observation
    written as helpful orientation is read as a specification: "only
    affects the newer locales" invites a locale-filtered fix, naming one
    component invites a component-filtered fix, and the unlisted instances
    lose their owner the moment the ticket closes — the fix report matches
    the ticket exactly, so the gap closes silently. When the ticket is
    written from sampled crawl or log data, state the sampling method in
    one line; a reader cannot otherwise distinguish an enumeration from an
    inventory. Where a correlation is offered as a scoping heuristic
    (newest locales, one template, one market), say what would falsify it,
    so a developer who finds a counterexample recognises it as in-scope
    rather than as a separate issue.
25. **Modularity** — Each ticket's schema must be implementable independently.
26. **Dynamic value sources** — Use "identical to" when referencing where a value comes from (e.g., "The `headline` is identical to the H1 element"). Don't over-label what developers can clearly see.
27. **Exception handling** — Note exceptions inline in parentheses.
28. **Avoid redundancy** — Don't repeat in Notes what's already clear from code comments.
29. **Include entities with partial data** — If some properties of an entity are easy to extract and others aren't, include the entity with only the easy properties. Don't skip the entire entity.
30. **Variable data handling** — Include a note: "If [property] is not available on any given page, please remove this property (on that page only)."
31. **Data availability** — Only include properties where data exists in structured sources, has a sensible default, or is determinable from page context or related sources the developer can access at render time.

#### Current Implementation Blocks
32. **Verbatim extraction** — Current Implementation blocks must be verbatim copies of
    the live schema JSON-LD, extracted directly from the page (via browser or from the
    user's HTML upload). Light formatting cleanup (indentation, whitespace) is
    acceptable. Selective omission of fields is not.
32b. **Location context** — Current Implementation blocks in the parent ticket must include a one-line location statement before the code block, specifying which page types have this schema and which don't. Example: 'Present on: product detail pages only. Not present on the homepage, category pages, or content pages.' This makes scope gaps immediately visible and provides an audit trail.
33. **Truncation rules** — If a schema is very long (e.g., Product with 15 shipping
    entries), truncation is acceptable but must be explicitly marked:
    `"shippingDetails": [ /* 15 entries — see live page */ ]`
34. **No reconstruction from memory** — Never reconstruct a Current Implementation
    block from a summary or from memory. A reconstructed block makes editorial
    choices about what to include, creating invisible gaps that cascade into
    incomplete Required Changes sections.

#### Notes Rules
35. **Notes specify WHAT, not HOW** — Notes should only clarify the expected schema output or flag genuinely ambiguous data. They should never:
    - **Delegate content decisions** — If a value needs choosing (description text, social profile list), choose it during the analysis phase. Don't pass the decision to the developer.
    - **Suggest implementation approaches** — Specify what the JSON-LD should look like, not where to pull the data from or how to structure the backend. Developers know their codebase. The test: if a Note describes an HTML element, attribute, or DOM traversal path, it's implementation guidance and should be removed. The code example already shows the expected output; the developer knows how to build it.
    - **Assign research tasks** — If something needs investigating (e.g., which social profiles exist), do that investigation before writing the ticket. The ticket should contain answers, not questions.

    **Examples of implementation guidance to remove:** "Derive the ISO code from the `lang` attribute on the `<html>` tag." "Extract each contact's name and profile URL from the key contacts card links." "Some key contacts may link to external domains — please use the actual href from the page." These all tell developers WHERE in the HTML to find data — the code example already shows what the schema should look like.

    This rule extends beyond ticket Notes to **post-deployment audit follow-up comments**: deliver the complete diagnosis, and hand the fix mechanism to the dev team as an open question rather than a prescribed approach — see "Follow-Up Comments to the Dev Team" in §5.2.
36. **Notes are data records, not justifications** — Write factual statements, not
    interpretive claims. Allowed: "Current schema value: X", "Live site value: Y",
    "Mismatch: X ≠ Y, update required", "Change from X to Y per [Requirement]."
    Disallowed: "This should be X because..." (interpretive), "X is better than Y"
    (evaluative), "We need to fix..." (reasoning, not fact).
37. **No "Not In Scope" sections** — Tickets focus on what to implement, not what to skip. Only include scope exclusions when the absence would genuinely confuse the developer (e.g., clarifying that a closely related page type is handled in a separate ticket).
38. **Assume correct HTML** — Schema tickets should assume well-structured HTML. If broken or missing markup is discovered (e.g., missing H1, broken breadcrumbs, malformed dates), flag it to the user as a separate issue — don't write schema workarounds for it.
39. **Verify before writing** — Never write vague conditionals for things that can be checked. "If the search URL adapts per language" is wrong if you can verify it. Check it, then write: "The search URL adapts per language."
39b. **Neutral language for deprecated features** — When including schema for features that have been deprecated by search engines (e.g., SearchAction after Google deprecated Sitelinks Search Box), use neutral descriptive language rather than claiming active search engine benefits. Say what the markup documents structurally (e.g., "SearchAction documents the site's internal search URL structure"), not what it does for SEO. Overselling deprecated features undermines the ticket's credibility.
39c. **Validate a rule against live data in both directions before it goes
    into a ticket** — A spec rule is a claim about data, and an unverified
    claim in a ticket ships as behaviour. Rule 39 covers checking what can
    be observed; this rule covers the two ways a rule fails that are
    invisible from the desk, because each looks reasonable until it is run
    against the pages it will govern.

    - **For any rule keyed to a field being absent or empty** ("when there
      is no X in the dataLayer, set Y"): confirm what that absence actually
      means. Find a live case where the field is empty and check the
      rendered page. An empty field means the value is *unknown*, not that
      it encodes the fact the rule wants. For example, "no location in the
      dataLayer → set `eventAttendanceMode` to `OnlineEventAttendanceMode`"
      marks every event with a missing location as online with nothing
      supporting it, while the page may render an explicit "Online" label
      for the genuinely online ones that the dataLayer never exposes. Absence of a field is evidence
      about the field, never about the world.
    - **For any rule that drops, filters or excludes** ("only emit values
      that are absolute http(s) URLs"): enumerate what it would remove on
      live data and confirm every removal is intended. For example, the filter
      above silently discards profile links that are missing only their
      scheme and are otherwise valid — often the most common bad value, and
      repairable, not junk. A rule that repairs those and skips the rest
      keeps them.
    - **When a ticket sources values from one system** (a dataLayer, an
      API, an export), state in the ticket that the rendered page was
      checked for signals that system does not carry. A single-source spec
      cannot see what its source omits, and the reader cannot tell an
      authored-from-the-dataLayer rule from a verified one unless the
      ticket says which it is.

    The two checks are symmetric: reading meaning into an absence, and not
    looking at what a rule throws away. Both would have shipped.

#### Code Examples
40. **Real data only** — Use data captured during Phase 1 analysis. All property values must be from real pages. Never use placeholder values like `[path-to-image]`.
41. **Inline comments for dynamic values** — Use `//` to mark values that change per page, with their source when non-obvious.
42. **Minimal comments** — Only comment what needs clarification. Don't comment self-explanatory fields.
43. **Inline comment style** — Use "identical to [source]" for field references (e.g., `// identical to H1`, `// identical to meta description`, `// identical to dataLayer.productName`). Use "reference to Organization from parent ticket" for `@id` references in child tickets. In the parent ticket itself, use "references the Organization entity defined above" — never reference the parent ticket from within itself. **Audience terminology**: when the development team uses internal terminology that differs from on-page labels (e.g., they call it "product families" internally while the page heading says "Collections"), prefer the team's language in inline comments — the comment's purpose is to help developers find the data source in their codebase, not to describe what the end user sees.

#### Workflow
44. **Collect all feedback before regenerating** — When receiving feedback on a ticket draft, collect all items. Ask "any more feedback?" until confirmed done, then regenerate once.
45. **Don't draft prematurely** — Do not create drafts until all required user input is available, unless seeing a draft would help the user provide better feedback. This prevents fictional data, rework, and confusion.
46. **Never use replace_all on tickets with verbatim blocks** — When editing tickets that contain both verbatim Current Implementation blocks and target Complete Implementation blocks, never use `replace_all` for content that appears in both (e.g., updating a URL). Edit each section individually. Verbatim blocks are sacred — bulk find-and-replace is incompatible with tickets that contain both 'as-is' and 'to-be' states of the same data.

### 3.4 Writing the Parent Ticket

**Locale Notes coverage.** When the project has multiple locale delivery
mechanisms (ccTLDs, subdomains, subdirectory paths — see Phase 1.2),
the parent ticket Notes must document the schema treatment for each.
ccTLD guidance alone leaves subdirectory locales (e.g., `/uk/`, `/us/`)
ambiguous, and the developer will default to using the root domain's
`@id` — which is wrong if those subdirectories function as independent
storefronts. The Notes section should have an explicit placeholder for
each locale mechanism present, e.g.:

> **Country / locale versions** — On country versions implemented as
> ccTLDs (e.g., `examplebrand.de`), use the local domain in the
> Organization `@id` and `url`, and treat the local home as the
> BreadcrumbList root. On subdirectory locales on the `.com` (e.g.,
> `/uk/`, `/us/`), the same per-locale treatment applies — own
> Organization `@id`, own WebSite, own BreadcrumbList root — because
> these subdirectories function as independent storefronts.

The parent ticket structure differs from child tickets:

```markdown
# Shared Schemas — Global Implementation

[Greeting]

This ticket contains schema.org markup that applies identically across
all [website] pages. Please implement these once globally — every page
uses the same structure.

Page-specific schemas are covered in separate tickets.

## Page Type Overview

[Table listing all page types, their shared schemas from this ticket, and whether
they have page-specific schemas in separate child tickets. The "page-specific
schemas in separate ticket" annotation should appear on EVERY row that has (or
could have) page-specific schema — not just the rows where child tickets exist
in the current scope. This makes the full picture visible even when only a subset
of child tickets is being delivered.]

## Schemas Included

1. **Organization** — Global [brand] (all pages)
2. **BreadcrumbList** — Breadcrumb navigation (all pages with visible breadcrumbs)

---

## 1. Organization

[Current Implementation → Required Changes → Complete Implementation → Notes]

---

## 2. BreadcrumbList

[Current Implementation → Required Changes → Complete Implementation → Notes]

---

## Placement

[Full example showing schemas in separate script tags for all page scenarios:
home pages, pages with breadcrumbs, and pages without breadcrumbs]
```

### 3.5 Pre-Flight Checklist (Before Delivery)

After all tickets are drafted but before presenting them to the user, re-read the ticket-writing rules and verify each ticket against them. For each property in each ticket, confirm:

1. Is the data available in a structured source? (Rule 31)
2. Does it require parsing unstructured text? (Anti-pattern 6)
3. Are there vague conditionals that could be verified? (Rule 39)
4. Are there implementation suggestions that overstep the ticket's role? (Rule 35) — specifically, do any Notes tell the developer WHERE in the HTML to find a value?
5. Are there decisions being delegated to the developer? (Rule 35)
6. Are all data values from real pages, with no placeholders? (Rule 40)
7. Are inline comments context-aware? (Rule 43)
8. **`@context` presence and value (Rule 15)** — For each ticket, enumerate
   every per-schema Complete Implementation block AND the Placement section
   code example. Confirm each one opens with `"@context": "https://schema.org"`
   as the first property. Presence first, value (no trailing slash) second.
   Per-schema blocks that rely on the Placement section to "carry" `@context`
   are a known failure mode — a developer copying from the per-schema block
   alone will silently drop `@context` in production. Spot-checking is not
   enough; enumerate every block.
9. Are all Notes sections purely factual, with no interpretive language? (Rule 36)
10. Are all Current Implementation blocks verbatim extractions? (Rule 32)
11. Do the Current Implementation, Required Changes, and Notes sections agree on scope and applicability? (e.g., if Current Implementation says "present on the English home page" but the schema is actually on all language versions, this must be corrected.)
12. **Property-by-property accounting** (Rule 9): For each ticket, enumerate every property in the Current Implementation JSON. Confirm each one appears in exactly one of: Keep, Update, Add, or Remove. Flag any property that appears in none of these lists — silent property drops are the hardest errors to catch and survive multiple review rounds.
13. Is the DataLayer disclaimer included only in tickets whose code examples reference `dataLayer.*` values? (Rule 13)
14. For Organization/parent ticket: have all `sameAs` URLs been verified live using the §5.4 three-part procedure (HTTP status / canonical form / destination identity)? Lists of URLs produced without the three-part check are recommendations the ticket cannot stand behind. Watch in particular for: redirected canonical slugs on LinkedIn (e.g. a legacy company slug redirecting to the current one), country-subdomain canonicals on LinkedIn (`{cc}.linkedin.com` rather than `www.linkedin.com`), non-existent X / Twitter accounts that still return HTTP 200, and misattributed profiles where the URL resolves to a third party rather than the claimed entity.
15. Do Placement scenario intro texts match the actual number and types of schemas shown in each scenario's code example? (e.g., if intro says "three schemas" but only two are shown, fix the intro.)
16. **Numeric-type enumeration** (Rule 7): List every numeric-typed property in the Current Implementation. Confirm each is a JSON number, not a string. Do not spot-check — enumerate. The familiar ones (`ratingValue`) are usually right; the silently-stringified ones (`price`, `offerCount`) are the ones that kill rich results. Check every one.
17. **Multilingual annotation coverage** (Section 2.5b): For multilingual projects, confirm every property in every entity has an explicit canonical / localized / per-language annotation — either as an inline comment or a one-line Multilingual note on the entity. Unannotated properties on multilingual tickets are a spec gap, not a stylistic omission.
18. **Google reference-example comparison.** For every schema type in the ticket, fetch Google's current rich-results reference example for that type (`developers.google.com/search/docs/appearance/structured-data/...`) and diff it property-by-property against the Complete Implementation. For every recommended-but-missing property, decide explicitly: include, defer to a follow-up ticket, or skip — and document the reason inline. The Complete Implementation example is meant to be a rich-results-friendly target, not a validation floor. Common traps this catches: `HowToStep.url` (anchor to the step — requires a matching `id="step-N"` on the rendered element), `HowToStep.name`, `HowToStep.image`; Recipe `video`; Event `performer` / `organizer`; Product `shippingDetails` / `hasMerchantReturnPolicy` for countries where Google requires them. Meeting "required" passes validation; matching the reference example earns rich-result eligibility. Cross-check the reference page against the dated Search Central updates changelog (developers.google.com/search/updates) — fetched doc pages can be cache-stale and miss recently added properties (see §1.1c).
19. **`HowToStep` four-property shape.** When tickets include any `HowToStep`, spec the full set — `@type`, `text`, `name`, `url`, and optional `image` — not just `@type` + `text`. For `name`, avoid Google's anti-pattern of numbered prefixes ("Step 1: Preheat oven"); concise step names sourced from per-step image `alt` text (or a short summary of the step) are cleaner. For `url`, emit a page-anchor fragment (`<page-url>#step-N`) and require a matching `id="step-N"` on the rendered step element in the template. A step without `url` triggers a non-critical GSC Recipe warning post-deployment.
20. **PostalAddress geographic consistency check.** For every `PostalAddress` in the ticket, cross-reference `addressLocality`, `addressRegion`, `postalCode`, and `addressCountry` against a basic geographic source (or a sanity read: is the region field actually a city? does the postal code belong to the named country?). Regional misfiles (e.g., `addressRegion: "Hamburg"` for an address in Lower Saxony) survive review because the text reads fluent — the only catch is explicit cross-reference.
21. **Rule validation in both directions (Rule 39c).** List every rule in the ticket that is keyed to a field being absent or empty, and every rule that drops, filters or excludes values. For each absence rule, confirm a live case where the field is empty was found and the rendered page checked for what the absence actually means. For each exclusion rule, confirm the set of values it would remove on live data was enumerated and every removal is intended. Where the ticket sources values from a single system (dataLayer, API, export), confirm it states that the rendered page was checked for signals that system does not carry. A rule that has not been run against live data is a hypothesis presented as a specification.
22. **Deferred-work carriers.** List every "follow-up", "defer" or "not audited" mention in the ticket set, including deferred conditional schemas (Phase 1.2) and recommended properties deferred under item 18. Confirm each has a line in the project's own backlog. Where the ticket introduces an `@id` architecture, confirm the other JSON-LD blocks on the sampled pages were grepped for inline entities of the same type (Phase 1.2, "Cheapest check for a deferred conditional schema").
23. **Placement claims checked against raw HTML (Rule 8b).** List every sentence in the ticket set that states where a block sits — each Placement section, a Current Implementation location statement, a prose placement sentence written in place of a script-tag wrapper (§4.4, WAF-protected tools). Confirm each was checked against the raw HTML of a sample page (View Source or a shell fetch), not inferred from the template, from the parent ticket or from this skill's examples. For a change to an existing block, confirm the sentence says "keep it where it is" and names the observed location.

**Verification of ticket examples:** When a ticket example contains specific URLs,
navigate to those exact URLs to verify. Do not infer the page or find a
"representative" alternative. Verification means checking what's in the example,
not what you think should be there.

**Cross-ticket reference check:** After any updates to the parent ticket's schema
or scope, scan all child tickets for references to parent schemas (e.g., "alongside
the [ParentSchema] from the parent ticket"). Verify those parent schemas still
apply to the child page type. This catches stale references when parent scope
changes (e.g., WebSite moved from "all pages" to "homepage only").

**Escalation principle:** When the user identifies an issue pattern (e.g., "let's
make sure we always avoid fabricated data"), escalate the verification scope to
cover all related deliverables, not just the immediate instance. User feedback
that identifies a failure pattern should trigger systematic coverage.

### 3.6 Feedback & Iteration Workflow

Aim for 2 versions per ticket: v1 draft + v2 final after feedback. The workflow:

1. **Create v1** with real data saved from Phase 1
2. **Present to user** for review — when moving to each ticket during feedback collection, present its content directly (as a downloadable file link) so the user doesn't have to context-switch
3. **Collect ALL feedback** before making changes (Rule 44)
4. **Regenerate** applying all feedback at once
5. **Extract general rules** from specific feedback — if feedback applies to all tickets, add it to the rules list

Track the version number in the filename: `[platform]-child-ticket-[name]-v[N].md` (e.g., `jira-child-ticket-product-v2.md` or `wrike-child-ticket-product-v2.md`).

**Version numbering discipline:** Every regeneration cycle (collect feedback → apply all changes → regenerate) must increment the version suffix in **all** ticket filenames. If v2 tickets receive feedback and are regenerated, the output is v3 — not an updated v2. After creating a new version, delete the previous version's files from the output folder. Only the latest version should remain, to avoid confusion about which files are current.

### 3.7 Common Schema Decisions

These decisions recur across projects. Use as defaults unless the project requires otherwise.

**Which property to use for people on different page types:**
The property depends on the person's role relative to the content:
- People who **wrote** the content → `author`
- People who **oversaw or curated** the content → `editor`
- People who **referenced or relevant** but didn't create it → `mentions`
- People who **belong to** a service, organization, or team → `member`
- People with no semantic relationship to the page content → exclude from schema

**Choosing between `member` and `employee`:**
Prefer `member` for contact persons — it avoids implying a direct employment relationship, which may not be accurate for people affiliated with an organization who aren't direct employees (freelancers, consultants, board members, etc.).

**People-page → about-page migration (mainEntity demotion + same-person merge
+ @id augmentation):** When a leadership/people section moves onto an
About/company page (the page's primary subject changes from the people to the
organisation), three decisions recur:

- **Demote the people from `mainEntity`; make the Organization the
  `mainEntity`.** Use `AboutPage` with the Organization as `mainEntity`
  (referenced by `@id`), and demote the people from `mainEntity` to the
  Organization's `employee` / `member` array. Do **not** use `ProfilePage` —
  that type is for a page *about* a person, and the page is now about the org.
  The schema's `@type` and `mainEntity` follow the new primary subject.
- **One real person = one Person node, even across multiple roles.** A person
  who appears in more than one role or section is **one** Person node with a
  `jobTitle` **array**, not two nodes that share a name. Entities are keyed by
  identity; multiple roles collapse into an array on a single node.
- **Augment a parent-defined entity by `@id` rather than redefining it.** To
  attach a page-scoped property to an entity defined elsewhere (e.g. adding
  `employee` to the Organization defined in the parent ticket), reference the
  entity by `@id` and add **only** the page-scoped property — never repeat the
  full entity. This is the sanctioned way to attach page-specific data without
  violating the "reference by `@id`, never repeat the entity" rule (Rule 10,
  §2.3). Note: once a person sits inside the Organization's `employee` array, a
  per-person `worksFor` back-reference to that same Organization is redundant —
  omit it.

**Page name sources (in order of preference):**
- H1 heading (most common, best for service pages)
- `<title>` tag (better for pages where H1 is too short or generic)
- Static string (when consistency across all pages matters more than per-page accuracy)

**Description sources (in order of preference):**
- Intro/lead text on the page (`.intro__text`, `p.lead`, etc.)
- `<meta name="description">` tag
- First paragraph of body content

**Organization `description` — special case:** The global Organization entity isn't tied to any single page, so the "visible on the page" rule (2.4, Rule 1) can be difficult to satisfy. If no suitable visible text exists on any page, `llms.txt` (or `/[lang]/llms.txt` for multilingual sites) is an acceptable source. `llms.txt` is site-authored, publicly accessible, and specifically designed to describe the organisation. Note the source in the ticket: "identical to the opening paragraph of the corresponding language version's `llms.txt` file."

**Article-family content properties (`articleBody`, `description`):**
Properties named after content need explicit length and source rules in the
spec. "Do the obvious thing" is too ambiguous — the obvious thing differs
between an SEO-tuned reading (short, truncated, meta-description-shaped) and
a careful schema.org reading (full, structural, content-as-data). A spec
that lists `articleBody` under "Add:" with no qualifier invites the
serialiser's default behaviour, whatever that happens to be. Defaults to
spec explicitly:

- **`articleBody`** — the full body text of the article (schema.org defines
  it as "the actual body of the article"). If a given page has no body
  content (e.g., a download landing page), omit the property entirely —
  never emit an empty string or a fabricated summary. LLM consumers
  increasingly parse `articleBody` to understand article content, so a
  truncated body is functionally useless for AI visibility — which is
  increasingly the point of having Article-family markup at all.
- **`description` on Article / Service / Person and similar** — the first
  paragraph of the descriptive field, in full: no mid-word truncation, no
  literal ellipsis. If no descriptive content exists, omit the property.
- **Anti-pattern — hard character truncation with a literal `"..."`
  suffix.** This is a common default in CMS serialisers and is structurally
  bad for any schema property: mid-word cuts read as corrupted data to both
  rich-results parsers and LLM consumers. The spec must state the length
  rule explicitly so the serialiser default never applies.
- **Duplicate-content note** — schema markup is not indexed as separate
  content. The same body text appearing in the rendered article element and
  in `articleBody` is the expected pattern, not a duplicate-content problem.

**Product `category` — Text vs CategoryCode (added by Google July 2026):**
Google's merchant listing documentation recommends `category` on Product
(mirroring Merchant Center feed attributes). It accepts
Text, CategoryCode, or a mixed array of both:

- **Plain Text** = custom product type, equivalent to the `product_type`
  feed attribute; recommended under 750 characters. Source from the site's
  own category breadcrumb/taxonomy.
- **CategoryCode** = Google Product Category (GPC), equivalent to
  `google_product_category`: `@type: CategoryCode`, `inCodeSet` = the
  Google Product Taxonomy URL (e.g.
  https://www.google.com/basepages/producttype/taxonomy-with-ids.en-US.txt),
  `codeValue` = a GPC ID (e.g. "2271") or full path ("Apparel &
  Accessories > Clothing > Dresses" — `>` separator, each segment must
  contain at least one letter).
- Multiple values are allowed (several GPC codes/paths plus several custom
  product type strings).
- **Sourcing rule:** when the client runs a Merchant Center feed, the
  feed's `google_product_category` mapping is the cheapest data source — a
  near-free win. Only emit GPC when a maintained mapping exists; never
  have developers hand-assign taxonomy IDs.
- Product tickets written before Google added the property won't include `category` — flag
  it in active e-commerce audits.
- Context: Google increasingly mirrors Merchant Center feed attributes
  into schema.org markup (`priceType`, `hasAdultConsideration`,
  `hasCertification`, now `category`). A client's feed spec is a leading
  indicator of upcoming schema properties, and the feed itself is often
  the cheapest source for populating them.

**`additionalProperty` labels are custom, not vocabulary — say so in the
ticket:** when a ticket emits `additionalProperty` on Product/Thing
(especially many entries at once), add a Note stating (a) that the labels
are custom `PropertyValue.name` values chosen by the spec author, not
schema.org properties, and (b) whether `PropertyValue.propertyID` links an
external controlled vocabulary or none is used. Example note text:
"additionalProperty labels are custom PropertyValue.name values, not
schema.org properties. No external controlled vocabulary is linked via
propertyID." Without this, developers and reviewers over-interpret the
labels as schema.org vocabulary — when a ticket uses an escape hatch, the
ticket should surface that it's an escape hatch.

**BreadcrumbList must mirror visible breadcrumb navigation exactly.** Do not add
elements to the BreadcrumbList schema that are not visually present in the
breadcrumb trail on the page. A common mistake is adding a "Home" item as the
first breadcrumb element "even if not visible in the breadcrumbs." If "Home" is
not shown in the visible breadcrumb UI, it must not appear in the schema. A
mismatch between markup and visible UI erodes trust in the structured data.
