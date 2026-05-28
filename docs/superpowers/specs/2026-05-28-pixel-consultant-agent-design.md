# Pixel Consultant Agent — Design

**Date:** 2026-05-28
**Status:** Approved
**Audience:** Management consultants (Accenture-style), no SQL background
**Inspiration:** [Czechitas Pixel Agent](https://ishalkin.github.io/czechitas-pixel-agent/index.html)

## Goal

Build a single-file, pixel-art HTML simulation that shows a consulting agent walking through 5 office stations, reading the `accenture-pptx` SKILL, calling MCP servers (Canva + draw.io), assembling one branded hero slide, and passing it through an automated QA evaluator. Audience can toggle Skill / MCP / Evals on and off to see what breaks.

The point is **simulation and beautiful presentation**, not technical accuracy or completeness — consultants must "get it" in 30 seconds.

## Non-goals

- Real PowerPoint generation (no PptxGenJS, no python-pptx)
- Real MCP wire protocol — visualize the *concept*, not the actual JSON-RPC traffic
- Full coverage of all `accenture-pptx` rules — only 5 main rule groups
- Multiple briefs / scenarios — single brief is enough
- Snowflake or any other data source

## Stations (5)

```
[INBOX] → [SKILL] → [MCP] → [BUILD] → [QA]
```

| # | Station | What happens |
|---|---|---|
| 1 | **INBOX** | Agent picks up brief: *"Cloud strategy, 1 slide, hero key message"* |
| 2 | **SKILL** | Agent opens `accenture-pptx/SKILL.md`, fills checklist of 5 rule groups |
| 3 | **MCP** | Two calls: Canva (logo, gradient bg, photo) → draw.io (architecture diagram). Toolbox callout shows each MCP call as a line. |
| 4 | **BUILD** | One hero slide assembles layer by layer on the board: bg → logo → title → chart → callout |
| 5 | **QA** | `accenture-pptx-qa` critic walks around the slide. Either ✅ pass, or red circles around violations with text labels ("8pt < floor", "Calibri off-brand"). |

## Output: one hero slide assembling layer by layer

Big board to the agent's left holds a single Accenture-style slide (16:9). During BUILD, layers stack:
1. White background with Accenture purple gradient bar
2. `>` GT mark (top-right, brand element)
3. Logo (top-left)
4. Title: "Cloud strategy 2026"
5. Body chart (faked bar chart, purple bars)
6. Callout box (purple highlight)

During QA, the critic agent draws red circles around violations, with mini-text labels.

## Toggles (3)

| Toggle | ON | OFF |
|---|---|---|
| **Skill loaded** | Graphik, A100FF, sentence case, 14pt | Calibri/Comic Sans, blue, ALL CAPS, 8pt |
| **MCP server** | Logo + photo + draw.io diagram render | `[MISSING ASSET]` grey placeholders |
| **Evals** | QA stage runs, red flags caught | QA skipped → broken slide ships → ✉ angry client e-mail in inbox |

Toggles drive the simulation: when OFF, replay shows the broken state. The point is to make consultants *feel* what each layer adds.

## Rule groups shown at SKILL station (5)

The SKILL station's recipe-card callout shows a checklist of 5 rule groups (not all rules — too much). Each item ticks during the SKILL phase as the agent reads:

1. **Brand colors** — Purple A100FF dominant, ≤5% secondary
2. **Typography** — Graphik, 10pt absolute floor, sentence case
3. **Layout** — margins 0.45"/0.35", squares not circles
4. **Slide modes** — Presentation 12-14pt vs. Detail 10-11pt
5. **Human-editable** — 1 info unit = 1 shape

## Style & tech

- Single HTML file: `pixel-consultant.html`
- Pixel-art canvas, 480×280 logical resolution, scaled 2× for crisp pixels
- Color palette: Accenture purple spectrum (`#A100FF`, `#460073`, `#7500C0`), neutrals (`#FFFFFF`, `#818180`, `#F1F1EF`)
- No external dependencies — sprites drawn programmatically as in Czechitas index.html
- Office furniture mood: meeting table, monitor, whiteboard, water cooler
- Agent sprite: business-casual character (re-skin the Julia sprite with a blazer color palette later if time)

## Hosting

- New GitHub repo: `IShalkin/pixel-consultant-agent`
- GitHub Pages from `main` branch root
- Final URL: `https://ishalkin.github.io/pixel-consultant-agent/`

## Scope boundary (what we DO NOT build now)

- Real `.pptx` export
- Real Canva or draw.io API integration
- Multiple briefs / scenario picker
- Speaker script / homework guides
- Hub / index landing page (this lives standalone, optionally linked from a future consulting hub)

## Acceptance criteria

A consultant who has never seen the deck:
1. Lands on the page, watches one full loop without instruction, and can describe in their own words: "agent reads brand rules, calls helper tools for assets, assembles slide, passes review"
2. Toggles Skill OFF and immediately sees that the output looks generic / broken
3. Toggles Evals OFF and sees the broken slide ship to the client
4. Total time from landing to "I get it" ≤ 30 seconds

## Open items deferred to plan

- Exact frame timing for layer-by-layer slide build
- Mini-deck view vs. single-slide view (decided: single hero slide)
- Whether to add Czech vs. English UI text — assume English (international consulting audience), confirm in plan phase
