# imagedistortion

Automatic lens correction: metadata + formula when possible, in-house ML when not; optional Step C (external AI) and self-evaluation. See **AUTOMATIC_LENS_CORRECTION_PLAN.md** for the full architecture.

## Setup

```bash
# From project root: create venv, activate, install deps
cd /path/to/imagedistortion
python3 -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Project layout

- **AUTOMATIC_LENS_CORRECTION_PLAN.md** — Full plan (Phase 0 data processing, Dataset A/B, lens formulas, correction paths, self-eval, Step C).
- **Dataset-Train/** — Training metadata (`training_metadata.json`) and, if present, the original/generated image files (same directory or configured source).
- **data/** — After Step 0: `raw_dataset/` (mirror of source, unchanged), `dataset_a/` (originals/ + corrected/), `dataset_b/` (one folder per lens + others), `manifest.json`, `manifest.csv`.
- **src/data/prepare_step0.py** — Step 0 data processing (Dataset A, B, manifest).
- **api/main.py** — FastAPI backend for Step 0 (and future steps).
- **frontend/** — React UI; proxies `/api` to the backend.

## Step 0 (Data processing)

From the frontend (Step 0 page) you can run:

1. **Build raw dataset** — Builds `data/raw_dataset/` as an exact mirror of the source directory (e.g. `Dataset-Train/`). All files are copied (or symlinked) unchanged — no renaming, no reorganization. Use this to keep a raw copy of the ~20k files for any future use.

2. **Run Dataset A** — Builds `data/dataset_a/` with two subfolders:
   - `originals/` — all non-corrected (original) images, named `<pair_id>_original.jpg`
   - `corrected/` — all rectified images, named `<pair_id>_corrected.jpg`
   - Displays how many pairs were copied and paths.

3. **Run Dataset B** — Builds `data/dataset_b/` using lens info from metadata (EXIF-derived lens names):
   - One subfolder per lens (e.g. `SONY_FE_28-70mm_F3_5-5_6_OSS`)
   - An `others/` folder for pairs with no lens info
   - Each lens folder contains that lens’s original + corrected pairs.

4. **Generate manifest** — Writes `data/manifest.json` and `data/manifest.csv` with:
   - Every lens type and how many images (pairs) per lens
   - Others count and total pairs
   - Basic Step 0 summary.

Image files are expected under **Dataset-Train/** with filenames matching `training_metadata.json` (e.g. `..._original.jpg`, `..._generated.jpg`). If your images live elsewhere, use the API with `source_dir` in the payload (or set it in a future config).

**How long it takes:** Raw dataset and Dataset A/B depend on file count and disk speed. For ~23k pairs (46k+ files), **copying** can take a few minutes; **symlinks** (`use_symlinks: true`) are much faster (seconds). The API response includes `elapsed_seconds`. Once a dataset is generated, running the same step again **skips** work and returns immediately ("already exists; skipping"). Use the **Force regenerate** checkbox on Step 0 (or `force: true` in the API) to re-run.

## Running the app

**Backend (required for Step 0):**

```bash
# From project root
source venv/bin/activate   # if not already activated
uvicorn api.main:app --reload --host 0.0.0.0 --port 8000
# Or without activating: ./venv/bin/uvicorn api.main:app --reload --host 0.0.0.0 --port 8000
```

**Frontend:**

```bash
cd frontend && npm install && npm run dev
```

Open http://localhost:5173. Step 0 buttons call the backend; other steps use mock responses.

## Usage

_See the plan for implementation order and scripts (prepare datasets, train, correct test set, zip & submit)._
