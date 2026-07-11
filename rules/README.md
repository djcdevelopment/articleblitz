# Rules Overview — v0.3 (draft, unratified)

Three files, one contract:

| File | What it is | Changes how |
|---|---|---|
| `contract.schema.json` | The shape every config must satisfy | PR + other-player approval (schema changes are rule changes) |
| `game.config.json` | The actual game: models, phases, weights, params | PR + other-player approval; frozen at lock |
| `ratification.json` | The handshake: ready_to_play flags, lock state | Each player flips only their own flag |

Everything below is **sample values for ease of use** — edit `game.config.json`
via PR; nothing here binds until both flags in `ratification.json` are `true`.

## Model slots

The config never hardcodes a model into a rule — phases reference slots,
slots hold `{model, effort}` pairs (or a sentinel like `"@original"`).
Amend = change one line. Extend = add a slot (see below).

| Slot | Phase | Sample | Rationale |
|---|---|---|---|
| `jury_panel` | 9 | 3× Sonnet 5 @ high | Biggest token line item (~3× every output); bounded rubric task; panel redundancy > single-reviewer capability |
| `adjudicator` | 9 | Opus 4.8 @ xhigh | Only contested reviews reach it |
| `appeals` | 10 | Opus 4.8 @ xhigh | Scarce by design; hardest eval task in the game |
| `rerun` | 10 | `@original` | A rerun on a different model isn't a rerun |
| `calibration` | 1 | Sonnet 5 @ low | Profile generation from the blind exercise |
| `telemetry` | 3–4 | Haiku 4.5 @ low | Ban/betting logs → notes |
| `report_stats` | 11 | Sonnet 5 @ low | Aggregation |
| `report_replay` | 11 | Opus 4.8 @ high | Counterfactuals are real game-tree reasoning |

**Pool** (`models.pool`): the game pieces. Cost/runtime/tokens are *scored*,
so tier spread is the game — the contract constrains shape
(`min_cost_tiers: 3`), never picks. Sample sets `fable_legal: false`
(conservative default; flip it via PR if premium pieces should be draftable).

## Phase gates

Each phase declares its `actors` (who runs it) and a `gate` (what must hold
before the next phase begins). Two invariants worth defending in review:

- **Deterministic work is `code`, never a model** — the lottery (5), budget
  accounting (6), and reveals (7) must stay model-free.
- **Human phases gate on *both* players** — no phase advances on one player's
  submission.

| # | Phase | Actors | Gate |
|---|---|---|---|
| 0 | draft | human | both players' pool suggestions submitted |
| 1 | preference_calibration | human + calibration slot | both profiles generated |
| 2 | candidate_pool | code | `pool_size` configurations published |
| 3 | ban_phase | human + telemetry slot | bans complete; `finalists` remain |
| 4 | betting_phase | human + telemetry slot | both players' priors recorded |
| 5 | assignment_lottery | code | assignments generated per `assignment_method` |
| 6 | prompt_budget | human + code | both allocations committed |
| 7 | information_purchases | code | window closed: both pass or budget spent |
| 8 | execution | **models.pool** | all prompts executed; artifacts captured |
| 9 | blind_jury | jury_panel + adjudicator | all reviews in; disagreements adjudicated |
| 10 | human_appeals | human + appeals/rerun slots | tokens exhausted or both waive |
| 11 | final_report | report slots | report delivered; retrospective opened |

## Weights

Sample jury weights (each group must sum to 1.0 — checked at PR review):

| Dimension | Weight |
|---|---|
| output_quality | 0.35 |
| evidence | 0.20 |
| cost_efficiency | 0.15 |
| citations | 0.10 |
| reproducibility | 0.10 |
| confidence_calibration | 0.10 |

Add a weight group the same way you add a slot — e.g. a `replay` group if the
counterfactual analysis should score dimensions differently.

## Params worth arguing about

- `adjudication_trigger`: sample says a **score spread ≥ 3 on a 10-scale**
  escalates to the adjudicator. `verdict_flip` is the schema's other option.
- `assignment_method`: sample is `fully_random` (simplest for game 1);
  `snake_draft`, `weighted_lottery`, `auction`, `hidden_until_execution` are
  contract-legal.
- `information_purchases`: reveal prices in budget points — currently priced
  so the rubric (30) costs 3× a context peek (10).
- `pool_size: 8` / `finalists: 5` — note: 8 → 5 means 3 bans, so the ban
  order won't split evenly between two players. Change either number or accept
  the asymmetry (going first has value).

## Extending

1. Add the slot/param/weight group to `game.config.json`.
2. Add it to `contract.schema.json` (new harness slots that follow the
   `{model, effort}` shape are already legal via the schema's open
   `additionalProperties` — named entries are only needed to make them
   *required*).
3. Open a PR; the other player approves; done.

Candidate add-on slots from the draft discussions, take or leave:
`color_commentary` (Haiku narrating lottery/bans, non-authoritative),
`devils_advocate` (extra jury seat prompted to refute consensus),
`second_adjudicator` (Fable tie-breaker).

## Cost reference (July 2026, for review context — not rules)

Fable 5 $10/$50 · Opus 4.8 $5/$25 · Sonnet 5 $3/$15 ($2/$10 intro through
2026-08-31) · Haiku 4.5 $1/$5 per million tokens in/out. One 8K-in/1K-out jury
review ≈ $0.13 Fable / $0.065 Opus / $0.026 Sonnet-intro; the sample config's
full jury pass (5 finalists × 10 prompts × 3 reviewers) ≈ $3.90, roughly half
that with `cache_jury_packets: true`.
