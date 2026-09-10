# HANDOFF — Chalk & Thread Suit Configurator

## What this is
A kiosk-style custom suit configurator for a budget custom-suit platform. Business model: $250 base suit, customization-first (lapel, buttons, lining, monogram, collar name, photo lining), manufactured in China with **deliberate extra fabric allowance** — the customer's own local tailor does final alterations. We never tailor. Sold online and on in-store iPads.

Working brand: **Chalk & Thread** (placeholder — keep it swappable, single wordmark in header).

## Files & where to find them
Both files live together in the project root (wherever this handoff was dropped — likely `~/Downloads` or the repo folder you were started in):
- `handoff-suit-configurator.md` — this document. Re-read it at the start of every session and after every phase.
- `suit-configurator.html` — the entire current build. This is the only source file. If it's not in the working directory, search for it: `find ~ -name "suit-configurator.html" 2>/dev/null` before doing anything else. Never start from scratch — if you can't find the file, stop and ask.

Work directly on `suit-configurator.html`. Before each phase, copy it to `backups/suit-configurator.pre-phase{N}.html`.

## Current state
One self-contained file: `suit-configurator.html`. No build step, no dependencies except Google Fonts (Cormorant Garamond + Inter). Everything inline. **Keep it single-file** unless a phase below explicitly says otherwise.

### Architecture inside the file
- **Design tokens** in `:root` — ink navy `#1b2a41`, chalk `#fbfaf6`, paper `#f3f1ea`, thread gold `#b8860b`, chalk-mark red `#c0392b`. Tailor's-ticket aesthetic. Do not drift toward generic SaaS cards.
- **SVG illustrators**: `jacketSVG(o)`, `backSVG(o)`, `trouserSVG(o)` — parametric line drawings (white fill, ink stroke, red accent on the feature being chosen). Option thumbnails AND the live preview all come from these. Any new style option must get a drawing, not a stock image.
- **State**: flat `state` object + `DEFAULTS`, `PRICES` map (upcharges), `NAV` array (categories → subtabs), `OPTGROUPS` (option definitions with `svg:` or `dot:` thumbnails).
- **Navigation**: GC-Clothiers-style — top category tabs with change-count badges, pill subtabs, sticky bottom bar with live total + Back/Continue + "Leave the styling to us" (resets styling to house standard, keeps fit + personalization, jumps to review).
- **Fit intake**: two methods — tailor measurement sheet (9 fields), or closet references (unlimited garments: brand+size, flat measurements pit-to-pit/shoulder/sleeve, fit verdict).
- **Alterations feedback form** on the review step — order #, cost, per-measurement deltas, notes, $20 credit hook. This is the sizing-engine training data. Never remove it.
- Order placement and alteration submit are **stubbed with alert()** — marked in code where manufacturer API / Stripe go.

## Rules
- Phased execution. Finish a phase fully before starting the next. No half-wired features.
- Single-file HTML per phase output unless the phase says otherwise.
- Preserve the honest-fit disclosure text on review ("arrives wearable, not finished... typically $40–90 at your tailor"). It's the expectation-setting that makes the model work.
- Every price change flows through `PRICES` only.
- iPad kiosk is a first-class target: touch targets ≥44px, works offline after first load where possible, no hover-dependent UI.

## Phases (in order)

### Phase 1 — Cloth-aware live preview
Fill the live preview jacket with the selected cloth color (`CLOTHS` map exists). Add subtle texture hints per cloth (herringbone = fine diagonal hatch pattern via SVG `<pattern>`, sharkskin = slight two-tone). Lining peek: show selected lining color inside the open jacket front. Keep thumbnails white/line-drawing.

### Phase 2 — Manufacturer spec export
On "Place order," generate a complete spec sheet: JSON (machine) + printable HTML view (human). Include every state field, all measurements or closet references, monogram/collar text verbatim, photo filename, and computed allowance instructions per garment zone (e.g. `waist: +1.5" allowance`, `sleeve: leave 1" hem`). Download as `CT-{orderNum}.json` and open print view. This is what goes to the China factory — zero ambiguity.

### Phase 3 — Persistence + kiosk mode
LocalStorage save/resume of in-progress configs. "Start over" control. A `?kiosk=1` URL param: hides hero, auto-resets to step 1 after order completion + 60s idle, blocks navigation away. In-store iPad runs this.

### Phase 4 — Stripe + backend seam
Split-ready: extract a `submitOrder(spec)` function that POSTs to `/api/orders` (endpoint configurable via one const at top of file). Stripe Checkout redirect on success path, graceful offline queue for kiosk. Backend itself is out of scope — just make the seam clean.

### Phase 5 — Sizing engine v0
From closet references, compute estimated measurements: flat measurements ×2 for circumferences, verdict adjustments (snug +0.75", loose −0.75"), height/weight sanity bounds. Show the estimate to the user for confirmation before order. Log predicted-vs-alteration deltas from the feedback form into the order record — this is the bias-correction dataset.

## Definition of done per phase
Open the file in a browser, walk hero → fit → all style tabs → personalization → review → order, on desktop AND 390px mobile width. No console errors. Total updates correctly with every pick. Then stop and report.
