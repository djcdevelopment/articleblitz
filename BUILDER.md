# BUILDER.md — Rules Board

Handoff for the next builder. A **working, verified prototype** already lives at
[`docs/tune.html`](docs/tune.html) — live at
<https://djcdevelopment.github.io/articleblitz/tune.html>.

Your job: elevate it to the polished, tactile design brief at the bottom **while keeping
the functional contract intact**. Treat the prototype's *look* as replaceable and its
*behavior* as the spec.

---

## What the Rules Board is

A tactile control board for editing the game's backing config. Today, proposing a rule
change means hand-editing JSON and filling a free-text "tell your agent" template that the
agent has to interpret. The board replaces that: you drag sliders / tap steppers, toggles,
chips, and segmented controls; it diffs your edits against the loaded config, runs the
contract checks the schema referee would, and — only when the result is a legal move —
emits a **change request that carries the exact resulting `game.config.json`**. The agent
just writes that file verbatim, validates it, and opens the PR. Nothing to misread.

JSON stays the single source of truth. The board never persists on its own.

## Backing store (do not change the governance)

- `rules/game.config.json` — the game (what the board edits).
- `rules/contract.schema.json` — the referee. A change is only legal if it validates.
- `rules/ratification.json` — the ready-to-play flags. **The board must never touch this.**

Change flow is unchanged: branch → PR → the *other* player approves → merge. No self-merges.
See [`AGENTS.md`](AGENTS.md) and the site's §III/§IV for the canonical rules.

## Working reference

`docs/tune.html` is the current prototype and the **behavior baseline to preserve**. It is
a single self-contained file (inline CSS/JS, no external requests, works offline). Open it
directly, or serve `docs/` and hit `tune.html`. Its inline baseline mirrors
`rules/game.config.json` at v0.3.0; the "Load latest from GitHub" button re-baselines from
the live raw file (works when served from `github.io`, blocked by CSP inside a claude.ai
Artifact — there, use the paste-to-rebaseline box).

---

## Functional contract to preserve

1. **Render controls from the config**, grouped by JSON path, with the path shown as a
   monospace sub-label so every control maps to exactly one place in the file.
2. **Live diff panel** — every changed leaf path shown as `old → new`.
3. **Contract-check panel** (the referee). Encode pass/fail in *form* (chip/stripe/pill),
   not just text. Checks:
   - `weights.jury` sums to exactly 1.00
   - `pool_size ≥ 2`; `finalists ≥ 1` and `finalists < pool_size`
   - `≥ 2` unique `candidates`
   - fable in the pool only if `fable_legal`
   - `≥ 2` jury seats
   - `adjudication_trigger.threshold ≥ 0`
   - soft **warning** if the pool spans fewer distinct cost tiers than `min_cost_tiers`
4. **Sticky status**: `in sync` / `N changes` / `illegal move`.
5. **Export** — a "Draft the change request" block that is **blocked with a clear reason
   whenever the config is not a legal move** (or there are no changes). When legal, it
   produces:
   - **(a)** a copy-ready agent prompt instructing the agent to create branch
     `cr/<player>/<topic>`, replace `rules/game.config.json` with the exact JSON verbatim,
     leave `ratification.json` untouched, verify JSON + schema + weight sums, and open a PR
     to `main` titled `CR: <summary>` with a per-change bulleted body — then **stop** (the
     other player approves; no self-merge).
   - **(b)** the exact resulting `game.config.json`.
   - Inputs: player (`djm` / `durracktu`), topic (→ branch slug), summary (→ PR title).
6. **Re-baseline**: paste a `game.config.json` to adopt as baseline; optional fetch of the
   live raw file.
7. **Reset** to the loaded baseline; **normalize → 1.00** for the weights.

## Data model / controls

Models: `claude-haiku-4-5`, `claude-sonnet-5`, `claude-opus-4-8`, `claude-fable-5`.
Efforts: `low`, `medium`, `high`, `xhigh`, `max`.

- `weights.jury` — 6 sliders (`output_quality`, `evidence`, `citations`, `cost_efficiency`,
  `reproducibility`, `confidence_calibration`) that must sum to 1.00. **Signature
  interaction** — dragging one against a live sum meter should feel great.
- `models.pool` — `fable_legal` (toggle), `min_cost_tiers` (stepper), `candidates` (model
  chips; fable chip disabled unless `fable_legal`).
- `params` — `pool_size`, `finalists`, `prompt_budget_points`, `challenge_tokens`
  (steppers); `assignment_method` (segmented: `fully_random` / `snake_draft` /
  `weighted_lottery` / `auction` / `hidden_until_execution`); `ban_order` (which player
  bans first); `cache_jury_packets` (toggle); `adjudication_trigger` (type segmented +
  `threshold` + `scale` steppers).
- `params.information_purchases` — 6 price steppers in budget points (`context_size`,
  `temperature`, `model_family`, `reasoning_depth`, `repository_count`,
  `evaluation_rubric`).
- `models.harness` — `jury_panel` (add/remove seats, each model select + effort segmented,
  min 2) and single slots (`adjudicator`, `appeals`, `calibration`, `telemetry`,
  `report_stats`, `report_replay`) each model + effort. `rerun` is the sentinel
  `@original` (display only).

---

## Design brief

This is an **operator tool**, not a landing page — scanned and operated, so the craft goes
into information design and tactile controls, not a hero. Ship one self-contained `.html`
file (inline CSS/JS, no external requests, works offline).

**Subject & voice.** The game's metaphor: *"billiards in a random poolhall — the rules
matter less than both players agreeing to them before the break."* A game of incomplete
information: ban, bet, budget, buy information, execute blind, face a jury. Keep that
poolhall / wager voice in the microcopy.

**Honor the existing brand** (do not drift to generic AI style). Match the live site:

- Ground: warm near-black `#14161a` (dark) / warm paper `#f6f4ef` (light). Panels `#1d2026`.
- Accent: a single amber `#e0a458` (dark) / `#a2661a` (light). Spend boldness only here.
- Semantic, separate from the accent: good `#7fb069`, bad `#e0685c`.
- Type: Georgia italic for display/headings; `ui-monospace` for labels, values, paths, and
  all data; `system-ui` for body. Uppercase mono eyebrows with letter-spacing.
- Ink `#e8e6e1` on dark, dim `#a09c94`. Support **both** themes via CSS-variable tokens +
  a manual toggle that overrides `prefers-color-scheme` (`data-theme` on `<html>`).
- `tabular-nums` everywhere digits align.

**Tactile goal.** Controls should feel physical — steppers with satisfying press states,
toggles that slide, sliders with a live value bubble, chips that select. Respect
`prefers-reduced-motion`. Keyboard-operable with visible focus. No layout shift on edit.
Take **one** real aesthetic risk that fits the poolhall subject (felt-table texture,
chalk-mark accents, a rack/wager motif for the export) — keep everything around it quiet,
never at the cost of reading state fast.

## Verify before you ship

- Weights sum guard flips the board to `illegal move` and blocks export; **normalize**
  restores `Σ 1.00`; reset returns to a clean baseline.
- Break each contract check and confirm export blocks with a clear reason.
- The exported prompt carries the **exact** resulting JSON, and it round-trips: applying it
  reproduces your edits and still validates against `rules/contract.schema.json`.
- `ratification.json` is never referenced by the export.
- Both light and dark themes; reduced-motion; keyboard nav with visible focus.
