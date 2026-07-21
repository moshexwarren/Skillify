# FRONTEND-SUITE — Measurable Benchmarks

## The unlock
Open-ended frontend is NOT measurable. Frontend becomes measurable only against a **published interface contract**. The task prompt specifies routes, data-testids, and behaviors. Hidden Playwright specs test exactly that contract. We measure **contract conformance**, never "quality."

If a metric requires opinion, taste, or an LLM judge → it is not a metric. Delete it.

## Task prompt MUST contain a contract block
```md
## Interface contract (implement exactly — automated tests depend on it)
Routes: /  /pricing  /signup
Required testids:
  [data-testid="nav"]
  [data-testid="pricing-toggle"]
  [data-testid="plan-card"]  (exactly 3)
  [data-testid="cta"]
Behaviors:
  - Clicking pricing-toggle switches all plan-card prices monthly<->yearly
  - Clicking cta navigates to /signup
  - Site must work at 360px, 768px, 1440px widths
```
No contract = no suite. Reject suite.json without a contract section in task.md.

## Measurables (all deterministic)

| # | Metric | Tool | Assertion / value |
|---|---|---|---|
| 1 | Functional pass rate | hidden Playwright specs vs contract | passed/total |
| 2 | Keyboard nav pass | Playwright: Tab order reaches contract elements, Enter/Space activates | binary per flow |
| 3 | Console errors | page.on('pageerror') + console type=error during specs | count |
| 4 | Failed requests | page.on('requestfailed') + responses ≥400 | count |
| 5 | axe violations | @axe-core/playwright per route | count by severity |
| 6 | Contrast failures | axe rule color-contrast | count |
| 7 | Semantic checks | DOM asserts: exactly one h1; no heading-level skips; <main>/<nav> present; img alt count; html[lang] | counts/binary |
| 8 | HTML validity | html-validate | error count |
| 9 | Responsive overflow | scrollWidth <= clientWidth at 360/768/1440 | binary ×3 |
| 10 | Element overlap | boundingBox intersection between contract elements at each viewport | count |
| 11 | RTL mirror pass | run specs dir=ltr and dir=rtl, compare getBoundingClientRect mirrored | binary (RTL suites only) |
| 12 | Hardcoded values | static scan: hex colors, px font-sizes outside token file | count — label PROXY |
| 13 | Bundle size | dist JS+CSS bytes after build | KB |
| 14 | LCP / FCP / TBT / CLS / INP + perf/a11y/BP scores | Lighthouse, pinned Docker + pinned Chrome, fixed throttle, median of 5 | numbers |
| 15 | Dead internal links | crawl anchors, count non-2xx | count |
| 16 | Pixel diff | expect(page).toHaveScreenshot, pinned container fonts | % — REPRODUCTION TASKS ONLY (golden reference exists) |

## Hidden spec examples (checked into suites/<name>/hidden/)
```ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('contract: pricing toggle switches prices', async ({ page }) => {
  await page.goto('/pricing');
  const card = page.getByTestId('plan-card').first();
  const before = await card.textContent();
  await page.getByTestId('pricing-toggle').click();
  await expect(card).not.toHaveText(before!);
});

test('contract: cta navigates to /signup', async ({ page }) => {
  await page.goto('/');
  await page.getByTestId('cta').click();
  await expect(page).toHaveURL(/\/signup/);
});

test('no horizontal overflow @360', async ({ page }) => {
  await page.setViewportSize({ width: 360, height: 800 });
  await page.goto('/');
  const ok = await page.evaluate(() =>
    document.documentElement.scrollWidth <= document.documentElement.clientWidth);
  expect(ok).toBe(true);
});

test('a11y: no serious/critical violations on /', async ({ page }) => {
  await page.goto('/');
  const r = await new AxeBuilder({ page }).analyze();
  const bad = r.violations.filter(v => ['serious','critical'].includes(v.impact ?? ''));
  expect(bad).toHaveLength(0);
});

test('zero console errors on /pricing', async ({ page }) => {
  const errs: string[] = [];
  page.on('pageerror', e => errs.push(e.message));
  page.on('console', m => m.type() === 'error' && errs.push(m.text()));
  await page.goto('/pricing');
  await page.getByTestId('pricing-toggle').click();
  expect(errs).toHaveLength(0);
});
```

## Environment (kills Lighthouse/pixel noise)
- Docker image pinned by digest, pinned Chromium, `--cpu-slowdown-multiplier=4`, fixed network throttle.
- Lighthouse: 5 runs, report median. Store env hash in bundle.
- Pixel diff: same container = same font rendering; maxDiffPixelRatio 0.01.

## Explicitly NOT measured (do not attempt)
Visual quality, design taste, "slop", spacing/typography quality, originality, code readability. Any collector needing an LLM or human opinion is out of scope.
# CREATIVE-SUITES — Frontend Task Archetypes (all deterministic)

Design rule: creativity lives in the TASK; measurement stays binary/countable. Every archetype produces ground truth via one of three mechanisms: **seeding** (we broke it, so we know), **fixtures** (we control the data, so we know edge cases), or **golden reference** (we built the answer, so we can diff).

Anti-gaming rule: every constraint metric is PAIRED with content-preservation hidden specs. (Can't win LCP by deleting the page — content specs must stay green.)

## Archetypes

| # | Archetype | Task | Ground truth | Measures |
|---|---|---|---|---|
| 1 | Broken-page repair | Working page seeded with N breaks (mobile overflow, dead toggle, keyboard trap, contrast fail, missing alt, z-index bug) | Seed manifest + hidden specs failing pre-fix | Fixed count /N, regressions on untouched specs |
| 2 | State-machine conformance | Build wizard/checkout defined as explicit state machine (states + transitions in contract) | Transition table | Hidden specs walk EVERY transition: passed/total = coverage |
| 3 | Data-fixture torture | Render provided JSON: 50 items incl. long names, missing images, zero price, empty array, Hebrew strings, `<script>` payload | Fixture is the truth | All rendered, no overflow, empty-state shown, **XSS escaped (script never executes)**, skeleton on delayed route |
| 4 | Chaos/resilience | Same app; harness intercepts routes: 500, timeout, offline, slow-3G | Route interception scripts | Error-state testid shown, retry works, zero console errors, loading indicator appears |
| 5 | Form gauntlet | Form with validation matrix (N invalid cases) | Matrix in suite | Each invalid → aria-invalid + error testid; valid → exactly 1 POST (double-submit blocked = request count); focus jumps to first invalid field; Enter submits |
| 6 | Constraint gauntlet | Build page under hard budgets | Budgets in contract | Bundle ≤ X KB, LCP ≤ Y ms, zero serious axe, works with JS disabled (content visible), print CSS (emulate media), prefers-reduced-motion → getAnimations() empty, dark mode → both schemes pass contrast |
| 7 | A11y remediation | Page with 25 seeded axe violations; fix without breaking function | Seeded violation list | Violations removed /25 + functional specs stay green |
| 8 | Performance rescue | Heavy page; get LCP under budget, content intact | Golden content specs | LCP delta + content-preservation pass |
| 9 | Keyboard-only mission | Complete full flow with keyboard only (no mouse events in spec) | Flow definition | Binary completion + focus-visible on each stop |
| 10 | Race/double-fire | Rapid double-click, back-button, concurrent fetches (delayed routes) | Request/DOM counts | Duplicate requests = 0, duplicate DOM items = 0 |
| 11 | 10k-row virtualization | Render 10k-row fixture smoothly | Fixture | DOM node count < threshold (proves virtualization), scroll-position restore, TBT budget |
| 12 | Reproduction clone | Clone provided reference page (we ship the golden build) | Golden reference | Pixel diff %, accessibility-tree snapshot diff vs reference (less brittle than pixels), Lighthouse parity |
| 13 | RTL flip mission | Take LTR page, deliver correct RTL | Dual-dir geometry | Mirrored getBoundingClientRect pass, overflow @360 in Hebrew fixture, Intl-formatted dates/numbers (regex on output) |
| 14 | Responsive breakpoint contract | Contract states what changes at each width (nav→burger @<768 etc.) | Breakpoint table | Per-breakpoint testid visibility asserts, overlap = 0, overflow = 0 |

## Why these are "creative" but still provable
- XSS escape, double-submit, keyboard-trap, JS-disabled, reduced-motion, dark-mode-contrast, virtualization-node-count: these FEEL like quality/craft — and each is one boolean/count.
- Archetypes 1, 3, 7 reuse the code-review seeding logic on frontend: known breaks in = detection/fix rate out.
- Archetype 2 (state machine) scales difficulty infinitely: more states = harder task, same measurement.

## Archetypes 15–24 (app-maturity dimensions)

| # | Archetype | Mechanism | Measures |
|---|---|---|---|
| 15 | Auth/session expiry | Mocked auth provider, token expires mid-flow (route interception + delays) | Redirect when unauthed, recoverable expiry state, pending form data preserved, no refresh-token infinite loop (request count cap), protected testid never visible while auth request pending |
| 16 | Permission matrix | Same UI × roles: admin/editor/viewer/anonymous | Forbidden controls hidden/disabled, forbidden API calls = 0 (request count), direct-URL blocked, readonly states render, role switch without reload bugs |
| 17 | Optimistic rollback | Route returns success / failure / modified canonical | Optimistic state immediate, rollback on fail, double-click no corruption, canonical wins, error toast testid, UI==server state at end |
| 18 | Persistence/reload | Interact → reload → reopen; inject corrupted localStorage | Filters/sort survive reload, draft preserved, corrupt storage no crash, URL↔UI state aligned |
| 19 | Deep-link routing | Open nested URLs directly | `/items/123?tab=x&filter=y` restores state, bad ID → 404/empty state, back/forward works, modal routes close, canonical URL preserved |
| 20 | Upload gauntlet | Mocked picker + endpoint: success/slow/fail/oversize/wrong MIME | Type+size enforced, progress shown, cancel works, retry works, no ghost attachment after fail, preview accessible |
| 21 | Time/timezone trap | `page.clock.install()` + per-context TZ + DST edge fixtures | Dates correct across TZ, relative times vs fixed clock, date input no day-shift, date sort correct, Intl output (regex) |
| 22 | Security boundary | Malicious fixtures + request inspection | `<script>` renders as text, external links rel=noopener (DOM check), user URLs sanitized, hidden admin fields absent from POST body, CSRF header present. Static grep for dangerouslySetInnerHTML = PROXY |
| 23 | Design-system migration | Legacy screen + existing DS; migrate | Required DS components imported, old classes = 0 (grep), token contract, behavior specs unchanged, pixel diff vs pre-migration golden within tolerance, no new duplicate component (jscpd — PROXY) |
| 24 | Loading/skeleton contract | Staged/slow/partial route responses | Skeleton testid < X ms, CLS during load ≤ budget, partial data no crash, loading ≠ empty state (distinct testids), error replaces skeleton, content appears post-resolve |

Implementation notes: 15–17, 19–20, 24 = pure route interception (fully deterministic). 21 = Playwright clock API. 18 = storage injection. 22–23 mix specs with static-grep PROXY checks — label them.

## META-RULE (suite validation gate)
Every suite must test ≥3 states per behavior: **happy path, hostile/edge input, recovery after failure.** Suites covering happy path only are rejected — that's how pretty demos win and real products lose.

## Suite authoring checklist
1. Pick archetype. 2. Write contract (routes/testids/behaviors/budgets). 3. Build golden or seed manifest. 4. Write hidden specs covering 100% of contract + the 3-state meta-rule. 5. Validation gate: golden passes all specs; seeded version fails exactly the mapped ones. 6. Pin container.
