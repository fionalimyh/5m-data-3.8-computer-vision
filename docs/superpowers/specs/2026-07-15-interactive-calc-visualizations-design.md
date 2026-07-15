# Interactive Calculation Visualizations — Design Spec

Date: 2026-07-15

## Goal

Replace three plain-text explanations in `docs/index.html` with interactive,
self-contained visualizations so the underlying calculations (CNN output
size, backprop's chain rule, why batch norm helps) are learned by
manipulating them rather than reading formulas. No external libraries or
CDNs — vanilla JS + inline SVG, consistent with the page's existing
single-file, offline-friendly nature.

## Source Materials

- `3.8 - Computer Vision DS5.pdf`, slide 34 ("Spatial Calculation") — the
  `n_out = floor((n_in + 2p - k)/s) + 1` formula and worked LeNet-5 example.
- `3.8 - Computer Vision DS5.pdf`, slides 37–39 ("Batch Normalisation" /
  "Why Batch Normalisation" / "Why Batch Normalisation works") — the
  easier-to-converge loss-surface comparison, sigmoid saturation, and
  regularizing-effect distribution scatter, plus the internal-covariate-shift
  explanation already in `docs/index.html`.
- User-supplied reference image "Understanding Backpropagation: The Chain
  Rule at a Hidden Node" (not present anywhere in this repo or the PDF —
  confirmed by grep and a full 49-slide visual scan). Its content (chain
  rule formula, detailed output-neuron/loss diagram, three intuitive
  component graphs) is reconstructed from general backprop math, not copied
  verbatim, and cross-checked conceptually against
  `5m-data-3.7-neural-network-deep-learning/docs/index.html`'s existing
  (static-image) backprop section and https://poloclub.github.io/cnn-explainer/
  for interaction style.

## Non-goals

- No changes to `docs/img/*` or any notebook/PDF asset.
- No new external dependencies (charting libraries, MathJax, etc.).
- Not reproducing the NTU-branded slide screenshots or the user's reference
  image directly — only the underlying concepts, redrawn as original SVG.

## Architecture

Each widget is a self-contained `<div class="interactive-viz" id="...">`
block with its own inline SVG and a small IIFE-scoped JS module appended to
the existing `<script>` tag at the bottom of `docs/index.html`. New shared
CSS classes (`.viz-controls`, `.viz-stage`, `.viz-readout`, `.viz-btn`) reuse
the page's existing custom properties (`--accent`, `--accent-bg`, `--bg-alt`,
`--border`, `--text`, `--text-muted`) so the widgets automatically follow
light/dark mode like the rest of the page. Each module queries its own
container by `id` and wires up its own listeners — no global state, no
interference with the existing tab/subtab click handlers already in the
file.

Widgets render their initial state on page load (not only after first
interaction), so a learner who never touches a slider still sees a
meaningful, labeled diagram — sliders/toggles are progressive enhancement
for a concept that already stands on its own visually.

## Component 1 — Spatial Calculation Explorer

**Placement:** Part 2 → Conv Layers subtab, replacing the current static
"Computing Output Size" paragraph (keeps the `n_out` formula text as a
caption, removes the hand-written LeNet-5 walkthrough sentence since the
widget now demonstrates it live).

**Controls:** four stepper inputs (+/− buttons, not free-drag sliders, since
only small integers are meaningful):
- `n_in`: 4–32 (default 8, small enough to render every cell)
- `k` (kernel size): 1, 3, 5, 7 (odd only, matches the "Kernel Size
  Trade-offs" text already on the page)
- `p` (padding): 0–3
- `s` (stride): 1–3

**Live output:** the formula `n_out = floor((n_in + 2p − k) / s) + 1`
recomputed on every change, shown as a large number, plus an SVG that draws:
- the input grid as an `n_in × n_in` cell grid (padding cells drawn as
  dashed-border extension cells around it)
- one highlighted `k × k` window on the input, with a "Step" button that
  advances the window by `s` cells (wrapping to the next row at the edge) so
  users can see the stride in action
- the resulting output grid (`n_out × n_out`), with the cell corresponding
  to the current window position highlighted in sync

Grids above ~20×20 total cells switch to an abstracted view (a single
rectangle labeled with its dimensions instead of per-cell rendering) so
large `n_in` values don't produce thousands of SVG nodes.

**Preset button:** "Load LeNet-5 CONV1 example" sets `n_in=32, k=5, p=0,
s=1` (matching the worked example already in the text), confirming
`n_out=28`.

## Component 2 — Backprop Chain Rule Walkthrough

**Placement:** Part 3 → Training Loop subtab. The existing one-paragraph
explanation is kept as an intro sentence; this widget follows it, replacing
the current passing mention of "backpropagate gradients" with a worked,
manipulable example.

**Editable inputs:** three inputs `i₁, i₂, i₃`, three weights `w₁, w₂, w₃`,
a `bias`, and a target `y` (small number fields, sane defaults e.g.
i=[0.5,0.8,0.2], w=[0.4,-0.3,0.6], bias=0.1, y=1).

**Live forward pass diagram (SVG):** three input nodes → weighted edges →
summing junction `Σ` (shows `Z = Σ wᵢiᵢ + bias` computed live) → activation
`σ(Z)` (sigmoid, computed live) → output `ŷ` → `Loss = ½(y − ŷ)²` (computed
live). Every node displays its current numeric value, recomputed on any
input change.

**Step-through control:** a "Step through backprop" button advances a
4-state walkthrough (Reset / Step 1 / Step 2 / Step 3 / Total), each step
highlighting the relevant node/edge in a distinct color (red/blue/green,
echoing the reference image's convention) and revealing:
1. `∂Loss/∂Activation = −(y − ŷ)` (red) — highlights the Loss↔ŷ edge
2. `∂Activation/∂Z = ŷ(1 − ŷ)` (blue) — highlights the σ↔Z edge
3. `∂Z/∂w₁ = i₁` (green) — highlights the Σ↔w₁ edge
4. Total: `∂Loss/∂w₁ = [step1] × [step2] × [step3] = <computed number>`,
   with each bracketed term shown with its live-computed value substituted

Only `w₁`'s gradient is walked through in detail (matching "the chain rule
at a hidden node" framing of the reference); a footnote notes `w₂`/`w₃`
follow the same pattern with `i₂`/`i₃` swapped in.

**Three linked mini-graphs**, redrawn (not copied) as small inline SVG line
charts, each with a dot that moves to reflect the current live values:
- Loss vs. Activation, with a tangent line at the current `ŷ` (slope =
  step 1's value) — "steep = urgent weight change"
- Sigmoid curve with the current `Z` marked — shaded flat zones at the
  tails (`|Z|` large) vs. the steep active zone in the middle, labeling
  which zone the current `Z` falls in
- `Z` vs. `w₁` (linear, slope = `i₁`) — "slope = input value"

## Component 3 — Why Batch Norm Visualizer

**Placement:** Part 2 → Regularization subtab, directly after the existing
Batch Normalization paragraphs (text kept as-is; widget illustrates it).

**Control:** a single "Without / With BatchNorm" toggle switch. No numeric
inputs here — there's no formula to manipulate, so this widget redraws
between two fixed illustrative states rather than computing anything live.

**Redraws on toggle:**
1. **Loss-contour path** — a small SVG with elliptical contour lines and an
   animated dot: "Without" traces a jagged zig-zag path toward the center
   (CSS-animated stepped path), "With" traces a smooth, mostly-direct path.
2. **Sigmoid saturation** — the same sigmoid curve as Component 2's mini
   sigmoid, but here plotting a small cluster of ~8 sample points: "Without"
   scatters them across the full x-range (several landing in the flat
   tails, labeled "gradient ≈ 0"), "With" clusters them in the steep middle
   region (labeled "gradient is large").
3. **Regularizing effect** — two small scatter plots side by side (kept
   static per state, just shown/hidden by the toggle): a wide uniform
   scatter ("full training set") vs. a tighter, color-grouped scatter
   ("batch training set"), illustrating the mild regularizing effect
   mentioned in the existing text.

## CSS Additions

New rules appended to the existing `<style>` block (no restructuring of
current rules):
```
.interactive-viz { ... }      /* container: border, radius, padding, matches .card */
.viz-controls { ... }         /* flex row of stepper/number inputs */
.viz-stepper { ... }          /* +/- button pair with numeric readout */
.viz-btn { ... }              /* step/toggle/preset buttons, accent-colored */
.viz-svg-stage { ... }        /* SVG container, max-width: 100%, responsive */
.viz-readout { ... }          /* large live-computed number display */
.viz-mini-graph { ... }       /* small linked line-chart SVGs, flex row on desktop, stack on mobile */
```
All colors reference existing custom properties — no new hard-coded hex
values — so dark mode (`prefers-color-scheme: dark`) is automatically
correct.

## Testing / Verification

No test framework exists for this static page. Verification is manual:
1. Open `docs/index.html` directly in a browser (`file://`) after edits.
2. For each widget: exercise every control (steppers to their min/max,
   step-through to completion, toggle both states) and confirm computed
   numbers match hand-calculated expected values for at least 2 input
   combinations each.
3. Resize the browser window / use devtools mobile emulation to confirm the
   SVGs stay responsive (no horizontal overflow).
4. Toggle OS/browser dark mode and confirm all three widgets remain legible
   (text contrast, SVG stroke colors).
5. Tab through each widget's controls with keyboard only, confirming focus
   order and that buttons are reachable (native `<button>`/`<input>`
   elements only, no custom-only-mouse controls).

Since no headless browser with working shared libs is available in this
environment (confirmed earlier this session), verification is a manual
checklist run by whoever implements the change, plus a request to the user
to spot-check in their own browser before considering this done.
