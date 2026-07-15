# Lesson Summary Page — Design Spec

Date: 2026-07-15

## Goal

Produce a single static reference page, `docs/index.html`, that summarizes the
Computer Vision lesson (3.8) for quick review — structured around the lesson's
own workflow, with a hero pipeline visualization at the top and tabbed
navigation for the material. No source materials in the repo are modified;
this is a purely additive, read-only summary artifact.

## Source Materials

- `README.md` — lesson objectives, lesson plan/timing table
- `lesson.md` — lesson brief/prep instructions
- `studies.md` — self-study prep guide
- `assignment.md` — assignment brief + starter code
- `reference.md` — external reference links
- `notebooks/computer_vision_lesson.ipynb` — the primary content (65 cells):
  markdown explanations, code, and 12 embedded execution-result images
- `assets/*` — diagrams and sample images used by the notebook/README

## Output

- `docs/index.html` — single self-contained file (inline CSS + vanilla JS,
  no CDN/build step required)
- `docs/img/` — images referenced by `index.html`:
  - Copied as-is from `assets/`: `cnn.png`, `convolution.gif`, `pooling.png`,
    `image_segmentation.jpeg`, `cute_cat.jpg`, `mbs.png`
  - Extracted from the notebook's embedded PNG execution outputs (the actual
    results students see when running the notebook): PIL grayscale, OpenCV
    grayscale, HSV conversion, Gaussian blur (original vs. blurred), Sobel
    X/Y, Canny edges, binary thresholding, SIFT features, CIFAR-10 prediction
    grid.

## Page Structure

**Header**: lesson title, one-line description, lesson objectives (from
README.md).

**Hero pipeline diagram**: custom HTML/CSS diagram (no image dependency)
showing the lesson's workflow: `Raw Image → Classical Processing → CNN →
Training → Transfer Learning (ResNet) → Classification`. Each stage is
clickable and jumps to the corresponding tab.

**Top-level tabs** (mirrors the README Lesson Plan's Parts 1–4):

- `Overview` — objectives, hero diagram, lesson plan/timing table
- `Part 1: Image Processing` — subtabs: Intro to CV, Reading/Writing Images,
  Color Spaces, Classical Techniques (blur, edge detection, segmentation,
  feature extraction)
- `Part 2: CNN Architecture` — subtabs: Conv Layers, Pooling, FC Layers &
  Classification, Regularization (data augmentation, batch norm, dropout)
- `Part 3: Training` — subtabs: Model & Data Pipeline, Training Loop,
  Evaluation & Predictions
- `Part 4: Transfer Learning` — subtabs: Concept, ResNet, Feature Extraction
  vs. Fine-Tuning
- `Resources` — links to `studies.md`, `lesson.md`, `assignment.md`,
  `reference.md`; lesson plan timing table

**Content style**: condensed prose distilled from the notebook's markdown
cells plus the real result images described above. No code blocks — this is
a concept + visual reference page, not a copy of the notebook.

## Constraints

- No files outside `docs/` are modified.
- No external network dependencies (fonts, JS libraries, CDNs) — the page
  must work fully offline.
- Not optimized for mobile, but must not visually break on a narrower
  desktop window.

## Testing

Open `docs/index.html` directly in a browser and verify:
- Tab and subtab switching works
- Every referenced image in `docs/img/` renders
- Hero diagram stage links jump to the correct tab
- Resource links resolve correctly relative to `docs/`
