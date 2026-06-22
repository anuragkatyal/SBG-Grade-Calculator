# Standards-Based Grade Calculator

A single-page, dependency-free web app that lets students see their projected
final letter grade under a **standards-based grading (SBG)** scheme. Students
drag three sliders — CORE Learning Targets mastered, OTHER Learning Targets
mastered, and Final Exam score — and the grade updates live, along with a
plain-language explanation of how it was computed.

> **Live site:** https://anuragkatyal.github.io/SBG-Grade-Calculator/

The entire app is one file: [`index.html`](index.html). HTML, CSS, and
JavaScript are inline, there is no build step, and the only external resource is
the Inter font loaded from Google Fonts.

---

## How the grade is calculated

There are two stages: a **base grade** from learning targets, then a **final
exam adjustment**. Both stages — and the rule lists shown to students — are
driven by two data tables near the top of the `<script>` block,
`BASE_GRADE_RULES` and `EXAM_RULES`. These are the single source of truth:
changing a rule there updates the calculation and the on-screen explanation
together.

### 1. Base grade from Learning Targets (LTs)

`calculateStandardsGrade(core, other)` walks `BASE_GRADE_RULES` top to bottom
and returns the first matching rule's grade. It maps the number of mastered
CORE LTs (0–10) and OTHER LTs (0–6) to a letter:

| CORE LTs | OTHER LTs        | Grade |
|----------|------------------|-------|
| 10       | 4, 5, or 6       | A     |
| 10       | 2 or 3           | B     |
| 10       | 0 or 1           | C     |
| 9        | 4, 5, or 6       | C     |
| 9        | 0–3              | D     |
| 8        | 5 or 6           | C     |
| 8        | 0–4              | D     |
| ≤ 7      | (any)            | F     |

### 2. Final exam adjustment

The base grade is then nudged up or down by `adjustGrade()` using the delta from
`EXAM_RULES` (looked up by `examAdjustment()`), based on the final exam score:

| Final exam | Letter-grade change |
|------------|---------------------|
| ≥ 90%      | up to **+3**        |
| 80–89.99%  | up to **+2**        |
| 70–79.99%  | up to **+1**        |
| 60–69.99%  | no change           |
| 50–59.99%  | up to **−1**        |
| 40–49.99%  | up to **−2**        |
| < 40%      | up to **−3**        |

"Up to" is enforced by clamping: grades never go above A or below F. The grade
order used for adjustment is `['F', 'D', 'C', 'B', 'A']` — note there is no
distinction between, say, A and A+; this is a 5-level scale.

---

## Using it locally

Because everything is inline and static, you can just open the file:

```bash
open index.html        # macOS
```

Or, if you prefer serving it over HTTP (closer to how it runs in production):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

No `npm install`, no bundler, no framework.

---

## Deployment

Deployment is automatic via GitHub Pages. The workflow in
[`.github/workflows/static.yml`](.github/workflows/static.yml) uploads the whole
repository and publishes it whenever you **push to `main`** (or trigger the
workflow manually from the Actions tab).

To deploy a change:

```bash
git add index.html
git commit -m "Update grading rules"
git push origin main
```

The Pages deployment runs on push and the live site updates within a minute or
two.

---

## Modifying it

Everything you'd want to change lives in [`index.html`](index.html). The most
common edits:

### Change the grading thresholds
Edit the `BASE_GRADE_RULES` table at the top of the `<script>` block. Each row is
`{ grade, core, other }`: `core` is the qualifying CORE count (or `'any'`),
`other` is the list of qualifying OTHER counts (or `'any'`). Rows are evaluated
top to bottom, first match wins, so keep the `F` catch-all row last. The
**Standards-Based Grading Scale** shown on the page is generated from this same
table — no need to edit any HTML.

### Change the final-exam impact rules
Edit the `EXAM_RULES` table — each row is `{ min, delta }`, evaluated high to low.
The **Final Exam Impact Rules** list on the page is generated from it.

### Change the number of CORE / OTHER targets
Set `max` on the `<input type="range">` elements (`id="coreTargets"`,
`id="otherTargets"`). The hashmarks read this `max` automatically, so the only
other thing to update is the `<label>` text ("out of 10" / "out of 6"). Remember
to adjust the `other` lists in `BASE_GRADE_RULES` if the OTHER range changes.

### Restyle
All styling is in the `<style>` block in `<head>`. The per-letter result colors
are the `.grade-A` … `.grade-F` gradient rules; the accent color used for
sliders and readouts is `#4f46e5`.

### Single source of truth
The two rule tables drive both the math and the rule lists students read, so the
displayed rules can't fall out of sync with the calculation. The letter-grade
order (used for the exam adjustment, the A→F display order, and the `grade-*` CSS
classes) all derives from one `GRADE_LADDER` constant.

---

## File structure

```
SBG-Grade-Calculator/
├── index.html                  # the entire app (HTML + CSS + JS)
├── README.md                   # this file
└── .github/workflows/static.yml  # GitHub Pages deploy workflow
```

## Accessibility notes

The result region uses `aria-live="polite"` so screen readers announce the new
grade as sliders move, and the range inputs have visible `:focus-visible`
outlines for keyboard users. Keep these in mind if you refactor the markup.
