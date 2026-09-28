# Tradeoff — Decision Lab

**An explainable, local-first decision workspace for comparing options, stress-testing assumptions, and understanding why a ranking changes.**

Tradeoff rebuilds an old movie tutorial repository into a deliberately different product: a compact decision-support system with deterministic scoring, editable scenarios, evidence confidence, sensitivity analysis, resilient local persistence, accessible analytical UI, and an optional multi-provider AI copilot.

> **Decision support, not decision replacement.** The product helps a person inspect trade-offs; it never makes the final choice for them.

## Product loop

```text
Frame the decision
      ↓
Define options + criteria
      ↓
Score evidence + confidence
      ↓
Compare ranked options
      ↓
Switch priority scenarios
      ↓
Stress-test sensitivity
      ↓
Investigate weak assumptions
```

## Why this project is different

Most comparison demos stop at a weighted average. Tradeoff deliberately keeps three signals separate:

1. **Score** — a deterministic 0–100 weighted preference result.
2. **Evidence confidence** — how strong the evidence behind the inputs is, shown independently.
3. **Sensitivity** — how much one criterion weight must move before the current leader changes.

A high-scoring option can still rely on weak evidence, and a narrow lead can be fragile even when every input is certain. Folding those concepts into one opaque number would hide useful uncertainty.

## Capabilities

- editable decision framing, option names and criterion definitions
- 2–8 options and 2–8 criteria with referentially safe add/remove flows
- 0–10 score matrix with per-cell `low / medium / high` evidence confidence
- normalized scenario weights without mutating evidence scores
- deterministic ranking and contribution breakdown
- bounded single-criterion sensitivity scan
- top trade-off explanation between leading options
- versioned, Zod-validated local persistence
- integrity checks for IDs, scenario references and score/weight coverage
- explicit JSON export/import
- responsive analytical UI with reduced-motion and forced-colors support
- unit, browser and automated accessibility verification

## Optional Decision Copilot

AI is a language layer around the deterministic product, **not** its decision engine.

```text
Tradeoff UI
    ↓ explicit user request
/api/ai — server-only gateway
    ↓
Gemini 3.8 Flash
    ↓ provider failure / timeout / invalid output
OpenRouter openrouter/free
    ↓ failure
bounded AI error; local analysis remains available
```

### Structure a rough decision

A free-form description can become an **editable draft** containing framing, options, criteria, suggested weights, and cautions. Applying that draft is an explicit user action; every score still starts neutral and requires user evidence.

### Challenge assumptions

Review mode can surface blind spots, questions, assumptions and the next useful evidence-gathering step. It receives the visible decision frame, numeric scores and confidence labels — **not local evidence notes**.

### AI safety and reliability boundaries

- provider keys exist only server-side
- no AI call on page load, scoring, persistence or scenario changes
- strict request schemas and structured response schemas
- server-side Zod validation after provider output
- user decision text treated as untrusted data for prompt-injection isolation
- Gemini Interactions requests are stateless (`store: false`)
- same-origin browser requests are enforced at the AI gateway
- no tools, browsing or external actions exposed to the model
- bounded request size, timeout and demo rate limiting
- Gemini primary + OpenRouter fallback
- complete deterministic functionality when AI is unavailable

See [docs/AI.md](docs/AI.md).

## Architecture

```text
React components
      ↓
App orchestration / explicit state ownership
      ↓
┌───────────────┬────────────────┬─────────────────┐
│ pure domain   │ storage adapter│ AI client       │
└───────────────┴────────────────┴─────────────────┘
                                   ↓
                                /api/ai
                           Gemini → OpenRouter
```

The domain module imports no React, DOM, storage, network or AI code. Persistence and AI are adapters around the core rather than dependencies of it.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Stack

### Runtime

- React 19.3
- strict TypeScript 6
- Vite 8.3
- Zod 4
- semantic HTML
- modern CSS
- Web Storage API
- native file APIs

### Verification

- Vitest
- Testing Library
- Playwright
- axe-core
- ESLint with type-aware rules
- Prettier
- GitHub Actions

### Deployment

- Vercel static frontend
- Vercel Functions at `/api/health` and `/api/ai`
- `GEMINI_API_KEY` primary provider secret
- `DEEPSEEK_API_KEY` primary fallback provider secret
- `OPENROUTER_API_KEY` secondary fallback provider secret
- production smoke verification via `npm run smoke:deployment -- <url>`

See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for the production runbook.

## Scoring model

For option `o` and criterion `c`:

```text
weighted contribution = (score[o,c] / 10) × normalizedWeight[c]
overall score = Σ weighted contribution
```

Confidence is intentionally **not multiplied into score and does not break score ties**. It is reported separately as evidence quality.

## Sensitivity model

Tradeoff varies one criterion at a time while proportionally rescaling the remaining weights. It searches outward from the current weight and reports the smallest tested shift that changes the first-ranked option.

This is a bounded local stress test, not a prediction or statistical certainty claim.

## Local development

Requirements: Node.js 24.x.

```bash
npm ci
npm run dev
```

The core product requires no secrets. For deployed AI capabilities configure at least one server-side key, preferably both:

```text
GEMINI_API_KEY
DEEPSEEK_API_KEY
OPENROUTER_API_KEY
TRADEOFF_ALLOWED_ORIGIN   # optional
TRADEOFF_SITE_URL         # optional
```

Never expose provider secrets through a `VITE_` variable.

## Verification

```bash
npm run check
npm run test:e2e
npm run smoke:deployment -- https://tradeoff-decision-lab.vercel.app
```

CI uses the committed lockfile and runs formatting, linting, type checking, unit tests, production build, Playwright journeys and axe analysis.

## Deliberate trade-offs

- **No backend database:** private decision state is local-first for this portfolio product.
- **No Redux/Zustand:** state scope does not justify another runtime abstraction.
- **No chart package:** visualizations remain purpose-built and inspectable.
- **No AI ranking:** LLMs help with language and critique, not preference arithmetic.
- **No fake certainty:** score, confidence and sensitivity remain separate concepts.
- **No provider SDK in the browser:** one small server boundary protects secrets and keeps providers replaceable.

## Origin

The repository originally contained a small Create React App movie/watchlist exercise. Its Git history is preserved, but the current problem domain, product, architecture, design language and quality strategy are intentionally new.
