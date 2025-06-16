# AGENTS.md – **EdgeBatch**

> Comprehensive guidance for AI agents (e.g. OpenAI Codex) working in the **EdgeBatch** Python project – a batch image‑processing tool that applies multiple state‑of‑the‑art OpenCV edge‑detectors to every image in a user‑selected folder, inverts the results (white background, dark lines) and stores each detector’s output in a dedicated sub‑directory.

---

## Supported Edge‑Detection Methods

*(Every method is wrapped in **`detectors.py`**; names below appear exactly in code & CLI flags so AI agents can reference them.)*

| CLI Flag / Class | Algorithm                              | OpenCV API                                   | Purpose                                                                        |
| ---------------- | -------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------ |
| `canny`          | **Canny**                              | `cv.Canny` / `cv.cuda.CannyEdgeDetector`     | Fast, classic baseline with hysteresis & non‑max suppression (CPU & GPU).      |
| `sed`            | **Structured Edge Detection**          | `cv.ximgproc.createStructuredEdgeDetection`  | Random‑forest model giving clean, continuous edges – ideal for natural images. |
| `hed`            | **Holistically‑Nested Edge Detection** | `cv.dnn.readNet` (Caffe)                     | CNN delivering pixel‑accurate object boundaries across scales.                 |
| `pidi`           | **PiDiNet**                            | `cv.dnn.readNet` (ONNX)                      | Lightweight DL model for real‑time crisp edges (optional).                     |
| `dexined`        | **DexiNed**                            | `cv.dnn.readNet` (ONNX)                      | Deep model excelling on fine details (optional).                               |
| `ed`             | **Edge Drawing**                       | `cv.ximgproc.createEdgeDrawing`              | Builds linked edge chains, great for subsequent geometry analysis.             |
| `edlines`        | **EDLines**                            | `cv.ximgproc.createEdgeDrawing` (mode LINES) | Specialized variant extracting line segments from Edge Drawing chains.         |
| `lsd`            | **Line Segment Detector (LSD)**        | `cv.createLineSegmentDetector`               | Sub‑pixel accurate straight‑line extraction; works well on CAD‑like imagery.   |
| `fld`            | **FastLineDetector (FLD)**             | `cv.ximgproc.createFastLineDetector`         | Real‑time optimized LSD alternative; can reuse external edge map.              |
| `sobel`          | **Sobel**                              | `cv.Sobel`                                   | Simple gradient snapshot – handy for shader pre‑passes.                        |
| `scharr`         | **Scharr**                             | `cv.Scharr`                                  | Improved rotation‑invariant variant of Sobel.                                  |
| `laplacian`      | **Laplacian**                          | `cv.Laplacian`                               | 2‑nd derivative edge emphasis; no non‑max suppression.                         |

> 🗂 Outputs are saved under `<root>/<method_name>/image_name.png`, already inverted (255‑background) and in the original resolution.

---

## Project Structure

```text
edgebatch/
├── src/
│   ├── edgebatch/
│   │   ├── __init__.py
│   │   ├── detectors.py        # All wrappers listed above
│   │   ├── pipeline.py         # Orchestrates batch run, inversion, thread‑pool
│   │   ├── io.py               # Folder picker (Tkinter) & image I/O
│   │   └── cli.py              # Typer CLI (`edgebatch --help`)
│   ├── gui/
│   │   └── app.py              # Optional desktop GUI (async‑Tkinter + ttk)
│   └── data/
│       ├── models/             # DL weight files validated via SHA‑256
│       │   ├── model.yml.gz               # SED model
│       │   ├── hed_pretrain.caffemodel    # HED weights
│       │   ├── hed_deploy.prototxt        # HED prototxt
│       │   ├── pidi.onnx                  # PiDiNet weights (opt.)
│       │   └── dexined.onnx               # DexiNed weights (opt.)
│       └── samples/
│           └── test.jpg
├── tests/
│   ├── test_detectors.py
│   ├── test_pipeline.py
│   └── fixtures/
│       └── sample_set/
├── pyproject.toml              # Build & tool config
├── README.md                   # User‑level instructions
└── AGENTS.md                   # ← this file
```

* **Entry point for AI tooling:** `edgebatch.pipeline.run_all(path: PathLike, methods: list[str] | None = None, workers: int = 4)`.
* GPU is auto‑detected; CUDA versions of Canny & HED are selected when available.
* GUI layer is isolated; safe to ignore for headless automation.

---

## Coding Conventions

| Topic            | Convention                                                                                                                                                      |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Formatter**    | `black` (line‑length = 88) – executed via pre‑commit                                                                                                            |
| **Imports**      | `isort` (profile `black`)                                                                                                                                       |
| **Linting**      | `flake8`, `flake8‑bugbear`, `flake8‑annotations`                                                                                                                |
| **Typing**       | `mypy --strict` (GUI modules exempt via `# type: ignore[import]`)                                                                                               |
| **Docstrings**   | Google style, checked with `pydocstyle`                                                                                                                         |
| **Security**     | `bandit -r src/` in CI                                                                                                                                          |
| **Dependencies** | Pinned in `pyproject.toml`; main runtime deps: `opencv‑python‑headless>=4.11`, `opencv‑contrib‑python‑headless>=4.11`, `numpy>=1.26`, `typer`, `pillow`, `tqdm` |

---

## Testing Requirements

* **Framework:** `pytest` ≥ 8 + `pytest‑mocker`, `pytest‑cov`.
* **Coverage Goal:** ≥ 90 % lines in `src/edgebatch/`.
* **Fixtures:** Lightweight (<100 kB) JPEGs in `tests/fixtures/` for deterministic pixel asserts.
* **Unit Tests:**

  * Each detector wrapper returns `np.ndarray` uint8 0/255 mask with shape `(h, w)`.
  * Verify inversion (background = 255).
* **Integration Tests:** Run `run_all()` on fixture set; expect 13 output folders (one per method) & consistent filenames.
* **Regression Build:** Nightly GH Actions workflow reruns full pipeline; failures gate merges.

Run locally:

```bash
pytest -q --cov=edgebatch --cov-report=term-missing
```

---

## PR Guidelines

1. **Branching:** `feature/<short‑desc>` from `main`.
2. **Commits:** Conventional Commits (`feat:`, `fix:`, `refactor:`…).
3. **Description Template:** *Why*, *What*, *How*.
4. **Checklist (CI‑enforced):** linters, tests, typing, CHANGELOG.
5. **Review:** ≥ 1 maintainer approval; module owners auto‑requested.
6. **Merge:** Squash‑merge with tidy message.

---

## Programmatic Checks

| Tool                    | Trigger              | Mandatory Outcome                                             |
| ----------------------- | -------------------- | ------------------------------------------------------------- |
| **pre‑commit**          | `git commit`         | black, isort, flake8, mypy, pydocstyle                        |
| **validate\_models.py** | `pip install .` & CI | SHA‑256 of every file in `data/models/` matches `models.json` |
| **bandit**              | CI (Job “security”)  | No medium/high issues                                         |
| **GitHub Actions**      | Push / PR            | Build wheels, run tests, upload coverage                      |
| **Dependabot**          | Weekly               | Auto PRs for updated deps                                     |

---

## Agents‑Specific Notes

* **Detector Registry:** `DETECTORS: dict[str, Type[BaseDetector]]` – keys equal CLI flags above.
* **Public API:** `detect(img)` only; internal helpers are underscored.
* **Extensibility:**

  1. Add new `*Detector` class.
  2. Register in `DETECTORS`.
  3. Pin weights under `data/models/` & update checksum list.
  4. Write unit tests & fixture sample.
* **Threading:** CPU‑bound detectors run in process pool; GPU ones in main process (CUDA context).

---

### Quickstart

```bash
# install dev env
pip install -e '.[dev]'
# batch process all detectors
edgebatch /path/to/images --workers 8
# only high‑quality methods
edgebatch /path --methods sed hed pidi dexined
```

---

### Reference Build Flags (native OpenCV)

```bash
cmake -DOPENCV_EXTRA_MODULES_PATH=$OPENCV_CONTRIB/modules \
      -D BUILD_opencv_ximgproc=ON  \
      -D BUILD_opencv_dnn=ON       \
      -D BUILD_opencv_imgcodecs=ON .. && \
make -j$(nproc)
```

---

## Maintainers

| Area           | GitHub Handle |
| -------------- | ------------- |
| Core / Release | @edge‑maint   |
| Docs & QA      | @docs‑writer  |

---

> With this **AGENTS.md**, AI tools have an explicit contract for navigating, extending and validating the EdgeBatch project – **listing every supported edge‑detection method by name** so nothing gets overlooked.
