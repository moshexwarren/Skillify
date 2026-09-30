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
