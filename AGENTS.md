# AGENTS.md â€“ **EdgeBatch**

> Comprehensive guidance for AI agents (e.g.â€¯OpenAIÂ Codex) working in the **EdgeBatch** Python project â€“ a batch imageâ€‘processing tool that applies multiple stateâ€‘ofâ€‘theâ€‘art OpenCV edgeâ€‘detectors to every image in a userâ€‘selected folder, inverts the results (white background, dark lines) and stores each detectorâ€™s output in a dedicated subâ€‘directory.

---

## Supported Edgeâ€‘Detection Methods  
*(Every method is wrapped in `detectors.py`; names below appear exactly in code & CLI flags so AI agents can reference them.)*

| CLI Flag / Class | Algorithm | OpenCV API | Purpose |
|------------------|-----------|-----------|---------|
| `canny` | **Canny** | `cv.Canny` / `cv.cuda.CannyEdgeDetector` | Fast, classic baseline with hysteresis & nonâ€‘max suppression (CPU & GPU). |
| `sed` | **Structuredâ€¯Edgeâ€¯Detection** | `cv.ximgproc.createStructuredEdgeDetection` | Randomâ€‘forest model giving clean, continuous edges â€“ ideal for natural images. |
| `hed` | **Holisticallyâ€‘Nestedâ€¯Edgeâ€¯Detection** | `cv.dnn.readNet` (Caffe) | CNN delivering pixelâ€‘accurate object boundaries across scales. |
| `pidi` | **PiDiNet** | `cv.dnn.readNet` (ONNX) | Lightweight DL model for realâ€‘time crisp edges (optional). |
| `dexined` | **DexiNed** | `cv.dnn.readNet` (ONNX) | Deep model excelling on fine details (optional). |
| `ed` | **Edgeâ€¯Drawing** | `cv.ximgproc.createEdgeDrawing` | Builds linked edge chains, great for subsequent geometry analysis. |
| `edlines` | **EDLines** | `cv.ximgproc.createEdgeDrawing` (modeÂ LINES) | Specialized variant extracting line segments from Edgeâ€¯Drawing chains. |
| `lsd` | **Lineâ€¯Segmentâ€¯Detector (LSD)** | `cv.createLineSegmentDetector` | Subâ€‘pixel accurate straightâ€‘line extraction; works well on CADâ€‘like imagery. |
| `fld` | **FastLineDetector (FLD)** | `cv.ximgproc.createFastLineDetector` | Realâ€‘time optimized LSD alternative; can reuse external edge map. |
| `sobel` | **Sobel** | `cv.Sobel` | Simple gradient snapshot â€“ handy for shader preâ€‘passes. |
| `scharr` | **Scharr** | `cv.Scharr` | Improved rotationâ€‘invariant variant of Sobel. |
| `laplacian` | **Laplacian** | `cv.Laplacian` | 2â€‘nd derivative edge emphasis; no nonâ€‘max suppression.

> ðŸ—‚Â Outputs are saved under `<root>/<method_name>/image_name.png`, already inverted (255â€‘background) and in the original resolution.

---

## Project Structure

```text
edgebatch/
â”œâ”€â”€ src/
â”‚   â”œâ”€â”€ edgebatch/
â”‚   â”‚   â”œâ”€â”€ __init__.py
â”‚   â”‚   â”œâ”€â”€ detectors.py        # All wrappers listed above
â”‚   â”‚   â”œâ”€â”€ pipeline.py         # Orchestrates batch run, inversion, threadâ€‘pool
â”‚   â”‚   â”œâ”€â”€ io.py               # Folder picker (Tkinter) & image I/O
â”‚   â”‚   â””â”€â”€ cli.py              # Typer CLI (`edgebatch --help`)
â”‚   â”œâ”€â”€ gui/
â”‚   â”‚   â””â”€â”€ app.py              # Optional desktop GUI (asyncâ€‘Tkinter + ttk)
â”‚   â””â”€â”€ data/
â”‚       â”œâ”€â”€ models/             # DL weight files validated via SHAâ€‘256
â”‚       â”‚   â”œâ”€â”€ model.yml.gz               # SED model
â”‚       â”‚   â”œâ”€â”€ hed_pretrain.caffemodel    # HED weights
â”‚       â”‚   â”œâ”€â”€ hed_deploy.prototxt        # HED prototxt
â”‚       â”‚   â”œâ”€â”€ pidi.onnx                  # PiDiNet weights (opt.)
â”‚       â”‚   â””â”€â”€ dexined.onnx               # DexiNed weights (opt.)
â”‚       â””â”€â”€ samples/
â”‚           â””â”€â”€ test.jpg
â”œâ”€â”€ tests/
â”‚   â”œâ”€â”€ test_detectors.py
â”‚   â”œâ”€â”€ test_pipeline.py
â”‚   â””â”€â”€ fixtures/
â”‚       â””â”€â”€ sample_set/
â”œâ”€â”€ pyproject.toml              # Build & tool config
â”œâ”€â”€ README.md                   # Userâ€‘level instructions
â””â”€â”€ AGENTS.md                   # â† this file
```

- **Entry point for AI tooling:** `edgebatch.pipeline.run_all(path: PathLike, methods: list[str] | None = None, workers: int = 4)`.
- GPU is autoâ€‘detected; CUDA versions of Canny & HED are selected when available.
- GUI layer is isolated; safe to ignore for headless automation.

---

## Coding Conventions

| Topic | Convention |
|-------|------------|
| **Formatter** | `black` (lineâ€‘lengthÂ =Â 88) â€“ executed via preâ€‘commit |
| **Imports** | `isort` (profileÂ `black`) |
| **Linting** | `flake8`, `flake8â€‘bugbear`, `flake8â€‘annotations` |
| **Typing** | `mypy --strict` (GUI modules exempt via `# type: ignore[import]`) |
| **Docstrings** | Google style, checked with `pydocstyle` |
| **Security** | `bandit -r src/` in CI |
| **Dependencies** | Pinned in `pyproject.toml`; main runtime deps: `opencvâ€‘pythonâ€‘headless>=4.11`, `opencvâ€‘contribâ€‘pythonâ€‘headless>=4.11`, `numpy>=1.26`, `typer`, `pillow`, `tqdm` |

---

## Testing Requirements

- **Framework:** `pytest`Â â‰¥â€¯8Â + `pytestâ€‘mocker`, `pytestâ€‘cov`.
- **Coverage Goal:** â‰¥â€¯90â€¯% lines in `src/edgebatch/`.
- **Fixtures:** Lightweight (<100â€¯kB) JPEGs in `tests/fixtures/` for deterministic pixel asserts.
- **Unit Tests:**
  - Each detector wrapper returns `np.ndarray` uint8 0/255 mask with shapeÂ `(h, w)`.
  - Verify inversion (background = 255).
- **Integration Tests:** Run `run_all()` on fixture set; expect 13 output folders (one per method) & consistent filenames.
- **Regression Build:** Nightly GHÂ Actions workflow reruns full pipeline; failures gate merges.

Run locally:

```bash
pytest -q --cov=edgebatch --cov-report=term-missing
```

---

## PR Guidelines

1. **Branching:** `feature/<shortâ€‘desc>` from `main`.
2. **Commits:** Conventional Commits (`feat:`,Â `fix:`,Â `refactor:`â€¦).
3. **Description Template:** *Why*, *What*, *How*.
4. **Checklist (CIâ€‘enforced):** linters, tests, typing, CHANGELOG.
5. **Review:** â‰¥â€¯1 maintainer approval; module owners autoâ€‘requested.
6. **Merge:** Squashâ€‘merge with tidy message.

---

## Programmatic Checks

| Tool | Trigger | Mandatory Outcome |
|------|---------|-------------------|
| **preâ€‘commit** | `git commit` | black, isort, flake8, mypy, pydocstyle |
| **validate_models.py** | `pip install .` & CI | SHAâ€‘256 of every file in `data/models/` matches `models.json` |
| **bandit** | CIÂ (JobÂ â€œsecurityâ€) | NoÂ medium/high issues |
| **GitHubÂ Actions** | PushÂ /Â PR | Build wheels, run tests, upload coverage |
| **Dependabot** | Weekly | Auto PRs for updated deps |

---

## Agentsâ€‘Specific Notes

- **Detector Registry:** `DETECTORS: dict[str, Type[BaseDetector]]` â€“ keys equal CLI flags above.
- **Public API:** `detect(img)` only; internal helpers are underscored.
- **Extensibility:**
  1. Add new `*Detector` class.
  2. Register in `DETECTORS`.
  3. Pin weights under `data/models/` & update checksum list.
  4. Write unit tests & fixture sample.
- **Threading:** CPUâ€‘bound detectors run in process pool; GPU ones in main process (CUDA context).

---

### Quickstart

```bash
# install dev env
pip install -e '.[dev]'
# batch process all detectors
edgebatch /path/to/images --workers 8
# only highâ€‘quality methods
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

| Area | GitHubÂ Handle |
|------|--------------|
| Core / Release | @edgeâ€‘maint |
| Docs & QA | @docsâ€‘writer |

---

> With this **AGENTS.md**, AI tools have an explicit contract for navigating, extending and validating the EdgeBatch project â€“ **listing every supported edgeâ€‘detection method by name** so nothing gets overlooked.
