# Lesson Summary Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `docs/index.html`, a single self-contained static reference page summarizing the 3.8 Computer Vision lesson, with a hero pipeline diagram and tabbed/subtabbed navigation matching the lesson's Part 1–4 structure.

**Architecture:** One static HTML file with inline `<style>` and inline `<script>` (no build step, no CDN dependencies). Images live in `docs/img/`, sourced either by copying existing `assets/*` files or by extracting the notebook's embedded base64 PNG execution outputs. Content is condensed prose distilled from `README.md`, `lesson.md`, `studies.md`, `reference.md`, and the notebook's markdown cells — not a verbatim copy, not code.

**Tech Stack:** Plain HTML5, CSS (flexbox/grid), vanilla JS (no frameworks), Python 3 stdlib (`json`, `base64`) for one-time image extraction.

**Reference spec:** `docs/superpowers/specs/2026-07-15-lesson-summary-page-design.md`

---

## File Structure

- Create: `docs/img/` — directory for all images referenced by the page
- Create: `docs/index.html` — the single deliverable page
- No other files are modified. `assets/*` and `notebooks/*` are read-only sources.

---

### Task 1: Extract notebook embedded result images

**Files:**
- Create: `docs/img/gray_pil.png`, `docs/img/gray_opencv.png`, `docs/img/hsv.png`, `docs/img/gaussian_blur.png`, `docs/img/sobel.png`, `docs/img/canny.png`, `docs/img/threshold.png`, `docs/img/sift.png`, `docs/img/cifar_predictions.png`
- Create (temp): `/tmp/claude-1000/.../scratchpad/extract_images.py` (or run inline via `python3 -c`)
- Read: `notebooks/computer_vision_lesson.ipynb`

These are the notebook's actual execution outputs (matplotlib-rendered PNGs embedded as base64 in the `.ipynb` JSON), mapped by cell index to a descriptive filename:

| Cell | Content | Output file |
|------|---------|--------------|
| 14 | PIL grayscale conversion | `gray_pil.png` |
| 16 | OpenCV grayscale conversion | `gray_opencv.png` |
| 17 | HSV color space conversion | `hsv.png` |
| 22 | Gaussian blur (original vs. blurred) | `gaussian_blur.png` |
| 24 | Sobel X/Y edge detection | `sobel.png` |
| 26 | Canny edge detection | `canny.png` |
| 28 | Binary thresholding | `threshold.png` |
| 31 | SIFT keypoint features | `sift.png` |
| 53 | CIFAR-10 prediction grid | `cifar_predictions.png` |

- [ ] **Step 1: Create the image output directory**

```bash
mkdir -p "docs/img"
```

- [ ] **Step 2: Run the extraction script**

```bash
python3 -c "
import json, base64

nb = json.load(open('notebooks/computer_vision_lesson.ipynb'))

mapping = {
    14: 'gray_pil.png',
    16: 'gray_opencv.png',
    17: 'hsv.png',
    22: 'gaussian_blur.png',
    24: 'sobel.png',
    26: 'canny.png',
    28: 'threshold.png',
    31: 'sift.png',
    53: 'cifar_predictions.png',
}

for idx, fname in mapping.items():
    cell = nb['cells'][idx]
    written = False
    for out in cell.get('outputs', []):
        data = out.get('data', {})
        if 'image/png' in data:
            b64 = data['image/png']
            if isinstance(b64, list):
                b64 = ''.join(b64)
            with open(f'docs/img/{fname}', 'wb') as f:
                f.write(base64.b64decode(b64))
            print(f'wrote docs/img/{fname}')
            written = True
    if not written:
        raise SystemExit(f'ERROR: cell {idx} had no image/png output')
"
```

Expected output: 9 lines of `wrote docs/img/<name>`, no errors.

- [ ] **Step 3: Verify the 9 files exist and are non-empty**

```bash
ls -la docs/img/
```

Expected: `gray_pil.png`, `gray_opencv.png`, `hsv.png`, `gaussian_blur.png`, `sobel.png`, `canny.png`, `threshold.png`, `sift.png`, `cifar_predictions.png` each with a nonzero size.

---

### Task 2: Copy static asset images

**Files:**
- Create: `docs/img/cnn.png`, `docs/img/convolution.gif`, `docs/img/pooling.png`, `docs/img/image_segmentation.jpeg`, `docs/img/cute_cat.jpg`
- Read (source, unmodified): `assets/cnn.png`, `assets/convolution.gif`, `assets/pooling.png`, `assets/image_segmentation.jpeg`, `assets/cute_cat.jpg`

`assets/mbs.png` (the source image for the Gaussian blur/Sobel/Canny/threshold/SIFT examples) is intentionally **not** copied — the `gaussian_blur.png` extracted in Task 1 already shows that same original image side-by-side with the blurred result, so a separate copy would be unused. `assets/resnet.pbm` is also not used anywhere in this plan — it isn't referenced by any markdown file or notebook cell in the source materials.

- [ ] **Step 1: Copy the five files**

```bash
cp assets/cnn.png docs/img/cnn.png
cp assets/convolution.gif docs/img/convolution.gif
cp assets/pooling.png docs/img/pooling.png
cp assets/image_segmentation.jpeg docs/img/image_segmentation.jpeg
cp assets/cute_cat.jpg docs/img/cute_cat.jpg
```

- [ ] **Step 2: Verify source assets are untouched and copies exist**

```bash
ls -la assets/ docs/img/
```

Expected: `assets/` unchanged (8 original files still present, same sizes as before); `docs/img/` now has 14 files total (5 from this task + 9 from Task 1).

---

### Task 3: HTML skeleton, CSS, and tab-switching engine

**Files:**
- Create: `docs/index.html`

This task creates the full page shell — `<head>`, CSS design system, header, hero diagram markup + styling, tab/subtab nav, and the JS that drives tab switching. Content tabs (Task 4–9) will be inserted into the `<main id="tab-content">` placeholder this task creates.

- [ ] **Step 1: Write `docs/index.html` with the shell**

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Computer Vision — Lesson 3.8 Summary</title>
<style>
  :root {
    --bg: #ffffff;
    --bg-alt: #f6f8fa;
    --border: #e1e4e8;
    --text: #1f2328;
    --text-muted: #57606a;
    --accent: #2563eb;
    --accent-bg: #eff6ff;
    --code-bg: #f6f8fa;
    --radius: 8px;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #0d1117;
      --bg-alt: #161b22;
      --border: #30363d;
      --text: #e6edf3;
      --text-muted: #9198a1;
      --accent: #60a5fa;
      --accent-bg: #10203a;
      --code-bg: #161b22;
    }
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
    background: var(--bg);
    color: var(--text);
    line-height: 1.6;
  }
  .wrap { max-width: 980px; margin: 0 auto; padding: 32px 20px 80px; }
  header.page-header h1 { margin: 0 0 6px; font-size: 1.9rem; }
  header.page-header p.lede { color: var(--text-muted); margin: 0 0 20px; }
  .objectives { background: var(--bg-alt); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px 20px; margin-bottom: 32px; }
  .objectives h2 { margin-top: 0; font-size: 1rem; text-transform: uppercase; letter-spacing: .04em; color: var(--text-muted); }
  .objectives ul { margin: 0; padding-left: 20px; }

  /* Hero pipeline diagram */
  .pipeline { display: flex; flex-wrap: wrap; align-items: center; gap: 6px; margin: 0 0 40px; }
  .pipeline-stage {
    flex: 1 1 140px;
    background: var(--accent-bg);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 14px 12px;
    text-align: center;
    cursor: pointer;
    font-size: .9rem;
    font-weight: 600;
    color: var(--text);
    transition: transform .15s ease;
  }
  .pipeline-stage:hover { transform: translateY(-2px); border-color: var(--accent); }
  .pipeline-arrow { color: var(--text-muted); font-size: 1.3rem; flex: 0 0 auto; }

  /* Tabs */
  nav.tabs { display: flex; flex-wrap: wrap; gap: 4px; border-bottom: 2px solid var(--border); margin-bottom: 24px; }
  nav.tabs button {
    background: none; border: none; cursor: pointer;
    padding: 10px 16px; font-size: .95rem; font-weight: 600;
    color: var(--text-muted); border-bottom: 2px solid transparent; margin-bottom: -2px;
  }
  nav.tabs button.active { color: var(--accent); border-bottom-color: var(--accent); }
  nav.tabs button:hover { color: var(--text); }

  .tab-panel { display: none; }
  .tab-panel.active { display: block; }

  nav.subtabs { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 20px; }
  nav.subtabs button {
    background: var(--bg-alt); border: 1px solid var(--border); border-radius: 999px;
    cursor: pointer; padding: 6px 14px; font-size: .85rem; color: var(--text-muted);
  }
  nav.subtabs button.active { background: var(--accent-bg); color: var(--accent); border-color: var(--accent); }

  .subtab-panel { display: none; }
  .subtab-panel.active { display: block; }

  section.card { background: var(--bg-alt); border: 1px solid var(--border); border-radius: var(--radius); padding: 20px 24px; margin-bottom: 20px; }
  section.card h3 { margin-top: 0; }
  section.card img, section.card .img-row img { max-width: 100%; border-radius: 6px; border: 1px solid var(--border); }
  .img-row { display: flex; flex-wrap: wrap; gap: 12px; margin: 12px 0; }
  .img-row figure { margin: 0; flex: 1 1 260px; }
  .img-row figcaption { font-size: .8rem; color: var(--text-muted); margin-top: 4px; text-align: center; }
  code { background: var(--code-bg); padding: 1px 5px; border-radius: 4px; font-size: .9em; }
  table.plan-table { border-collapse: collapse; width: 100%; }
  table.plan-table th, table.plan-table td { border: 1px solid var(--border); padding: 8px 10px; text-align: left; font-size: .9rem; }
  table.plan-table th { background: var(--bg-alt); }
  a { color: var(--accent); }
  ul.resource-links li { margin-bottom: 6px; }
</style>
</head>
<body>
<div class="wrap">

  <header class="page-header">
    <h1>Computer Vision — Lesson 3.8</h1>
    <p class="lede">From pixels to predictions: classical image processing, CNNs, and transfer learning.</p>
  </header>

  <div class="objectives">
    <h2>Lesson Objectives</h2>
    <ul>
      <li>Explain the fundamentals of Computer Vision and apply classical image processing techniques, using Python libraries to analyze and enhance digital images.</li>
      <li>Construct and train Convolutional Neural Networks (CNNs) for image classification, utilizing essential architectural components and optimization strategies.</li>
      <li>Implement Transfer Learning strategies by adapting pre-trained models through feature extraction and fine-tuning to efficiently solve complex computer vision tasks.</li>
    </ul>
  </div>

  <div class="pipeline" id="pipeline">
    <div class="pipeline-stage" data-jump="overview">Raw Image</div>
    <div class="pipeline-arrow">&rarr;</div>
    <div class="pipeline-stage" data-jump="part1">Classical Processing</div>
    <div class="pipeline-arrow">&rarr;</div>
    <div class="pipeline-stage" data-jump="part2">CNN</div>
    <div class="pipeline-arrow">&rarr;</div>
    <div class="pipeline-stage" data-jump="part3">Training</div>
    <div class="pipeline-arrow">&rarr;</div>
    <div class="pipeline-stage" data-jump="part4">Transfer Learning</div>
    <div class="pipeline-arrow">&rarr;</div>
    <div class="pipeline-stage" data-jump="part3">Classification</div>
  </div>

  <nav class="tabs" id="main-tabs">
    <button data-tab="overview" class="active">Overview</button>
    <button data-tab="part1">Part 1: Image Processing</button>
    <button data-tab="part2">Part 2: CNN Architecture</button>
    <button data-tab="part3">Part 3: Training</button>
    <button data-tab="part4">Part 4: Transfer Learning</button>
    <button data-tab="resources">Resources</button>
  </nav>

  <main id="tab-content">
    <!-- TAB_PANELS_PLACEHOLDER: Tasks 4-9 insert <section class="tab-panel" data-panel="..."> blocks here, in order: overview, part1, part2, part3, part4, resources -->
  </main>

</div>

<script>
(function () {
  const tabButtons = document.querySelectorAll('#main-tabs button');
  const panels = document.querySelectorAll('.tab-panel');

  function showTab(name) {
    tabButtons.forEach(b => b.classList.toggle('active', b.dataset.tab === name));
    panels.forEach(p => p.classList.toggle('active', p.dataset.panel === name));
  }

  tabButtons.forEach(b => b.addEventListener('click', () => showTab(b.dataset.tab)));

  document.querySelectorAll('#pipeline [data-jump]').forEach(el => {
    el.addEventListener('click', () => showTab(el.dataset.jump));
  });

  // Subtabs: scoped per tab-panel so each panel's subtabs are independent
  document.querySelectorAll('.tab-panel').forEach(panel => {
    const subButtons = panel.querySelectorAll('nav.subtabs button');
    const subPanels = panel.querySelectorAll('.subtab-panel');
    subButtons.forEach(b => {
      b.addEventListener('click', () => {
        subButtons.forEach(x => x.classList.toggle('active', x === b));
        subPanels.forEach(p => p.classList.toggle('active', p.dataset.subpanel === b.dataset.subtab));
      });
    });
  });

  showTab('overview');
})();
</script>
</body>
</html>
```

- [ ] **Step 2: Verify the file opens without JS errors**

```bash
python3 -m http.server 8123 --directory docs &
SERVER_PID=$!
sleep 1
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8123/index.html
kill $SERVER_PID
```

Expected: `200`. (Full interactive verification happens in Task 10 once content panels exist.)

---

### Task 4: Overview tab content

**Files:**
- Modify: `docs/index.html` — replace `<!-- TAB_PANELS_PLACEHOLDER -->` comment, inserting this as the first panel (keep the comment below it until Task 9 replaces the last piece)

Source: `README.md` Lesson Plan table.

- [ ] **Step 1: Insert the Overview panel**

```html
    <section class="tab-panel active" data-panel="overview">
      <section class="card">
        <h3>What this lesson covers</h3>
        <p>This lesson walks from raw pixels to a trained image classifier. It starts with classical image processing (reading/writing images, color spaces, filtering, edge detection, segmentation, feature extraction), moves into Convolutional Neural Network (CNN) architecture and training with PyTorch on CIFAR-10, and finishes with transfer learning using a pre-trained ResNet. Use the pipeline above or the tabs below to jump to any part.</p>
      </section>
      <section class="card">
        <h3>Lesson Plan</h3>
        <table class="plan-table">
          <thead><tr><th>Duration</th><th>What</th><th>How / Why</th></tr></thead>
          <tbody>
            <tr><td>5 mins</td><td>Start zoom session</td><td>So learners can join early and start on time.</td></tr>
            <tr><td>20 mins</td><td>Activity</td><td>Recap on self-study and prework materials.</td></tr>
            <tr><td>40 mins</td><td>Code-along</td><td>Part 1: Introduction to CV and Image Processing.</td></tr>
            <tr><td>30 mins</td><td>Code-along</td><td>Part 2: CNN architecture.</td></tr>
            <tr><td>10 mins</td><td>Break</td><td>&mdash;</td></tr>
            <tr><td>20 mins</td><td>Code-along</td><td>Part 3: Training CNNs.</td></tr>
            <tr><td>50 mins</td><td>Code-along</td><td>Part 4: Transfer Learning and Pre-trained Models.</td></tr>
            <tr><td>10 mins</td><td>Briefing / Q&amp;A</td><td>References, assignment, quiz and Q&amp;A.</td></tr>
          </tbody>
        </table>
      </section>
    </section>
```

- [ ] **Step 2: Verify HTML is well-formed**

```bash
python3 -c "import html.parser; html.parser.HTMLParser().feed(open('docs/index.html').read()); print('parsed OK')"
```

Expected: `parsed OK` (no exception).

---

### Task 5: Part 1 tab content — Image Processing

**Files:**
- Modify: `docs/index.html` — insert after the Overview panel

Source: notebook cells 0, 1, 4–17, 18–31 (condensed); images from Tasks 1–2.

- [ ] **Step 1: Insert the Part 1 panel**

```html
    <section class="tab-panel" data-panel="part1">
      <nav class="subtabs">
        <button class="active" data-subtab="intro">Intro to CV</button>
        <button data-subtab="io">Reading/Writing Images</button>
        <button data-subtab="color">Color Spaces</button>
        <button data-subtab="classical">Classical Techniques</button>
      </nav>

      <div class="subtab-panel active" data-subpanel="intro">
        <section class="card">
          <h3>What is Computer Vision?</h3>
          <p>Computer vision is a field of AI that enables computers to interpret and understand the visual world, combining image processing (acquiring, analyzing, enhancing images) with machine learning (learning from and deciding based on visual data). Its scope includes image classification, object detection, object tracking, image segmentation, image restoration, scene reconstruction, event detection, image captioning, and pattern recognition.</p>
          <p><strong>Brief history:</strong> early automated object-recognition systems date to the 1950s, with major advances in the 1970s–80s as algorithms and compute improved. Machine learning and feature extraction matured through the 1990s–2000s, and the 2010s introduction of deep learning &mdash; especially CNNs &mdash; revolutionized classification and detection.</p>
          <div class="img-row">
            <figure><img src="img/image_segmentation.jpeg" alt="Image segmentation example"><figcaption>Image segmentation, one of CV's core tasks</figcaption></figure>
          </div>
          <p><strong>Applications:</strong> Healthcare (diagnosing disease from X-rays/MRIs/CT scans), Automotive (autonomous vehicles, ADAS), Retail (consumer behavior, inventory, virtual fitting rooms), Manufacturing (quality control, defect detection), Agriculture (crop monitoring, automated harvesting), Security (surveillance, facial recognition), Entertainment (AR/VR), Smartphones (face unlock, camera AR).</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="io">
        <section class="card">
          <h3>Reading, Writing, and Displaying Images</h3>
          <p>The notebook covers two libraries: <strong>PIL (Pillow)</strong>, a friendly library for opening, manipulating and saving many image formats (JPEG, PNG, BMP, TIFF, ...), and <strong>OpenCV</strong>, a more advanced computer-vision-focused library that also handles basic image I/O. A key gotcha: OpenCV reads images in <strong>BGR</strong> order by default, so images must be converted to RGB before displaying with matplotlib.</p>
          <div class="img-row">
            <figure><img src="img/cute_cat.jpg" alt="Sample image used for PIL/OpenCV I/O examples"><figcaption>Sample image used throughout the I/O examples</figcaption></figure>
          </div>
          <p>An image is fundamentally a grid of <strong>pixels</strong> &mdash; the smallest controllable picture elements, each holding color information, most commonly in the RGB color space.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="color">
        <section class="card">
          <h3>Color Spaces and Conversions</h3>
          <p>Images can be converted between color spaces with either library &mdash; e.g. RGB to grayscale, or RGB to HSV (hue/saturation/value), which is often more convenient than RGB for tasks like color-based thresholding.</p>
          <div class="img-row">
            <figure><img src="img/gray_pil.png" alt="Grayscale conversion using PIL"><figcaption>Grayscale via PIL (<code>.convert('LA')</code>)</figcaption></figure>
            <figure><img src="img/gray_opencv.png" alt="Grayscale conversion using OpenCV"><figcaption>Grayscale via OpenCV (<code>cv2.COLOR_BGR2GRAY</code>)</figcaption></figure>
            <figure><img src="img/hsv.png" alt="HSV color space conversion"><figcaption>HSV conversion via OpenCV</figcaption></figure>
          </div>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="classical">
        <section class="card">
          <h3>Image Filtering &amp; Convolution</h3>
          <p>Image filtering enhances or suppresses features in an image. Convolution applies a small kernel (filter) over the image to produce a filtered output &mdash; the same core operation CNNs later learn to apply automatically.</p>
          <h4>Gaussian Blur</h4>
          <p>A smoothing technique that reduces noise and detail by convolving the image with a Gaussian-weighted kernel, blurring while preserving edges better than a simple mean filter.</p>
          <div class="img-row">
            <figure><img src="img/gaussian_blur.png" alt="Original vs Gaussian blurred image"><figcaption>Original vs. Gaussian-blurred</figcaption></figure>
          </div>
          <h4>Edge Detection: Sobel &amp; Canny</h4>
          <p><strong>Sobel</strong> convolves the image with a pair of 3x3 kernels (x- and y-direction gradients) and combines them into an edge-strength map. <strong>Canny</strong> is a multi-stage detector: Gaussian noise reduction &rarr; gradient calculation &rarr; non-maximum suppression &rarr; double thresholding &rarr; edge tracking by hysteresis. Canny is generally better at localizing edges and more robust to noise.</p>
          <div class="img-row">
            <figure><img src="img/sobel.png" alt="Sobel X and Y edge detection"><figcaption>Sobel X / Y gradients</figcaption></figure>
            <figure><img src="img/canny.png" alt="Canny edge detection"><figcaption>Canny edges</figcaption></figure>
          </div>
          <h4>Image Segmentation: Thresholding</h4>
          <p>Thresholding converts a grayscale image into a binary image: pixels above a threshold become one value (e.g. white), pixels below become another (e.g. black) &mdash; a simple way to separate an object from its background.</p>
          <div class="img-row">
            <figure><img src="img/threshold.png" alt="Binary thresholding"><figcaption>Binary thresholding</figcaption></figure>
          </div>
          <h4>Feature Extraction: HOG, SIFT, SURF</h4>
          <p><strong>HOG</strong> (Histogram of Oriented Gradients) counts gradient-orientation occurrences in localized regions &mdash; strong for human detection. <strong>SIFT</strong> (Scale-Invariant Feature Transform) detects and describes local keypoints that are scale- and rotation-invariant, useful for matching across different views. <strong>SURF</strong> is a faster, patented variant of SIFT using integral images and a Hessian-matrix-based detector.</p>
          <div class="img-row">
            <figure><img src="img/sift.png" alt="SIFT keypoint features"><figcaption>SIFT keypoints</figcaption></figure>
          </div>
        </section>
      </div>
    </section>
```

- [ ] **Step 2: Verify HTML is well-formed**

```bash
python3 -c "import html.parser; html.parser.HTMLParser().feed(open('docs/index.html').read()); print('parsed OK')"
```

Expected: `parsed OK`.

---

### Task 6: Part 2 tab content — CNN Architecture

**Files:**
- Modify: `docs/index.html` — insert after the Part 1 panel

Source: notebook cells 32–35; images `cnn.png`, `convolution.gif`, `pooling.png`.

- [ ] **Step 1: Insert the Part 2 panel**

```html
    <section class="tab-panel" data-panel="part2">
      <nav class="subtabs">
        <button class="active" data-subtab="conv">Conv Layers</button>
        <button data-subtab="pooling">Pooling</button>
        <button data-subtab="fc">FC Layers &amp; Classification</button>
        <button data-subtab="reg">Regularization</button>
      </nav>

      <div class="subtab-panel active" data-subpanel="conv">
        <section class="card">
          <h3>CNN Architecture Overview</h3>
          <p>A typical CNN is built from: an <strong>Input Layer</strong> (raw pixels), <strong>Convolutional Layers</strong> (the core feature-extracting building block), an <strong>Activation/ReLU Layer</strong> (adds non-linearity), <strong>Pooling Layers</strong> (reduce spatial size/parameters), <strong>Fully Connected Layers</strong> (combine features for classification), and an <strong>Output Layer</strong> (class probabilities).</p>
          <div class="img-row">
            <figure><img src="img/cnn.png" alt="CNN architecture diagram"><figcaption>A typical CNN architecture</figcaption></figure>
          </div>
          <h4>Convolutional Layers</h4>
          <p>A convolution slides a small kernel (matrix of weights) across the image, computing a dot product at each position to produce a feature map highlighting patterns like edges or textures. <strong>Padding</strong> (commonly zero-padding) preserves spatial dimensions at the borders; <strong>stride</strong> controls how many pixels the kernel moves each step &mdash; larger strides shrink the output.</p>
          <div class="img-row">
            <figure><img src="img/convolution.gif" alt="Convolution operation animation"><figcaption>The convolution operation (kernel sliding over the input)</figcaption></figure>
          </div>
          <h4>Hierarchical Feature Learning</h4>
          <p>CNNs learn a hierarchy of features automatically: early layers learn <strong>low-level</strong> features (edges, basic textures), middle layers learn <strong>mid-level</strong> features (shapes, textural patterns), and deep layers learn <strong>high-level</strong> features (entire objects/scenes). ReLU is the standard activation used after convolutions for its efficiency and effectiveness.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="pooling">
        <section class="card">
          <h3>Pooling Layers &amp; Spatial Hierarchy</h3>
          <p>Pooling reduces the spatial size of feature maps (and the parameter count), which helps control overfitting, by summarizing regions of the feature map.</p>
          <div class="img-row">
            <figure><img src="img/pooling.png" alt="Pooling operation diagram"><figcaption>Pooling reduces spatial dimensions</figcaption></figure>
          </div>
          <p><strong>Max pooling</strong> (most common) slides a window (e.g. 2x2) over the feature map and keeps the maximum value in each window. <strong>Average pooling</strong> takes the mean instead &mdash; less common, but sometimes useful. Progressive pooling builds a spatial hierarchy: early layers see local patterns, deeper layers see increasingly global ones.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="fc">
        <section class="card">
          <h3>Fully Connected Layers &amp; Classification</h3>
          <p>After the convolution/pooling stack, a CNN typically transitions to fully connected (FC) layers, where every neuron connects to all activations in the previous layer &mdash; just like a standard neural network. The FC stack takes the high-level features learned earlier and uses them to classify the input.</p>
          <p>The final FC layer has one neuron per class, and a <strong>softmax</strong> activation converts the raw outputs into a probability distribution over the classes.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="reg">
        <section class="card">
          <h3>Training Techniques: Augmentation, BatchNorm, Dropout</h3>
          <p><strong>Data Augmentation</strong> increases training-set diversity via random transformations (rotation, scaling, translation, flipping, cropping) &mdash; helping the model generalize.</p>
          <p><strong>Batch Normalization</strong> normalizes each layer's inputs to zero mean / unit variance (with learned scale &amp; shift), combating internal covariate shift and enabling higher learning rates and faster convergence.</p>
          <p><strong>Dropout</strong> randomly zeroes a fraction of a layer's outputs during training, a regularization technique that reduces overfitting by preventing co-adaptation of neurons.</p>
        </section>
      </div>
    </section>
```

- [ ] **Step 2: Verify HTML is well-formed**

```bash
python3 -c "import html.parser; html.parser.HTMLParser().feed(open('docs/index.html').read()); print('parsed OK')"
```

Expected: `parsed OK`.

---

### Task 7: Part 3 tab content — Training

**Files:**
- Modify: `docs/index.html` — insert after the Part 2 panel

Source: notebook cells 36–53; image `cifar_predictions.png`.

- [ ] **Step 1: Insert the Part 3 panel**

```html
    <section class="tab-panel" data-panel="part3">
      <nav class="subtabs">
        <button class="active" data-subtab="pipeline">Model &amp; Data Pipeline</button>
        <button data-subtab="loop">Training Loop</button>
        <button data-subtab="eval">Evaluation &amp; Predictions</button>
      </nav>

      <div class="subtab-panel active" data-subpanel="pipeline">
        <section class="card">
          <h3>Training a CNN with PyTorch on CIFAR-10</h3>
          <p>The notebook builds a <code>SimpleCNN</code> (a PyTorch <code>nn.Module</code>) and trains it on the <a href="https://en.wikipedia.org/wiki/CIFAR-10" target="_blank" rel="noopener">CIFAR-10 dataset</a> (60,000 32x32 color images across 10 classes: plane, car, bird, cat, deer, dog, frog, horse, ship, truck).</p>
          <p>The data pipeline: define <strong>transforms</strong> for data augmentation &rarr; load train/test datasets &rarr; wrap them in PyTorch <code>DataLoader</code>s for batching and shuffling &rarr; define a <strong>loss function</strong> and <strong>optimizer</strong>.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="loop">
        <section class="card">
          <h3>The Training Loop</h3>
          <p>For each epoch, the training function iterates over batches from the training <code>DataLoader</code>: forward pass through <code>SimpleCNN</code> &rarr; compute loss against the true labels &rarr; backpropagate gradients &rarr; step the optimizer to update weights. Data augmentation and batch normalization (introduced in Part 2) are applied during this process to improve generalization and training stability.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="eval">
        <section class="card">
          <h3>Evaluating the Model</h3>
          <p>After training, the model is evaluated on the held-out test set to measure accuracy on unseen data. The notebook then samples random test images and displays each with its predicted class alongside the actual class, making it easy to spot correct vs. incorrect predictions at a glance.</p>
          <div class="img-row">
            <figure><img src="img/cifar_predictions.png" alt="Grid of CIFAR-10 predictions vs actual labels"><figcaption>Predicted vs. actual labels on sample CIFAR-10 test images</figcaption></figure>
          </div>
        </section>
      </div>
    </section>
```

- [ ] **Step 2: Verify HTML is well-formed**

```bash
python3 -c "import html.parser; html.parser.HTMLParser().feed(open('docs/index.html').read()); print('parsed OK')"
```

Expected: `parsed OK`.

---

### Task 8: Part 4 tab content — Transfer Learning

**Files:**
- Modify: `docs/index.html` — insert after the Part 3 panel

Source: notebook cells 54–61.

- [ ] **Step 1: Insert the Part 4 panel**

```html
    <section class="tab-panel" data-panel="part4">
      <nav class="subtabs">
        <button class="active" data-subtab="concept">Concept</button>
        <button data-subtab="resnet">ResNet</button>
        <button data-subtab="strategy">Feature Extraction vs. Fine-Tuning</button>
      </nav>

      <div class="subtab-panel active" data-subpanel="concept">
        <section class="card">
          <h3>Transfer Learning</h3>
          <p>Transfer learning reuses a model developed for one task as the starting point for a second task &mdash; instead of training from scratch. Models pre-trained on huge benchmark datasets like <a href="https://en.wikipedia.org/wiki/ImageNet" target="_blank" rel="noopener">ImageNet</a> (1M+ images, 1000 categories) offer three main benefits: <strong>time-saving</strong> (skip most of the training time), <strong>improved performance</strong> (especially with limited new-task data), and reusable <strong>feature extraction</strong> (learned features transfer even to different classes).</p>
          <p>Popular pre-trained CNN architectures include LeNet-5, AlexNet, VGGNet, GoogLeNet (Inception), and ResNet.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="resnet">
        <section class="card">
          <h3>ResNet (Residual Network)</h3>
          <p>ResNet was designed to make training very deep networks practical by solving the vanishing-gradient problem. Its core idea is the <strong>residual block</strong>: a <strong>skip connection</strong> lets the input bypass one or more layers and be added back to their output, so gradients can flow directly through the network during backpropagation. ResNet also applies batch normalization after every convolution.</p>
          <p>Common depths are ResNet-18, 34, 50, 101, and 152 (the number = layer count). ResNet-18/34 use basic residual blocks (two 3x3 convolutions); ResNet-50/101/152 use bottleneck blocks (1x1 &rarr; 3x3 &rarr; 1x1 convolutions) to reduce computational cost at greater depth.</p>
        </section>
      </div>

      <div class="subtab-panel" data-subpanel="strategy">
        <section class="card">
          <h3>Feature Extraction vs. Fine-Tuning</h3>
          <p><strong>Feature Extraction:</strong> use the pre-trained ResNet as a fixed feature extractor &mdash; replace only the final fully connected layer (<code>model.fc</code>) with a new one for your task, freeze everything else, and train just that new layer.</p>
          <p><strong>Fine-Tuning:</strong> unfreeze some or all of the pre-trained layers and continue training on the new dataset at a very low learning rate, letting the model adjust its learned representations to the new task.</p>
          <p>The notebook applies this to CIFAR-10 using a pre-trained ResNet, replacing the final layer for the 10 CIFAR-10 classes. Full training runs are computationally heavy on CPU and are recommended to run on GPU (e.g. Google Colab).</p>
        </section>
      </div>
    </section>
```

- [ ] **Step 2: Verify HTML is well-formed**

```bash
python3 -c "import html.parser; html.parser.HTMLParser().feed(open('docs/index.html').read()); print('parsed OK')"
```

Expected: `parsed OK`.

---

### Task 9: Resources tab content

**Files:**
- Modify: `docs/index.html` — insert after the Part 4 panel, replacing the final `<!-- TAB_PANELS_PLACEHOLDER -->` comment

Source: `README.md` dependencies list, `reference.md`.

- [ ] **Step 1: Insert the Resources panel**

```html
    <section class="tab-panel" data-panel="resources">
      <section class="card">
        <h3>Course Materials</h3>
        <ul class="resource-links">
          <li><a href="../studies.md">Self-Study Preparation Guide</a> &mdash; pre-class reading and reflection tasks (~45-60 min)</li>
          <li><a href="../lesson.md">Lesson Brief</a> &mdash; environment setup and in-class notebook instructions</li>
          <li><a href="../assignment.md">Assignment</a> &mdash; build a Fashion MNIST image classifier</li>
          <li><a href="../reference.md">Reference</a> &mdash; external documentation links</li>
          <li><a href="../notebooks/computer_vision_lesson.ipynb">Lesson Notebook</a> &mdash; the full code-along notebook this summary is based on</li>
        </ul>
      </section>
      <section class="card">
        <h3>External References</h3>
        <ul class="resource-links">
          <li><a href="https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html" target="_blank" rel="noopener">PyTorch: Transfer Learning Tutorial</a></li>
          <li><a href="https://learn.microsoft.com/en-us/training/modules/intro-computer-vision-pytorch/" target="_blank" rel="noopener">Microsoft Learn: Introduction to Computer Vision with PyTorch</a></li>
        </ul>
      </section>
    </section>
```

- [ ] **Step 2: Verify HTML is well-formed and all 6 panels are present**

```bash
python3 -c "
import html.parser, re
content = open('docs/index.html').read()
html.parser.HTMLParser().feed(content)
panels = re.findall(r'data-panel=\"(\w+)\"', content)
print('parsed OK, panels:', panels)
assert panels == ['overview', 'part1', 'part2', 'part3', 'part4', 'resources'], panels
"
```

Expected: `parsed OK, panels: ['overview', 'part1', 'part2', 'part3', 'part4', 'resources']`.

---

### Task 10: Full manual verification

**Files:** none (verification only)

- [ ] **Step 1: Serve `docs/` locally**

```bash
python3 -m http.server 8123 --directory docs &
echo "Open http://localhost:8123/index.html in a browser"
```

- [ ] **Step 2: Manually check in browser**

Checklist:
- All 6 top-level tabs switch correctly and only one panel is visible at a time
- Every subtab within each Part switches correctly and is scoped to its own Part (clicking a Part 1 subtab doesn't affect Part 2's subtabs)
- All 15 images in `docs/img/` render (no broken image icons) — 6 copied assets + 9 extracted notebook outputs
- Clicking each hero pipeline stage jumps to the correct tab
- Resource links in the Resources tab resolve (relative `../*.md` links open/download correctly; the notebook link opens/downloads)
- Page does not visually break (no horizontal scroll, no overlapping text) at both a wide desktop width and a narrower ~900px width

- [ ] **Step 3: Stop the local server**

```bash
kill %1 2>/dev/null || true
```

- [ ] **Step 4: Confirm no source files outside `docs/` were modified**

```bash
git status --short
```

Expected: only new/untracked paths under `docs/` (`docs/index.html`, `docs/img/*`, and the spec/plan docs under `docs/superpowers/`); no modifications to `assets/`, `notebooks/`, `README.md`, `lesson.md`, `studies.md`, `assignment.md`, or `reference.md`.

---

## Notes

- No `git commit` steps are included — per project convention, commits are only made when the user explicitly requests them. Once verification (Task 10) passes, ask the user whether to commit.
- No content in `assets/`, `notebooks/`, or any lesson markdown file is modified at any point in this plan.
