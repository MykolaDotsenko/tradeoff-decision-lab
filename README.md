# Tradeoff — Decision Lab

**An explainable decision workspace for comparing options, stress-testing assumptions and seeing why a ranking changes.**

[**Open the live app →**](https://tradeoff-decision-lab.vercel.app/) ·
[Architecture](docs/ARCHITECTURE.md)

> **Decision support, not decision replacement.** Tradeoff helps inspect trade-offs; it does not make the final choice for the user.

## The model

Most comparison tools collapse everything into one weighted score. Tradeoff keeps three different questions separate:

1. **Score** — how well an option fits the current priorities.
2. **Evidence confidence** — how strong the evidence behind those inputs is.
3. **Sensitivity** — how much a criterion weight has to move before the ranking changes.

A high score can still rest on weak evidence. A narrow lead can still be fragile. Those signals remain visible instead of being blended into one opaque number.

```text
frame the decision
      ↓
options + criteria
      ↓
scores + evidence confidence
      ↓
scenario weights
      ↓
ranking + contribution breakdown
      ↓
sensitivity test
      ↓
investigate weak assumptions
```

## What the app supports

- 2–8 options and 2–8 criteria;
- editable 0–10 score matrix;
- low / medium / high evidence confidence per score;
- multiple priority scenarios with normalized weights;
- deterministic ranking and contribution breakdown;
- bounded single-criterion sensitivity analysis;
- explanation of the leading trade-off;
- versioned Zod-validated local persistence;
- JSON export/import;
- responsive analytical UI with reduced-motion and forced-colors support.

## Scoring

For option `o` and criterion `c`:

```text
weighted contribution = (score[o,c] / 10) × normalizedWeight[c]
overall score = Σ weighted contribution
```

Confidence is **not** multiplied into the score and does not break ties. It is shown separately as evidence quality.

## Sensitivity

Tradeoff varies one criterion at a time while proportionally rescaling the others. It reports the smallest tested change that flips the first-ranked option.

This is a local stress test of the current assumptions, not a forecast or statistical certainty claim.

## Optional Decision Copilot

AI is a language layer around the deterministic model, not the ranking engine.

It can:

- turn a rough description into an editable decision draft;
- suggest questions, blind spots and evidence to gather next.

It cannot silently change scores or make the decision.

Provider keys stay server-side behind `/api/ai`; model output is schema-validated before it reaches the UI, and the core application remains fully usable when AI is unavailable.

See [AI notes](docs/AI.md).

## Architecture

```text
React UI
   ↓
application state / orchestration
   ↓
┌──────────────┬────────────────┬──────────────┐
│ pure domain  │ storage adapter│ AI client    │
└──────────────┴────────────────┴──────────────┘
                                  ↓
                               /api/ai
```

The domain module has no React, DOM, storage, network or AI dependency.

## Stack

**Runtime**

- React 19
- TypeScript 6 strict
- Vite 8
- Zod 4
- Web Storage
- Vercel Functions for optional AI/health endpoints

**Verification**

- Vitest
- Testing Library
- Playwright
- axe-core
- ESLint
- Prettier
- GitHub Actions

## Deliberate trade-offs

- no backend database — private decision state stays local to the browser;
- no Redux/Zustand — state scope does not justify another runtime abstraction;
- no chart package — visualizations are purpose-built;
- no AI ranking — arithmetic remains deterministic and inspectable;
- no fake certainty — score, confidence and sensitivity stay separate.

## Run locally

Requires Node.js 24.x.

```bash
npm ci
npm run dev
```

The core product needs no provider secrets. Optional AI configuration is documented in [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

## Verification

```bash
npm run check
npm run test:e2e
npm run smoke:deployment -- https://tradeoff-decision-lab.vercel.app
```

## Origin

This repository began as a small React movie/watchlist exercise. The Git history is preserved, but the current product, architecture and problem domain are intentionally different.
