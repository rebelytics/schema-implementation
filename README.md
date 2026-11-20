# schema-implementation

Open-source skill that **implements schema.org structured data across a website**: the full workflow from page type analysis through developer-ready ticket creation (JIRA, Wrike, or other project management tools). It provides an end-to-end methodology for analyzing websites, designing schema.org markup and producing developer-ready tickets, refined through real-world projects implementing structured data across multilingual, multi-regional websites. The methodology is client-agnostic: it works for any website where the goal is comprehensive schema.org coverage with developer handoff via a project management tool. Load it into any MCP-capable AI agent (Claude, Cursor, Codex, n8n, etc.).

Part of the [Open Skill Library](https://github.com/rebelytics/one-skill-to-rule-them-all/blob/main/OPEN-SKILL-LIBRARY.md), the open-source agent skills maintained with Task Observer.

## When it triggers

Any mention of "schema.org", "structured data", "rich results", "JSON-LD", "JIRA tickets for schema", "Wrike tasks for schema", or requests to analyze website pages for SEO markup. Also when the user references schema implementation methodology, ticket writing rules, or page type analysis workflows.

## What this repo contains

- `SKILL.md` — how the skill works, the pointer list for the reference files, Phase 0 (setup and tooling) in full, one stub per later phase, the universal technical requirements, the anti-patterns, session management and the required user input checklist.
- `references/discovery.md` — Phase 1: page type catalog, triage, Google guidance check, per-page-type analysis, saving real data, consolidation, classification.
- `references/schema-design.md` — Phase 2: global and page-specific schema, `@id` cross-references, deprecation vs removal, property selection rules, multilingual property values, verifying assumptions, project rules.
- `references/ticket-writing.md` — Phase 3: the ticket template, subagent guidelines, the numbered ticket-writing rules, the parent ticket, the pre-delivery checklist, the feedback workflow and common schema decisions.
- `references/delivery.md` — Phase 4: regeneration plan, platform-specific formatting, the delivery-format decision tree, editorial pass, cross-ticket consistency checks, packaging.
- `references/post-deployment-verification.md` — Phase 5: cross-language verification and fetch techniques, finding categories and report skeleton, follow-up comments, Search Console error reconciliation and diagnosis, coverage audits from crawl exports.
- `references/measuring-impact.md` — Phase 6: the Search appearance breakdown, the expectation statement, the measurable channels, data retention.
- `LICENSE` — CC BY 4.0.

## Install

Install the whole skill directory — `SKILL.md` and `references/` must travel together — into your MCP-capable agent's skills directory, following the client-specific path, and restart the client.

## Contributing

Open an issue or a PR at [github.com/rebelytics/schema-implementation](https://github.com/rebelytics/schema-implementation).

## Credits

- Original author: [Eoghan Henn](https://www.rebelytics.com) / [LinkedIn](https://www.linkedin.com/in/eoghanhenn)

## License

CC BY 4.0 — the licence text is in [LICENSE](./LICENSE): the full legal code, unmodified, as Creative Commons publishes it; the [human-readable summary](https://creativecommons.org/licenses/by/4.0/) says what it allows. Use it, fork it, adapt it, monetise it. Keep the attribution.

Copyright (c) 2026 Eoghan Henn / rebelytics.com
