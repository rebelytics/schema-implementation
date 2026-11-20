# Phase 4: Final Regeneration & Delivery — reference for the schema-implementation skill

Load triggers: the final regeneration pass; formatting or delivering tickets on the target project management tool

Reference file for the `schema-implementation` skill (Eoghan Henn and contributors, CC BY 4.0). The core `SKILL.md` — the phase overview, §2.8 Universal Technical Requirements, the Anti-Patterns list and the Required User Input checklist — applies alongside this file. Section numbers (§1.2, §5.4) and rule numbers (Rule 35) are the same in every file; the core's section map says which file holds each phase.

## Phase 4: Final Regeneration & Delivery

**Goal:** Regenerate all tickets in a single pass, applying every accumulated rule, decision, and correction for cross-ticket consistency.

### 4.1 Build a Regeneration Plan

**Skill compliance sub-check (run first):** before drafting the checklist,
re-read the Phase 3 rules (Ticket-Writing Rules, Pre-Flight Checklist,
Anti-Patterns) and diff each ticket's *current* content against them. Flag
every violation as a global change for the regen. Iterative regen cycles
re-inherit prose accumulated in earlier cycles, and rules added to this
skill mid-project are exactly the ones existing tickets silently violate.
Regeneration plans that only track changes in the deliverable's subject
matter miss changes in the guidance for producing the deliverable — both
need a checkpoint.

Then create a master checklist document that captures:

1. **Global changes** — Changes that apply to every ticket (including the
   skill-compliance violations found above)
2. **Ticket-specific changes** — Changes unique to each ticket
3. **Open questions** — Inconsistencies or unresolved decisions to clarify with the user
4. **Cross-ticket consistency checks** — Properties and patterns to verify across all tickets

### 4.2 Resolve All Open Questions First

Before regenerating any ticket, resolve every open question with the user. Common open questions:
- How to handle recurring elements (e.g., people, categories) that serve different roles across page types
- How to handle inconsistent data formats across page types
- Whether to harmonize formatting (e.g., Language objects vs. simple strings)

### 4.3 Regenerate in Sequence

Recommended order:
1. Parent ticket first (foundation for all references)
2. Enhancement tickets (simpler — schema already exists)
3. Creation tickets (more complex — full new schema)

### 4.4 Platform-Specific Formatting Pass

Before delivering tickets, verify that formatting works on the target platform (as chosen in Phase 0). Different platforms render markdown differently, and what looks correct in a markdown editor may break in the target tool.

**Wrike-specific issues:**
- **Nested code blocks in list items** — Code blocks (fenced with triple backticks) nested inside bullet points or numbered lists render as inline text in Wrike, with the language identifier prefix visible (e.g., `json { "@type": "Brand" }` instead of a formatted block). **Fix:** Do not nest code blocks inside Required Changes list items. Describe the change in prose within the bullet point and rely on the Complete Implementation section to show the exact JSON structure. For short inline JSON snippets, use backtick inline code formatting instead of fenced code blocks.
- **Top-level code blocks DO render correctly** — at last verification, Wrike's Quill-based description editor supported `<pre>` blocks and inline `<code>` (despite older community-forum posts that mark this as "Not planned" or "Backburner" — those threads pre-date the current editor). When pasted from a rendered HTML source, `<pre>` becomes a styled code block (`<pre spellcheck="false">`) and JSON-LD examples survive intact. Don't strip code blocks from tickets pre-emptively based on web search results about the tool — verify against the live editor first. Tool capability claims from community posts and feature-request threads have a shelf life; the live DOM is the source of truth.
- **Hierarchy context** — When tickets live inside a parent/child hierarchy (e.g., Wrike subitems), add contextual specificity to help readers orient: "(parent ticket)", "(subitems)", "(desktop)", etc.

**JIRA** — Standard markdown code blocks render reliably. No special formatting adjustments needed beyond the default ticket structure.

**Rich text editors (Zoho Projects, Basecamp, Monday.com, etc.):**
- Code blocks frequently break in rich text editors — HTML renders instead of
  displaying as code, line breaks are lost, and pasted content escapes the code
  block. JSON-LD examples are the most fragile element.
- **Recommended format:** Generate tickets as .docx files instead of
  markdown/HTML. Word documents preserve code formatting reliably and can be
  edited in any word processor (Pages, Word, Google Docs) before being pasted
  or attached to the PM tool.
- **PDF as final delivery:** If the client needs a non-editable reference,
  export the .docx to PDF after review.
- **Do not attempt to paste code blocks into rich text editors.** This
  consistently fails and wastes time on workarounds.

**WAF-protected tools (any self-hosted or proxied PM tool sitting behind
Cloudflare or a comparable WAF):**
- **A submission that vanishes on save is a payload problem before it is a
  formatting problem.** Schema tickets are unusually exposed here, because the
  Complete Implementation format deliberately includes a literal
  `application/ld+json` script-tag wrapper, and WAF XSS rules match script tags
  in POST bodies. The symptom mimics a rendering bug — the comment silently
  fails to save, or content is lost on submit — so the instinct is to reformat.
- **Check the console/network response before changing anything.** The
  issue-update API call returning a WAF block page ("you have been blocked …
  submitting a certain word or phrase … or malformed data") settles the
  diagnosis in one look and saves a round of format experiments.
- **Defang the trigger; do not rebuild the deliverable.** Drop the script-tag
  wrapper from the code block and state the placement in prose instead — "please
  keep the schema in its existing separate script tag of the type
  application/ld+json" followed by the location observed in View Source for
  that block on the sampled pages ("in the head section" / "in the body, next
  to the video embed"), never a default (Rule 8b) — or attach the spec as a
  file. The rest of the ticket is fine as it is.

### Delivery Format Decision Tree

When starting a project (Phase 0), determine the delivery format early:

1. **Target PM tool supports Markdown natively?** (GitHub Issues, GitLab,
   JIRA with Markdown mode) → Generate as Markdown files
2. **Target PM tool supports raw HTML input?** → Generate as HTML files
3. **Target PM tool uses a rich text editor?** (Zoho, Basecamp, Monday,
   Asana descriptions) → Generate as .docx files, with PDF export as an
   option for final delivery
4. **Unsure?** → Test-paste ONE ticket with a code block into the target tool
   before generating the full set. Code snippets (JSON-LD examples) are the
   most fragile element — if they survive the paste, everything else will too.
   Make sure the test payload includes the literal script-tag wrapper: on a
   WAF-fronted tool that is the element that gets the submission blocked, and a
   wrapper-free test passes while the real tickets fail.

If using a platform not listed here, test-paste one ticket before generating the full set, and adjust formatting based on what renders correctly.

### 4.5 Editorial Pass

Before delivering final tickets, apply these editorial checks (refined from real publishing feedback):

1. **Redundancy removal** — Delete any Note that duplicates information already stated in another section of the same ticket (e.g., a body class mentioned in both Detection and Notes, or an optimisation opportunity restated in Required Changes and Notes).
2. **Contextual specificity** — Add precision that helps readers orient within the project management hierarchy and page context (e.g., "parent ticket", "subitems", "desktop layout", "when recipe page template is updated").
3. **Diplomatic softening** — For instructions to external development teams, prefer softer language where reasonable flexibility exists. "varies" → "may vary", "instead of" → "instead of (or in addition to)". Avoid absolutes unless the requirement is genuinely absolute.
4. **Conciseness** — One good example is sufficient if developers can extrapolate. Remove secondary examples that don't add new information.

### 4.6 Cross-Ticket Consistency Checks

After regenerating all tickets, verify:

| Check | What to verify |
|-------|----------------|
| `@id` references | Same reference format across all tickets |
| `@context` format | `https://schema.org` (no trailing slash) everywhere |
| Detection patterns | Consistent format (body class OR dataLayer condition OR URL pattern) |
| DataLayer path | Same path format everywhere (e.g., `dataLayer.content.pageType`) |
| Language objects | Same structure (name + alternateName) in every ticket |
| Placement section | Word-for-word identical across all child tickets |
| Parent schema references | All child Placement sections accurately reflect parent scope |
| DataLayer disclaimer | Present in every ticket whose code examples reference `dataLayer.*` values (Rule 13) |
| No fictional data | Zero invented names, URLs, or descriptions |
| Comment style | `// identical to` used consistently |
| Greeting | Consistent with agreed style |
| No "Related Tickets" | Zero occurrences |
| No "Not In Scope" | Zero occurrences (unless genuinely disambiguating) |
| No implementation suggestions | Zero "taken from", "generated from", "derive from", "extract from", or HOW directives |
| Notes factual only | Zero interpretive or evaluative language in Notes sections |
| Internal consistency | Current Implementation, Required Changes, and Notes agree on scope within each ticket |
| Property accounting | Every Current Implementation property tracked in Keep, Update, Add, or Remove — no silent drops |

### 4.7 Package for Delivery

Final deliverables:
- 1 parent ticket (markdown file)
- N child tickets (markdown files, one per page type or consolidated group)
- Each file is self-contained and copy-paste ready for the target platform

Naming convention: `[platform]-parent-ticket-[name]-v[N].md` and `[platform]-child-ticket-[name]-v[N].md` (e.g., `jira-child-ticket-product-v3.md` or `wrike-child-ticket-product-v3.md`)

### 4.8 Wrike Delivery — XLS Skeleton + HTML Paste

Wrike's importer creates the task hierarchy reliably from an XLS file but strips formatting from the Description column. Wrike's description editor (Quill-based) accepts richly-formatted content via clipboard paste from a rendered HTML page. The combination of these two facts gives a two-phase delivery pattern that other PM tools with similar import-vs-editor split (Asana, Monday.com, Trello via Power-Up imports) often share.

**Workflow:**

1. **Generate the XLS skeleton.** One row per ticket. Required columns: `Key` (ticket ID for the parent → child reference), `Parent Task` (the Key of the parent for child tickets; blank for the parent itself), `Title`, `Status`, `Priority`, `Assigned To`, `Description` (leave blank or place a one-line placeholder — descriptions get pasted in step 4). Use a Python `openpyxl` script driven from the same source-of-truth markdown that produced the v[N] tickets, so the skeleton can't drift from the ticket bodies.
2. **Convert each ticket markdown to clean semantic HTML.** Keep only the tags Wrike's Quill editor renders reliably: `h1`, `h2`, `h3`, `p`, `strong`, `em`, `ul`, `ol`, `li`, `pre`, `code`, `table`, `thead`, `tbody`, `tr`, `th`, `td`, `a`. Strip wrapper `<html>` / `<body>` / class attributes / inline styles — the paste path tolerates only the semantic content. A small Python helper using `markdown` + `BeautifulSoup` is the right shape; one HTML file per ticket, named after its Key.
3. **Import the XLS to Wrike.** This creates every task and the parent → child hierarchy in one pass. Verify the count and the assignments before moving on; correct skeletal mistakes here, not after pasting bodies.
4. **Paste descriptions from rendered HTML.** Open each ticket's HTML file in a browser tab. `Cmd+A` to select all → `Cmd+C` to copy → click into the Wrike description editor → `Cmd+V` to paste. Headings, tables, code blocks (the `<pre>` form, not nested in lists — see §4.4), and inline code all survive the paste intact. Verify a few tickets via the Chrome extension's `read_page` to confirm the rendered output if formatting fidelity matters.

**Why two phases instead of one.** The XLS importer is fast and correct for hierarchy + metadata but loses formatting; the editor preserves formatting but is slow to drive programmatically. Each tool is used for what it does well. Trying to embed rich descriptions into the XLS column (HTML, markdown, or otherwise) loses the formatting; trying to drive task creation through the editor doesn't scale.

**Generalisation.** The pattern — *import structure via spreadsheet, paste formatting via clipboard from a browser-rendered HTML page* — is portable to any PM tool whose importer strips description formatting but whose editor accepts pasted rich content. Verify the editor's behaviour with one test paste before generating the full set; the Wrike-specific detail is that pasted code blocks DO render despite forum posts to the contrary (see §4.4).
