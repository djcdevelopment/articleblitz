# ArticleBlitz

A prompt tournament played as a game of incomplete information. Two players,
one repo, and a gentlemen's agreement.

## The Gentlemen's Agreement

Like two people playing billiards in a random poolhall: the specific rules
matter less than both players agreeing to them before the break.

1. **Rules live in this repo** as JSON configs under `rules/`, validated by a
   schema contract.
2. **Any player may propose a change** — branch, edit, open a pull request.
3. **A change lands only when the other player approves the PR.** No
   self-merges on rule changes.
4. **Ratification:** each player flips their own flag in
   `rules/ratification.json` (via PR, approved by the other). When both flags
   are `true`, the commit is tagged `game-<n>-locked`.
5. **After lock, no rule changes for that game.** Ideas that come up mid-game
   go to `RETROSPECTIVE.md`, not into the config.
6. **Post-game retrospective** lists future improvements; each becomes a PR
   before the next game. Version bumps, both flags reset to `false`, repeat.

## New here? Start with onboarding

Open **<https://djcdevelopment.github.io/articleblitz/>** — mobile-first
walkthrough of the premise, every phase (with opt-in samples), how to agree,
and how to submit change requests via your agent. (Source:
[`docs/index.html`](docs/index.html); Pages currently builds from the
`rules/v0.3-draft` branch — switch the Pages source to `main` after the first
PR merges.)

**Agents:** read [`AGENTS.md`](AGENTS.md) before your first write. It is the
binding protocol — branch patterns, preflight checks, merge rules, and what
to refuse.

## Repo layout

```
docs/index.html          onboarding — mobile-first, premise + samples + how-to
AGENTS.md                binding protocol for player agents (legalese + JSON)
rules/
  README.md              overview — slots, weights, phase gates
  contract.schema.json   the contract: JSON Schema every config must satisfy
  game.config.json       the game: models, phases, weights, params
  ratification.json      the handshake: ready_to_play flags + lock state
RETROSPECTIVE.md         post-game notes → next game's PRs
```

## Players

- djm (djcdevelopment)
- durracktu
