# Phase 5: Post-Deployment Verification — reference for the schema-implementation skill

Load triggers: verifying deployed schema against the spec; confirming a ticket as done or answering a client-reported discrepancy; diagnosing a Search Console structured-data error; a coverage audit from a crawl export

Reference file for the `schema-implementation` skill (Eoghan Henn and contributors, CC BY 4.0). The core `SKILL.md` — the phase overview, §2.8 Universal Technical Requirements, the Anti-Patterns list and the Required User Input checklist — applies alongside this file. Section numbers (§1.2, §5.4) and rule numbers (Rule 35) are the same in every file; the core's section map says which file holds each phase.

## Contents

- Phase 5: Post-Deployment Verification
  - Phase 5 Pre-Flight Checklist (Before Sending the Report)
  - 5.1 Cross-Language Verification Pattern
  - 5.2 Audit Finding Categories
    - Source-Divergence Triage — Run Before Classifying as Failed
    - UI-Existence Check — Run Before Classifying Absent Conditional Schema as Failed
    - Per-Finding Routing — Pre-Write Step
    - Broadening Sweep — Shared-Code Findings
    - Report Template Skeleton
    - Follow-Up Comments to the Dev Team — Diagnosis, Not Prescription
  - 5.3 When to Re-Open the Spec
  - 5.4 URL Verification — Pre-Publish Procedure
  - 5.5 Source-Value Verification — Mirror Audits Aren't Enough
  - 5.6 Reconciling GSC Error Counts with a Live Crawl
  - 5.7 Single-Fetch False Greens on Dynamically-Rendered Fields
  - 5.8 Diagnosing GSC "Unparsable Structured Data / Incorrect Value Type" Across Same-Page-Type URLs
  - 5.9 Coverage Audit from Crawl Exports

## Phase 5: Post-Deployment Verification

**Goal:** After the developer has deployed the schema, confirm the live output matches the spec — with particular attention to multilingual coverage and numeric-type correctness.

This phase is optional but strongly recommended whenever a non-trivial implementation reaches production. Fast, targeted verification catches regressions that would otherwise only surface weeks later via Search Console warnings.

**Scope note:** Phase 5 verifies conformance to this skill's own Phase 2 spec. When the task is verifying a *reported fix* across locales rather than conformance to the spec, run a JSON-LD validation audit instead — it needs the cache-busting gate, population enumeration from the sitemap index / live hreflang set, and the positive control that a spec-conformance pass does not.

**Two triggers require the full-population sweep before anything is written.** On a
multi-domain or multi-locale implementation, run §5.1 across every locale and §5.9
across every URL of the page type *first* in both of these cases:

1. **Before confirming a ticket as done on the thread.** A spot-check-based "looks
   great now" after a fix leaves partial rollouts and unspecced additions sitting in
   production for someone else to find. Sweep, then confirm, and state the swept
   scope in the confirmation.
2. **When a client reports a cross-instance discrepancy** — "this property has 12
   entries on one domain and 8 on the others". That observation is a sample of size
   one from an unaudited population. Sweep before replying: the sweep confirms what
   they saw, attributes its cause, and typically surfaces adjacent deviations they
   did not see (a rollout that missed one domain, an unspecced addition present
   everywhere, a later improvement that supersedes the spec — the §5.2
   "spec superseded" category). Answering from the instance the client happened to
   notice concedes the scope of the conversation to whoever noticed it, and turns
   what could be an authoritative reply into a defensive one.

**For a sitewide (parent) ticket, the page-type population is the site's own
template inventory, not the ticket set's.** When the claim under test includes
a parent ticket — entities on all pages — enumerate the page types from every
template the site links from its global navigation and footer, plus the
sitemap index; one fetch of the home page per locale yields the list. Do not
build the sample from the page types the child tickets name plus a handful of
static pages. Every template no child ticket covers is still checked for the
parent's entities and for extra JSON-LD blocks outside the `@graph`. The
failure shape: a template linked from the main navigation (an archive or
community index, say) that no child ticket covers, so nobody samples it,
emitting a legacy block of entities (no `@id`, HTML in `text`, truncated
`text`, null or date-only `dateModified`) that the search console may already
be flagging. Such a block surfaces only on a later verification that
enumerates from the navigation, and by then nobody can tell whether it is
new or was simply never looked at.
The templates no ticket mentions are exactly where unspecced output survives
unseen.

### Phase 5 Pre-Flight Checklist (Before Sending the Report)

Before delivering a post-deployment audit report, verify each of the
following. These are the checks most likely to be missed under the
cognitive load of writing up a long verification pass:

1. **Four-category routing applied (§5.2).** Have all findings been
   classified as Passed / Failed / Exceeded-spec / Spec-superseded before
   write-up? Are Exceeded-spec and Spec-superseded findings pointed at a
   spec update (revise the Phase 2 design document), not at a fix ticket?
   No findings should be sitting in a generic "Medium issues" or
   "deviations" bucket without an explicit category.
2. **Failed and Exceeded-spec are in separate sections.** The report
   structure follows the §5.2 template skeleton — named sections, not a
   single severity ladder. A finding under "Exceeded spec" must have an
   action of "update the spec", not "fix the implementation".
3. **`@context` presence audited.** Live JSON-LD blocks were checked for
   `@context` presence and value (`https://schema.org`, no trailing
   slash) — this is the most common silent omission when developers
   copy from per-schema Complete Implementation blocks rather than the
   Placement section (Rule 15 / Pre-Flight item 8). When the audit
   reveals a missing `@context` in production, that's a Failed finding;
   it's not Exceeded-spec.
4. **Live values spot-checked (§5.1).** The "Live values — spot check"
   subsection includes verbatim live values for fields where
   visible-UI-to-schema textual leakage is plausible.
5. **Spec changes proposed are dated.** Any "Exceeded spec" finding
   that calls for a Phase 2 spec update has a dated entry in the
   "Spec changes proposed" section so the audit trail survives.
6. **Source extraction was unfiltered (§5.1).** For any property that
   mirrors a source container (footer block, CMS field, etc.), the
   audit script extracted the full set of source values before
   comparing against the schema. No comparison-time filtering by
   "expected" patterns. The filter, if any, was applied only to the
   final report grouping — never to the comparison.
7. **URLs in audit recommendations are verified (§5.4).** Every URL
   proposed as a replacement, addition, or canonical form has been
   verified for all three of: HTTP status / final URL after redirects,
   `<link rel="canonical">` form, and destination-identity match. Lists
   of "correct replacement URLs" produced without the three-part check
   are recommendations the audit cannot stand behind.
8. **Source-value verification ran for source-mirror properties
   (§5.5).** For any schema property pulled from a source at render
   time (`sameAs` from footer, organisation `description` from a CMS
   field, etc.), the audit verified both that the schema mirrors the
   source (channel) AND that the source values themselves are accurate
   (payload). A clean mirror of wrong source data is still a Failed
   finding; it just routes to an editorial fix rather than a dev fix.
9. **Source-divergence triage ran (§5.2).** For any finding where the
   implementation's data source differs from the spec's, the triage was
   run before classification: live source identified, 2–3 same-template
   pages sampled for consistency, the spec's source tested on the same
   pages. No "wrong source" finding sits in the Failed section without
   this evidence.
10. **Broadening sweep ran for shared-code findings (§5.2).** Any Failed
    finding implicating a property serialiser, `@id` constructor, address
    builder, or other shared code path was sampled across 5–10 other page
    types, and the report states its scope as the affected code path —
    not per-template.
11. **UI-existence check ran for absent conditional-on-UI schema
    (§5.2).** No missing BreadcrumbList, FAQPage, or other
    conditional-on-UI schema sits in the Failed section without a live
    template check confirming the corresponding UI element actually
    exists on that page type. Absent UI makes absent schema correct.
12. **Full-population sweep preceded the reply.** Where the output is a
    close-out confirmation on a ticket, or a response to a
    client-reported discrepancy, the §5.1/§5.9 sweep across every domain
    and locale was run before drafting — not a spot check on the
    instance under discussion — and the reply states the scope swept.
13. **Sitewide ticket swept against the site's templates.** Where the
    claim includes a parent ticket, the page types swept were listed from
    the global navigation and footer of the home page per locale plus the
    sitemap index — not from the page types the child tickets name — and
    every template no child ticket covers was checked for the parent's
    entities and for extra blocks outside the `@graph`. The report names
    the templates enumerated and their source.

### 5.1 Cross-Language Verification Pattern

For multilingual sites, the spec in Phase 2.5b assigned each property one of three tags (canonical / localized / CMS-derived per-language). Post-deployment, verify those tags held up on the live site across every language version.

**Efficient pattern — fetch once per language, classify via boolean flags.** Avoid copy-pasting schema blocks between languages and eyeballing diffs. Instead, fetch the raw HTML for one representative URL per language and run a scripted pass that:

1. Extracts the JSON-LD blocks via regex (`<script type="application/ld+json">...</script>`).
2. Parses each block with a JSON parser.
3. For each property in the spec, records:
   - **Presence** — is the property present? (boolean)
   - **JSON type** — is it a number, string, array, object? (string)
   - **Non-empty** — does it have a value? (boolean)
   - **Equals canonical value** (for canonical-tagged properties) — does it match the canonical reference? (boolean)
   - **Differs from other languages** (for localized-tagged properties) — is the value different across language versions? (boolean)
4. Returns only the booleans and types, not the values themselves.

Boolean-and-type output is sufficient to classify every property as "matches spec" or "violates spec" without returning the localized values (which would trigger browser content filters and inflate the output). It also side-steps the "everything looks right" failure mode of visual diffing.

**Report structure.** Present findings as a matrix: rows are properties, columns are language versions, cells are pass/fail with the failure reason (`missing`, `wrong-type: string`, `canonical-mismatch`, `unexpectedly-identical`, etc.). A dense matrix makes gaps impossible to miss.

**Live values — spot-check section.** In addition to the pass/fail matrix, include a "Live values — spot check" section that quotes the actual live schema value for every field where visible-UI and schema content overlap textually. Pass/fail alone hides silent formatting leakage from the visible page into the schema — numeric prefixes (`"1. Preheat oven"` leaking into `HowToStep.text`), HTML entities (`&#039;`), currency symbols, unit labels, trailing whitespace, "Step N:" name patterns. The skeptical reader should not have to go and read source to verify what the report already checked.

Include this section only for fields where visible-vs-schema textual overlap is plausible: `name`, `description`, `HowToStep.text`, `recipeCategory`, `recipeIngredient`, breadcrumb `name`, author byline, `Review.reviewBody`. Format: field name, live schema value verbatim, what the visible page shows in the same location — side by side. Any field whose value is derived from rendered HTML should appear here with its raw value. Result: the report tells the reader not just "this passed" but "this is what's in there, and here's what's on the page — you can eyeball the match in five seconds."

**Script template (Python, runnable from Bash in Cowork or Claude Code):**

```python
import re, json, gzip, urllib.request

LANGS = ["en", "de", "fr"]  # adjust per project
URL_TEMPLATE = "https://example.com/{lang}/path-to-page"
SCHEMA_BLOCK = re.compile(
    r'<script type="application/ld\+json">(.*?)</script>',
    re.DOTALL,
)

# Desktop-Chrome UA + Accept-Encoding by default. A bare urlopen() with no
# UA gets a 403 from many WAF/Cloudflare-protected sites; a desktop-browser
# UA usually succeeds where a header-less request AND a Googlebot UA both 403.
HEADERS = {
    "User-Agent": ("Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                   "AppleWebKit/537.36 (KHTML, like Gecko) "
                   "Chrome/124.0.0.0 Safari/537.36"),
    "Accept-Encoding": "gzip, identity",
}

def fetch(url):
    req = urllib.request.Request(url, headers=HEADERS)
    resp = urllib.request.urlopen(req, timeout=15)
    raw = resp.read()
    if resp.headers.get("Content-Encoding") == "gzip":
        raw = gzip.decompress(raw)  # pages can be ~1MB; JSON-LD often past 300KB
    return raw.decode("utf-8", "replace")

results = {}
for lang in LANGS:
    html = fetch(URL_TEMPLATE.format(lang=lang))
    blocks = [json.loads(m.group(1)) for m in SCHEMA_BLOCK.finditer(html)]
    results[lang] = blocks

# Then per property, build a classification table.
# Example: price should be numeric on every language version.
for lang, blocks in results.items():
    # Find the Product / Offer entity and classify `price`:
    ...
```

The script runs in seconds and its output goes into the post-deployment report. Keep a reusable copy for each multilingual client — the only per-project piece is the URL template and the property list to audit.

**Fetching raw HTML for JSON-LD — tool choice.** `mcp__workspace__web_fetch`
returned (at last verification) markdown-rendered content with the `<head>` JSON-LD stripped, which
is useless for schema verification. Run the script above in the **Cowork
sandbox** (`mcp__workspace__bash`) instead: it returns full raw HTML
including every `application/ld+json` block, for non-Latin locales too, and
needs no Chrome browser tooling. "Raw HTML is unreliable in Cowork" is too
absolute — the usual blocker is UA/header-based bot filtering, and the
desktop-Chrome UA above clears it on most SSR sites. For a full-catalogue
crawl (thousands of product/category URLs), the gzip path keeps transfer
~10x smaller and makes it feasible to batch.

**Cross-template audit technique — batch fetch in one browser call.** A
Phase 5 sweep often needs 10–20+ pages across templates and languages —
e.g., to confirm a serialiser bug is sitewide (§5.2 broadening sweep) or to
check whether a finding is example-specific. When the site's bot protection
blocks the scripted-fetch path above, do NOT fall back to navigating to
each URL and extracting per page: each navigation is a separate browser
tool call with a large payload (cost compounds quadratically — see Phase
0.2), and WAF/bot-protected sites re-run their challenge on every
navigation. Instead:

1. **Navigate once** to any page on the target origin to clear the
   WAF/bot challenge (e.g., a Cloudflare interstitial).
2. **Run ONE in-browser JavaScript call** (`javascript_tool` with the
   Claude in Chrome extension, or the equivalent JS-execution browser
   tool) that fetches every target URL with
   `fetch(url, { credentials: 'include' })`, parses each response with
   `DOMParser`, and returns a compact aggregated matrix. Same-origin
   `fetch()` from a cleared tab inherits the challenge clearance, so the
   protected origin behaves like an open one. 18 URLs become one tool
   call instead of 18 navigations.
3. **Return per-URL boolean flags plus small string samples, not raw
   schema blocks.** Compact output avoids both the browser content filter
   (see Phase 1.3) and the result-size truncation browser tools apply to
   large returns — and it's exactly the matrix shape the §5.1 report
   needs.

Adaptable template:

```javascript
(async () => {
  const urls = [/* target URLs, same origin as the current tab */];
  const rows = [];
  for (const u of urls) {
    const r = await fetch(u, { credentials: 'include' });
    if (r.status !== 200) { rows.push({url: u, http: r.status}); continue; }
    const html = await r.text();
    const doc = new DOMParser().parseFromString(html, 'text/html');
    const scripts = Array.from(doc.querySelectorAll('script[type="application/ld+json"]'));
    const blocks = scripts.map(s => { try { return JSON.parse(s.textContent); } catch(e) { return null; } }).filter(Boolean);
    // Aggregate per-page flags from `blocks`: presence, JSON type,
    // length class, prefix/suffix samples — booleans + small strings only.
    rows.push({ url: u, /* flags */ });
  }
  return rows.map(r => JSON.stringify(r)).join('\n'); // one compact line per page
})();
```

Prefer the sandbox Python fetch script above when the site allows direct
HTTP — it costs no browser tool calls at all. Use this in-browser pattern
when bot protection blocks scripted fetching, or when the audit is already
mid-session in the browser and the cleared tab is available.

**Verify ImageObject dimensions browserlessly.** Phase 1.3 documents
dimension verification via the browser (`naturalWidth`/`naturalHeight`), but a
Phase 5 pass often runs entirely through the sandbox (curl/Python) with no
browser open — in which case the browser-only technique becomes a skipped
check. Verify dimensions cheaply by fetching the asset bytes and reading them
from the file header: for JPEG, scan for the Start-Of-Frame (SOF) marker and
read height/width from the bytes that follow; for PNG, the IHDR chunk holds
width and height as big-endian 32-bit integers at bytes 16–24. This catches
swapped or wrong dimensions without any browser tooling.

```python
import struct, urllib.request

def jpeg_dimensions(data):
    i = 2  # skip SOI marker (0xFFD8)
    while i < len(data):
        if data[i] != 0xFF:
            i += 1; continue
        marker = data[i + 1]
        # SOF0..SOF15 carry frame dimensions (excluding DHT/DAC/RST markers)
        if 0xC0 <= marker <= 0xCF and marker not in (0xC4, 0xC8, 0xCC):
            h, w = struct.unpack(">HH", data[i + 5:i + 9])
            return w, h
        seg_len = struct.unpack(">H", data[i + 2:i + 4])[0]
        i += 2 + seg_len
    return None
# PNG: width, height = struct.unpack(">II", data[16:24])

raw = urllib.request.urlopen(image_url, timeout=15).read()
print(jpeg_dimensions(raw))  # → (width, height) from the SOF marker
```

**Verify markup absence with case-insensitive matching.** When grepping live
HTML for an attribute or block to confirm it is *missing* (e.g. "no
hreflang", "no canonical"), match case-insensitively (`(?i)`). React/Next.js/
Vue emit camelCase DOM attributes (`hrefLang`, `crossOrigin`, `charSet`); a
case-sensitive `hreflang="` regex returns 0 matches on a site that actually
has correct hreflang (HTML attributes are case-insensitive, so Google reads
it fine). Never bank an "X is missing entirely" finding from a single
case-sensitive query — confirm with a second method before it reaches a
client report. Absence claims are high-stakes and cheap to get wrong.

**Mirror-the-visible-UI properties: verify against the page, not the ticket's
example labels.** For any property whose spec is "mirror the visible UI
exactly" — breadcrumb `name` values especially — the ticket's Complete
Implementation block necessarily contains *illustrative* label values from
whatever page was used as the example. During Phase 5, diffing the live schema
against those example values produces a **false mismatch** even when the
implementation is correct, because the live page being audited has different
visible labels than the example page. For these properties, verify against the
**live visible UI** as the source of truth, not against the ticket's example
label values. A divergence from the example that matches the visible page is a
PASS. Principle: when the spec for a property is "match the visible page," the
visible page is the oracle, not the ticket example.

**Re-enumerate completeness against the live DOM, not the spec's count.** When
verifying a schema specced as "all X visible on the page" (all people, all
cards, all offices), re-enumerate the full set from the live page rather than
checking the count against the number the spec or source ticket assumed (see
Phase 1.1). Count the repeated card/person structures in the container and
cross-check with a role/label occurrence sweep; account for dynamically-
hydrated sections that sit behind a "Loading…" placeholder in the server HTML.
A schema can be structurally perfect yet incomplete because an entity was added
to the page after the spec was written.

**Source extraction: enumerate first, filter second.** When a schema property mirrors a source container on the page (e.g., `sameAs` mirrors the footer's social-links block; `breadcrumb` mirrors the visible breadcrumb), extract the **full set** of values from the source container before comparing against the schema. Filter only AFTER extraction, for grouping or categorisation in the final report — never as a comparison-time filter. Concretely for footer/`sameAs` audits: pull every `<a href>` inside the social-links container regardless of domain or pattern, then compare against the schema, then group/categorise the matches and mismatches by domain for the write-up.

Pre-filtering the source by "what we expect to find" produces a comparison against a fictional reference rather than the actual one. If the schema includes anything unexpected (an internal URL, a non-social link, a podcast link mixed in with the social icons), the filtered source extraction silently omits it, and the audit incorrectly concludes "schema has extra values not in source" when the source had them too — they just didn't match the audit's filter. The failure mode is most dangerous when the audit's purpose is to find unexpected content, because the filter encodes the very assumption the audit is supposed to challenge.

### 5.2 Audit Finding Categories

Traditional pass/fail is not enough. Use **four** finding categories, not two:

1. **Passed** — Live output matches the spec.
2. **Failed** — Live output violates the spec. Requires a fix ticket.
3. **Exceeded spec** — Live output is correct and well-formed, but includes more than the spec required (e.g., a property that wasn't in the spec, or a language version with richer data than the spec mandated).
4. **Spec superseded** — Live output diverges from the spec (most commonly: a property sourced from a different page element or CMS field than the spec called for), but triage shows the implementation's choice is structurally better than what the spec prescribed. Requires a spec update, not a fix ticket. The first three categories assume the spec is correct; this one covers the case where it isn't. See the source-divergence triage below.

The "exceeded spec" category is not just a vanity label — it triggers a specific action: **update the spec to capture the additional correct output**, so that future audits don't flag it as an unexpected difference and so that future refactors don't regress. Spec updates following an "exceeded spec" finding should be dated and logged the same way fix tickets are.

**Why this matters.** A two-category audit (pass/fail) treats any unexpected-but-correct output the same as an ambiguous near-miss — it either gets flagged wrongly or missed entirely. Explicitly naming the "nice surprise" outcomes says what to do with them (document them). In one post-deployment audit, a content subtree had correctly emitted Article schema that the spec had inadvertently scoped too narrowly. In another, an Organization `logo` property in a global parent ticket was initially mis-categorised as a "Medium issue" deviation when the live output was in fact richer than the spec required — the report had to be reframed to put `logo` under "Exceeded spec" with a spec-update action rather than a fix ticket.

#### Source-Divergence Triage — Run Before Classifying as Failed

When the implementation's data source differs from the spec's data source
(the schema value clearly comes from a different page element or CMS field
than the ticket specified), "wrong source" is a hypothesis, not a
conclusion. Run an explicit triage BEFORE classifying the finding as
Failed:

1. **Identify what the implementation IS using.** Parse the page (e.g.,
   `DOMParser`) and find the element or field whose text matches the live
   schema value.
2. **Sample 2–3 more pages of the same template** to confirm the
   alternative source is used consistently, not just on the audited
   example.
3. **Test the spec's source on the same pages.** Does the spec's element
   exist on all of them? Is its content actually descriptive of the page,
   or is it some other kind of copy (positioning text, boilerplate, a
   marketing strapline)?
4. **Compare structurally.** If the implementation's source is better —
   more available across the template's pages, more genuinely descriptive,
   more consistent — classify the finding as **Spec superseded** (category
   4), not Failed. The action is to revise the Phase 2 design document; the
   implementation stays.

A divergence can decompose into both categories: the source choice may be
spec-superseded while a secondary behaviour on the same property (e.g., a
truncation applied by the serialiser) is still a genuine Failed finding.
Split them. For example, a spec sources `Product.description` from an
intro-text element with a meta-description fallback, and the live
implementation pulls from a dedicated CMS description field instead. Triage
shows the spec's element absent on every sampled page (so the fallback would
always have fired) and holding marketing copy where it does exist, while the
implementation's field is present on all sampled pages with genuinely
descriptive content.
The correct framing is "spec is wrong, not implementation" — the only
Failed finding left is the serialiser's truncate-and-ellipsis behaviour.

#### UI-Existence Check — Run Before Classifying Absent Conditional Schema as Failed

Some schema types are conditional on visible UI: BreadcrumbList mirrors
the visible breadcrumb trail, FAQPage mirrors a visible FAQ block, and
so on. For these types, the absence of the schema is only a gap if the
UI element actually exists on the template. Before classifying a missing
conditional-on-UI schema as Failed:

1. **Load 1–2 live pages of the affected template** and confirm the UI
   element (breadcrumb trail, FAQ block, etc.) is actually present.
2. **Treat catalog and audit-trail claims about visible UI as
   hypotheses, not ground truth** — especially hedged ones ("visible
   breadcrumbs on some of these pages"). An earlier phase's claim about
   the UI is a lead to verify, not evidence.
3. **If the UI element is absent, the absent schema is correct.** The
   finding flips from a fix ticket to, at most, a clarification comment
   — plus a correction to the Phase 1 catalog if its claim was wrong.

"Mirror the visible UI" cuts both ways: absent UI makes absent schema
correct. A coverage gap is only a gap relative to the live UI, not
relative to what an earlier audit assumed the UI was. Like the
source-divergence triage above, this is a check that runs BEFORE a
finding is routed to Failed.

#### Per-Finding Routing — Pre-Write Step

Before writing up any audit finding, classify it explicitly into one of
the four categories above. The default mental model under cognitive load
collapses to "deviation = problem to fix" even when the categories
are loaded — make the categorisation an explicit step, not an implicit
one.

For each finding:

1. **Does the live output use a different data source than the spec
   prescribed?** → Run the source-divergence triage above first. If the
   implementation's source is structurally better → **Spec superseded**.
   Goes in the spec-update section.
2. **Is the live output a violation of the spec?** → **Failed**. Goes
   in the fix-ticket section. If the finding implicates shared code, run
   the broadening sweep below before writing it up. If the finding is an
   *absent* conditional-on-UI schema (BreadcrumbList, FAQPage), run the
   UI-existence check above first — absent UI makes absent schema
   correct (→ **Passed**, with a catalog correction if needed).
3. **Is the live output correct and well-formed but richer than the spec
   required?** → **Exceeded spec**. Goes in the spec-update section.
   Action: revise the Phase 2 design document, not the implementation.
4. **Does the live output match the spec exactly?** → **Passed**.

Do NOT mix Failed and Exceeded-spec / Spec-superseded findings in the same
severity ladder (e.g., "Medium issues" containing both a missing property
and an extra property). They have opposite directions of action and must
live in separate sections of the report — see the report template skeleton
below.

#### Broadening Sweep — Shared-Code Findings

Serialiser-level defects propagate by code path, not by template. When a
Failed finding implicates shared code — a property serialiser, an `@id`
constructor, an address builder, or any other emitter used by multiple
templates — broaden the sample BEFORE writing the finding up:

1. **Identify what the bug implies about the code path.** E.g., "any
   emitter that builds an Organization `@id` reference", "any emitter that
   builds a `description` from CMS content", "any emitter that builds a
   PostalAddress from a dataLayer record".
2. **Sample 5–10 other page types plausibly on the same code path.** The
   cross-template audit technique in §5.1 makes this a single browser tool
   call rather than 5–10 navigations.
3. **Report the scope as the affected code path, with the sampled page
   types listed as evidence:** "affects every page type that emits an
   Organization `@id` reference, including [list]" — not "affects
   template X".

This changes the framing to the dev team from "fix N templates" to "fix
one serialiser, sitewide impact" — which is what's actually wanted: one
shared-code fix beats N per-template fixes, and per-template framing
actively misleads the team into scoping N separate changes. For example,
three findings (a trailing slash on referenced `@id`s, a fixed-length
truncation with literal ellipsis, a dropped `addressCountry`) each first
appear on a single template; the broadening sweep shows all three are
sitewide serialiser defects — the same `@id` constructor behind `publisher`,
`brand` and `seller` references, the same truncating serialiser behind
`Product.description` and `Article.articleBody`, and the same address
builder behind every store's `address`.

#### Report Template Skeleton

Use named sections, not a single severity ladder, so a misclassification
is visible structurally rather than buried in prose:

```markdown
# Post-Deployment Verification Report — [Ticket / Page Type] — [Date]

## Summary
[1–2 sentences. Counts of Passed / Failed / Exceeded-spec /
Spec-superseded findings.]

## Failed (requires fix)
[Fix-ticket items. Each entry: property, expected, actual, severity,
recommended fix. For shared-code findings, state the scope as the
affected code path per the broadening sweep.]

## Exceeded spec (spec update recommended)
[Each entry: property, what the live output adds beyond the spec, why
it's correct, recommended update to the Phase 2 design document. The
implementation stays as-is. Spec-superseded findings (category 4) also
go here, labelled "spec superseded", with the triage evidence — source
availability and content quality across the sampled pages.]

## Passed
[Properties / structural elements where live output matches the spec.
Either an enumerated list or the pass/fail matrix from §5.1.]

## Spec changes proposed
[Concise summary of changes the spec should absorb based on the
Exceeded-spec section. Date-stamp these the same way fix tickets are.]
```

The "Exceeded spec" section is not optional. Even if it ends up empty
on a given audit, the section header should be present — its absence
is what lets exceeded-spec findings drift into the Failed section by
default.

#### Follow-Up Comments to the Dev Team — Diagnosis, Not Prescription

Post-deployment findings are often delivered as follow-up comments on
existing tickets rather than as new formal tickets. The "Notes specify
WHAT, not HOW" rule (§3.3, Rule 35) applies here in full, and the
register shifts one step further toward the collaborative:

- **Deliver the complete diagnosis.** Affected items, the evidence
  (expected vs actual, the exact malformation or gap), a healthy
  counterexample from the same template where one exists, and the
  consumer-impact framing (which consumers are affected — see the
  parser-relative rule in §5.8).
- **Hand the fix decision to the dev team as an open question.** "Could
  you please check why these items produce X while their siblings
  produce Y?" — not "regenerate these through the current output path"
  or "please move the steps into property Z". Prescribing the mechanism
  oversteps the auditor's lane and risks being wrong about how the
  team's build actually works. The auditor owns the evidence and the
  expected outcome; the mechanism of the fix belongs to the team that
  owns the code.

**State the placement rationale for cross-scope consolidation.** When a
follow-up comment (or ticket) consolidates findings that cross the
ticket's nominal scope — because they share one mechanism and one fix —
open with a single line stating why they are grouped there (e.g., "Not
all of this relates directly to this page type, but since it all
concerns the same output mechanism, it fits best here"). This pairs with
the §1.5 merge criteria: consolidation is by shared fix, and the
grouping logic must travel with the comment. Without that line,
out-of-scope items read as a misfiled report rather than a deliberate
consolidation.

**Comment form.** A follow-up comment is read by someone who will
reproduce each finding before fixing it: every claim should be
reproducible in one click, and everything that is not an ask or its
evidence is length the reader pays for. Before posting, check:

- (a) **One verified live URL per example value**, taken from the same
  fetch that surfaced it. A value named without a URL (a handle, a
  language label, an event type) is a lookup task handed to the
  developer.
- (b) **A consistency or normalisation ask carries two URLs** — one
  showing the wanted form and one showing the unwanted form. A bad
  example alone tells the developer what to remove, not what to
  produce.
- (c) **No example appears under two points.** If a value is cited
  once as missing a code and again as needing a different label, fold
  it into the point that fixes it.
- (d) **Cut verification scope, sample sizes and proof counts.** "Some
  profile pages", not "14 of 37 sampled". Those figures belong in the
  internal report; the comment carries the ask and the evidence.
- (e) **A correction to the auditor's own spec is phrased as a rule
  that produced wrong results on live data**, with the data that showed
  it — an unexpected outcome to be corrected, not an apology. "The
  rule for X turned out to produce Y on these pages: <URLs>; please use
  Z instead."

### 5.3 When to Re-Open the Spec

The post-deployment verification closes the loop back to Phase 2:

- **Failed** findings feed fix tickets in the normal Phase 3 flow.
- **Exceeded spec** findings feed spec updates — revise the Phase 2 design document so the next audit knows what to expect.
- **Spec superseded** findings also feed spec updates — revise the Phase 2 design document to adopt the implementation's source, and record the triage evidence (availability and content quality across sampled pages) so the decision is auditable.

A verification pass that produces zero findings of either kind is rare; don't treat the absence of findings as confirmation that the audit was thorough enough. If the audit script returned all-green, spot-check at least one property manually to confirm the audit wasn't misaligned with the live output structure (wrong selector, cached URL, etc.).

### 5.4 URL Verification — Pre-Publish Procedure

Every URL proposed as a replacement, addition, or canonical form for `sameAs` (or any other URL-typed schema property) MUST be verified before delivery — whether the URL is being written into a Phase 3 ticket draft directly, or surfaced in a Phase 5 cleanup recommendation. Skipping verification produces recommendations that look complete but contain URLs that don't resolve (404), redirect to a different canonical, point to wrong entities (misattribution to third parties), or use non-canonical forms that downstream crawls will flag.

The cost of verification (one browser load per URL) is far smaller than the cost of shipping a recommendation whose URLs are broken or misattributed. The downstream cost is high: the team implements the recommendation, the broken/misattributed URL goes into the schema's `sameAs`, and the problem only surfaces in a subsequent audit or when externally noticed.

**Three-part verification — every URL, every time:**

1. **HTTP status / final URL after redirects.** Confirm 200 and capture the redirect target. If the request resolves to a different URL than the proposed one (301/302 redirect to a canonical slug, e.g. `linkedin.com/company/legacy-name/` → `linkedin.com/company/current-name/`), use the **redirect target** as the recommendation, not the original.
2. **`<link rel="canonical">` check.** Confirm the URL is the canonical form. Screaming Frog crawl data is one good source for this (`Canonical Link Element 1` column); live navigation with `document.querySelector('link[rel=canonical]').href` is another. Use the canonical form in the recommendation. Non-canonical URLs in `sameAs` will be flagged by downstream crawls as "Canonicalised: Non-Indexable".
3. **Destination-identity check.** Load the page and confirm the visible content matches the entity the audit claims (correct firm name, correct brand, correct office). This is the check that catches the most dangerous failure mode — a `sameAs` URL that resolves cleanly to a wrong entity (e.g., a stranger's personal X profile being claimed as a canonical reference for the brand).

**Tactical notes:**

- **LinkedIn canonical URLs use country subdomains, not `www.`.** The canonical form is `https://{cc}.linkedin.com/company/{slug}` (`{cc}` a country code) rather than `https://www.linkedin.com/company/{slug}/`. The country subdomain matches the firm's HQ, not the visitor's locale; this is stable. Screaming Frog will mark the `www.linkedin.com` form as "Canonicalised: Non-Indexable" pointing at the subdomain form. Always use the canonical subdomain form in schema.
- **Social platforms are JS-heavy SPAs — simple `fetch` doesn't render the page.** For X, LinkedIn, Instagram, TikTok, etc., verification needs browser navigation with a polling pattern: navigate, then wait for the rendered profile element to appear (`[data-testid="UserName"]` on X, `h1` on LinkedIn), then read identity from it. HTTP status alone is not enough — a non-existent X account returns 200 with a "This account doesn't exist" screen.
- **Don't trust the original audit input.** When verifying a list of URLs sourced from another channel (page footer, CMS field, sitemap), the verification still applies. Mirrored data is no more trustworthy than first-party data.
- **A brand-name string in the page title is NOT identity confirmation.** Common brand names have namesakes — different companies, different industries, different cities. A `linkedin.com/company/{slug}` that returns page title "BRAND | LinkedIn" can still be the wrong entity (e.g. a consulting firm that happens to share the name, not the client). Identity confirmation requires **corroborating detail**: matching address/city, industry, a cross-link back to the site, or logo — or a human who knows the brand's real accounts. When an automated check can only see a name match, report the link as **"unconfirmed — name matches but identity not verified"**, never "probably correct". For client deliverables, route all name-only matches to human confirmation: a link that matches by name alone can still belong to a namesake firm. Same name ≠ same entity.

### 5.5 Source-Value Verification — Mirror Audits Aren't Enough

When a schema property is pulled from another source at render time (footer block, CMS user-entered field, sitemap, external API), confirming that "schema correctly mirrors source" is only half the audit. The other half — verifying that the **source values themselves** are accurate — is essential because schema can faithfully reproduce wrong source data.

A clean mirror of misattributed, non-existent, or non-canonical source values produces a `sameAs` (or other URL-typed property) that's externally wrong while internally consistent. Mirror-only auditing ships green verdicts while the live schema is broken. The failure mode is invisible to standard schema validation tools — the JSON-LD parses cleanly, the URLs are syntactically valid, the markup is "correct" by every automatic check. Only browser-level destination verification catches it.

**Two-part rule for source-mirror audits.** When a schema property is pulled from a source at render time, the Phase 5 audit must do both of the following before reporting green:

1. **Channel check (the standard mirror audit).** Confirm the schema correctly mirrors the source — every value present in the source appears in the schema, and vice versa. Use the full-extraction rule from §5.1 ("enumerate first, filter second").
2. **Payload check (source-value verification).** For every URL in `sameAs` (and other URL-typed properties pulled from external sources), apply the §5.4 URL verification procedure: resolves to the claimed entity, in canonical form, with no broken/misattributed cases.

Both parts must pass before the audit can report green. If part (b) surfaces issues, they typically route to editorial fixes (e.g., update the CMS source) rather than schema dev fixes, but they affect the live schema output regardless and must be tracked alongside dev findings.

**Routing source-value findings.** Source-value issues are still **Failed** findings in the §5.2 finding model — the live schema is incorrect, regardless of where the fix lives. The fix-ticket recipient may be different (content team / CMS editor rather than schema dev), and the report should make that explicit, but the finding category is the same.

> "The schema correctly reflects the source" is not the same as "the schema is correct." Mirror-only auditing produces clean-looking reports for `sameAs` implementations that are externally wrong, because the mirror step doesn't inspect the values, only the channel. Audits that test only the channel are systematically blind to source-level errors that the schema dutifully propagates.

### 5.6 Reconciling GSC Error Counts with a Live Crawl

GSC structured-data error counts and a live crawl legitimately disagree, and
the disagreement is itself diagnostic. Three patterns recur:

**1. GSC count >> live crawl reproduces → suspect a state-dependent field
omission.** GSC is a time-integral over its crawl window; a live crawl is a
point-in-time snapshot. If GSC reports, say, 16 products missing `offers`
but a full live crawl reproduces only 1, the cause is usually a conditional
branch in the generator that drops a field based on transient state — e.g.
the generator omits the entire `offers` block when a product is **out of
stock**, and stock fluctuates, so Google accumulates OOS products across its
window while one snapshot catches only those OOS right now. Don't conclude
"can't reproduce / stale GSC data" — find the conditional branch (fetch a
known out-of-stock product to trigger it). The fix is to **always emit the
field with the correct state value** (`availability: OutOfStock`,
`price` present) rather than omitting the block. The same applies to any
state-dependent field (price, availability).
The emitted state must also be the page's state: map every availability
state the shop sells under (supplier stock, pre-order) to its schema value,
never in-stock-or-not, and build a listing page's `ItemList` from the same
query as the rendered grid — the audit side of both checks belongs to a
JSON-LD validation audit.

**2. The error report category is not the page type.** A rich-result error
can be **attributed to one GSC report (e.g. "Product snippets") while
originating on a page type you wouldn't expect** (e.g. blog/wiki articles
whose `mentions` arrays reference products — see §2.3 on bare `@id`
references). A product-page-only crawl then "can't find" the error. Always
follow the **GSC example URLs** rather than assuming the obvious page type;
the report name describes the *schema feature* flagged, not where the markup
lives.

**3. The error references markup the live source no longer contains → stale
crawl between releases.** When a GSC error points at a markup format or block
that View-Source confirms is genuinely gone — most commonly right after a
Microdata→JSON-LD migration where the old format was removed — the cause is a
stale crawl: GSC is still reporting against a version it last crawled before
the release shipped. This is a distinct, benign diagnosis from "error
reproduces live." Action: first confirm the markup is genuinely gone in
View-Source — verify-absence-first (see §5.1 case-insensitive absence rule)
guards against the trap where a still-present *second* instance of the same
block is the real culprit. Once absence is confirmed, use GSC's "Validate
Fix" / await the next recrawl. No dev work is needed; the report will clear on
recrawl.

### 5.7 Single-Fetch False Greens on Dynamically-Rendered Fields

§5.6 covers the case where GSC's time-integral count exceeds what a live
crawl reproduces. The gap one step earlier is that the **live crawl itself
can produce a false green from a single sample.** On a dynamically-rendered
page a verification fetch is a *point sample of a non-deterministic
process*: an all-green from one fetch is not evidence the field reliably
passes — only that it passed *on this request*. Only repeated sampling
distinguishes "correct" from "correct on this request."

**Check cache headers first.** A page served `cache-control: no-store` /
`cf-cache-status: DYNAMIC` is freshly rendered per request, so inter-request
variance is possible and single-sample verification is unsafe. Make the
header check part of the audit before recording any pass.

**Sample multiple times for array- and count-valued properties.** For any
property whose population depends on a render-time data fetch —
`ItemList.itemListElement` + `numberOfItems`, `BreadcrumbList.itemListElement`,
any list/count — sample each verification URL **multiple times (e.g. 5–8
fetches)** before recording a pass, and especially on `no-store` / `DYNAMIC`
pages. Treat a populated result from a single fetch as **unconfirmed**.

**Failure pattern — empty array with a valid wrapper.** Flag explicitly the
shape where the entity wrapper is fully correct (`@context`, `name`,
`description`, `url` all present and right) but `itemListElement: []` and
`numberOfItems: 0`. This is a **state-dependent-population failure** — a
sibling to the §5.6 OOS-drops-`offers` pattern — and a single lucky fetch
hides it. The typical sequence: the first fetch of a category URL returns a
fully populated, spec-passing `ItemList` and the page is signed off; minutes
later, repeated fetches of the same URL and of sibling categories return an
empty `itemListElement` despite product-detail links present in the same
server HTML. On `no-store` / `DYNAMIC` pages that means the origin
intermittently emits an empty array, not a stale CDN cache. A list on another
render path (a blog index's `CollectionPage`, say) can populate reliably
every time while only the category/product path is affected. The lesson: scope the multi-fetch
check to the code path, not the whole site, but never sign off a
dynamic list/count field on one sample.

### 5.8 Diagnosing GSC "Unparsable Structured Data / Incorrect Value Type" Across Same-Page-Type URLs

When GSC reports an "Unparsable structured data" or "Incorrect value type"
error across a set of example URLs that **share a page type**, treat it as a
**single template bug, not N per-page content problems**. A page type is
rendered by one template, so a malformed field in that template surfaces on
every URL it produces — diagnose on one instance and fix once.

**Fastest diagnostic.** Load one affected page, select all
`script[type="application/ld+json"]` blocks, and `JSON.parse()` each one
**individually**. The block that throws is the culprit, and the parser's error
message points at the malformed field. Validating the page as a whole is slower
and can mask which of several blocks is broken — per-block parsing isolates it
immediately.

**Common failure modes to check first.** Most trace to the same root cause —
values concatenated into the JSON string by hand rather than emitted by a
serialiser — but they surface in distinct ways, and an "unescaped character"
mental model alone won't anticipate them all:

- **Bare `"key": ,`** — a property key with an empty value (the serialiser
  emitted the key but the value resolved to nothing).
- **Unescaped characters in interpolated values** — a literal `"` terminates
  the string early ("Expected ',' or '}'"); raw newlines → "Invalid control
  character".
- **Smart/curly-quote substitution** — a curly closing quote U+201D where a
  straight `"` belongs (ends a description/jobTitle early), or a curly opening
  quote U+201E at the start of a value (e.g., an image URL).
- **Structural errors** — stray or duplicated braces, e.g. an `@id` value
  wrapped in an extra set of braces (`"@id": { "..." }`).
- **Legacy CDATA comment wrappers** — the script content is wrapped in
  `// <![CDATA[` … `// ]]>` comment lines around perfectly valid JSON.
  This is an XHTML-era CMS output pattern that some legacy output paths
  still emit; the JSON grammar inside is fine, but the wrapper makes the
  block unparsable as delivered. The strip test confirms it in one call:
  remove the wrapper lines and `JSON.parse()` succeeds. Wrapper-level
  corruption means parse diagnostics must look at the raw script
  content, not just the JSON grammar.

The fix for the whole class is always the same: emit via a real JSON
serializer (see §2.8), not per-character patching.

**Parse validity is parser-relative — name the failing parser.** Google's
extraction pipeline is lenient: it strips comment wrappers and tolerates
raw control characters that a strict RFC-8259 parser rejects. Crawlers,
monitoring tools, and AI/LLM consumers typically parse strictly. So the
same block can be "unparsable" to the crawler that flagged it while
showing 0 errors in validator.schema.org — and an audit that just says
"unparsable" invites dev-team pushback ("validates fine for me"). Before
reporting a malformed-JSON finding, test the block in the consumer whose
behaviour motivated the finding AND in validator.schema.org, and report
both results, naming the parser that fails. Severity framing follows:
malformations Google tolerates are lower severity — the risk is strict
JSON consumers (AI/LLM crawlers, monitoring tools), not Google's rich
results.

**Cross-locale sweeps.** When the page type exists across many locales,
run the per-block `JSON.parse()` as a boolean sweep over every locale first
(which locales parse, which don't), then localize the exact breaking block and
character on the failures. Parse-validity as a boolean finds the affected
subset fast; per-block parsing pinpoints which of several blocks failed while
the others validate. A template-level fix can also ship unevenly across
locales, so re-verify the full locale set rather than trusting a prior
affected-locale list.

**Template bug vs per-content-item bug.** The same-page-type heuristic has
one important exception, with a clean diagnostic signature: when only
specific content items fail while other items of the same template are
healthy — and each failing item fails on **every** locale it exists in —
the bug is in that item's stored or generated output, not the template. (A
template bug breaks all items; a locale-rollout gap breaks all items on
some locales; a per-item output bug breaks the same few items on all
locales.) Fix scope is then per-item, not per-template: run the per-block
parse (and, for wrapper corruption, the strip test) on one failing
instance to confirm the mechanism, and report the affected items by
ID/slug with a healthy sibling from the same template as counterexample.

**Scope of the fix.** Fix at the template level; expect one fix to clear the
whole set of flagged URLs on recrawl. Principle: structured-data errors
reported across URLs of one page type are template bugs — diagnose on one
instance, parse blocks individually, fix once. (Unless the cross-locale
pattern above shows a per-content-item bug — then scope the fix to the
affected items.)

### 5.9 Coverage Audit from Crawl Exports

Per-URL verification (§5.1–§5.8) proves **correctness** on the pages
checked. It cannot prove **completeness**: a schema emitter that silently
skips an entire page type, cuts off deep in pagination, or fails on
specific content items will look perfect on every example URL picked for
verification. Example-URL verification and full-crawl coverage auditing
find disjoint error classes — a thorough Phase 5 needs both.

**Input.** A full-site crawl export with at least three columns: URL,
detected schema types, HTTP status (e.g., a Screaming Frog structured
data export, or any crawler that records schema types per URL).

**Workflow.**

1. **Clean the export.** Exclude junk hosts (staging, CDN, third-party
   domains swept into the crawl) and non-200 URLs. Coverage claims only
   make sense over 200-status URLs on the audited host(s).
2. **Classify URLs into the Phase 1 page types** via URL patterns. The
   Phase 1 catalog is the classification key; URLs matching no pattern go
   into an "unclassified" bucket that is itself worth inspecting.
3. **Pivot schema types per page type.** For each page type, compute the
   percentage of URLs carrying each schema type the Phase 2 spec expects
   there.
4. **Treat <100% coverage on an expected type as a lead, not a verdict.**
   Drill into every gap before reporting: per-domain and per-locale
   splits, plus the two diagnostic slices below.
5. **Live-verify a sample from each gap before classifying.** The §5.2
   routing applies in full — including the UI-existence check for
   conditional-on-UI types, and per-block parsing per §5.8 for
   zero-schema URLs (some crawlers report "no schema" for a page when
   any single block on it fails strict parsing, so "zero schema" in an
   export can mean "one malformed block").

**Two decisive diagnostic slices.**

- **Pagination-position analysis.** Split coverage by pagination position
  (`?p=n` or equivalent). A deterministic emitter cutoff shows up as a
  hard edge (e.g., ItemList present on pages 1–3 and absent from every
  page ≥ 4, fleet-wide); an intermittent rendering issue shows up as a
  scattered percentage across positions. The two have entirely different
  fixes, and only the position split distinguishes them.
- **Cross-locale clustering of zero-schema URLs.** Group zero-schema URLs
  by content item across locales. When the same few content items fail on
  **every** locale while template siblings are healthy, the cause is
  content-item-level malformed output (see §5.8's per-content-item
  diagnostic), not a template or locale bug — and the fix list is those
  items, by ID/slug.

**Principle:** example-URL verification proves correctness; only a
full-crawl coverage pivot proves completeness. Run the coverage audit
whenever a fleet-wide crawl export is available or cheap to obtain.
