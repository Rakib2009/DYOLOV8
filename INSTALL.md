# Installation & Usage Guide

Complete step-by-step guide to install, run, test, and train the Camera-Based
Grocery Product Detection system.

---

## What's in this project

```
rakib_job_project/
├── README.md              ← full project docs
├── INSTALL.md             ← this file
├── TASK_BRIEF.md          ← original hiring task
├── requirements.txt
├── data/data.yaml.example ← template for your dataset
├── samples/               ← put test images here
├── models/                ← trained best.pt lands here
└── src/
    ├── detector.py        ← shared YOLO wrapper
    ├── api.py             ← FastAPI service
    ├── infer.py           ← CLI inference
    ├── train.py           ← fine-tuning
    └── webcam.py          ← real-time bonus
```

---

## Step 1 — Prerequisites

You need **Python 3.10 or newer** (tested on 3.14, works fine). Check:

```bash
python3 --version
```

That's it. No Docker, no CUDA setup, no GPU required.

---

## Step 2 — Install

Run these from the project folder:

```bash
# 1. Create a virtual environment (skip if .venv already exists)
python3 -m venv .venv

# 2. Activate it
source .venv/bin/activate
# On Windows:  .venv\Scripts\activate

# 3. Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

This installs `ultralytics`, `fastapi`, `uvicorn`, `pillow`, `opencv-python`,
`numpy`. Takes ~1–2 minutes (~500 MB; PyTorch is the largest part).

**First-run note:** Ultralytics auto-downloads `yolov8n.pt` (~6 MB) the first
time you run inference.

---

## Step 3 — Run the API

```bash
source .venv/bin/activate
uvicorn src.api:app --host 0.0.0.0 --port 8000 --reload
```

You should see:

```
Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

The service is live at **http://localhost:8000**.

### Endpoints

| Method | Path       | Purpose                                            |
| ------ | ---------- | -------------------------------------------------- |
| GET    | `/`        | Service info, current model                        |
| GET    | `/health`  | Health check (returns `{"status":"ok"}`)           |
| GET    | `/docs`    | **Swagger UI — use this in your demo video**       |
| POST   | `/detect`  | Upload an image, get JSON detections               |

---

## Step 4 — Test the API

### Option A — Browser (easiest, best for video demo)

1. Open http://localhost:8000/docs
2. Expand **POST /detect** → click **Try it out**
3. Click **Choose File**, pick any image (e.g. `samples/test.jpg`)
4. Click **Execute**
5. Scroll down — you'll see the JSON response

### Option B — curl

```bash
curl -X POST http://localhost:8000/detect -F "file=@samples/test.jpg"
```

### Option C — Postman

- Method: **POST**
- URL: `http://localhost:8000/detect`
- Body → **form-data** → key: `file` (type **File**) → choose an image
- Send

### Expected response

```json
{
  "detections": [
    {"class": "bus",    "confidence": 0.8734, "bbox": [22.87, 231.28, 805.0, 756.84]},
    {"class": "person", "confidence": 0.8657, "bbox": [48.55, 398.55, 245.35, 902.7]}
  ],
  "model": "yolov8n.pt",
  "custom_trained": false
}
```

The required `class` + `confidence` shape matches the task brief; `bbox` and
`model` are added for usefulness and traceability.

---

## Step 5 — CLI inference (no server)

```bash
source .venv/bin/activate
python -m src.infer samples/test.jpg
```

Prints the same JSON to your terminal.

Flags:

```bash
python -m src.infer samples/test.jpg --weights models/best.pt --conf 0.4
```

---

## Step 6 — Real-time webcam (bonus)

```bash
source .venv/bin/activate
python -m src.webcam              # default camera
python -m src.webcam --source 1   # other camera
python -m src.webcam --source path/to/video.mp4
```

A window opens with live bounding boxes and FPS. Press **`q`** to quit.

*On macOS:* Terminal needs Camera permission → System Settings → Privacy &
Security → Camera.

---

## Step 7 — Train on a grocery dataset (recommended for submission)

The task brief says *"customization required"* — so for a real submission,
fine-tune on actual grocery data instead of using only COCO pretrained.

### Get a dataset

1. Go to https://universe.roboflow.com
2. Search **"grocery detection"** or **"supermarket products"**
3. Pick one (anything with `rice`, `oil`, `soap`, `packet` style classes)
4. **Export → YOLOv8 → download zip**
5. Unzip into `data/` so the structure looks like:

   ```
   data/
   ├── data.yaml
   ├── train/images/ + train/labels/
   └── valid/images/ + valid/labels/
   ```

### Train

```bash
source .venv/bin/activate
python -m src.train --data data/data.yaml --epochs 50 --imgsz 640
```

Device flags:

- Apple Silicon: add `--device mps`
- NVIDIA CUDA:    add `--device 0`
- CPU (default):  no flag needed (slower but works)

When training finishes, `best.pt` is automatically copied to
`models/best.pt`. **Restart the API** and it will pick it up automatically:

```
GET /  →  "model": "best.pt", "custom_trained": true
```

---

## Verification checklist

After install, you can confirm everything works with this quick smoke test:

```bash
source .venv/bin/activate

# 1. CLI inference
python -m src.infer samples/test.jpg

# 2. Start API in one terminal:
uvicorn src.api:app --port 8000

# 3. In another terminal, test it:
curl http://localhost:8000/health
curl -X POST http://localhost:8000/detect -F "file=@samples/test.jpg"
```

Expected on my last full re-verification:

| Check                                  | Result            |
| -------------------------------------- | ----------------- |
| All source files present               | OK (12 files)     |
| Python imports load cleanly            | OK                |
| `uvicorn` starts the API               | OK                |
| `GET /health`                          | `{"status":"ok"}` |
| `GET /`                                | Returns model info|
| `POST /detect` with `samples/test.jpg` | 6 detections      |
| `POST /detect` with missing file       | HTTP 422          |
| `POST /detect` with non-image          | HTTP 400          |

---

## Common issues

| Symptom                              | Fix                                                |
| ------------------------------------ | -------------------------------------------------- |
| `ModuleNotFoundError: ultralytics`   | Activate venv first: `source .venv/bin/activate`   |
| First `/detect` request slow (~10 s) | Ultralytics downloading `yolov8n.pt` — only once   |
| `Port 8000 in use`                   | Pick another: `uvicorn src.api:app --port 8001`    |
| Webcam window doesn't open (macOS)   | Grant Terminal Camera permission                   |
| `MPS out of memory` during training  | Use `--device cpu` or smaller `--batch`            |

---

## What to do next (submission checklist)

1. **Train on a Roboflow grocery dataset** (Step 7) — main missing piece for a
   strong submission. Without this, the model is still COCO-only.
2. **Record your 3–10 min demo video** — see the checklist in `README.md`
   under "What to record for the demo video".
3. **`git init`**, commit everything, **push to GitHub**, then submit the
   GitHub link + video link as the task brief requires.
