# Camera-Based Grocery Product Detection (YOLOv8 + FastAPI)

A small computer-vision service that detects common grocery items (rice, oil
bottles, soap, packets, etc.) in an uploaded image and returns the result as
JSON. Built on **Ultralytics YOLOv8** and served with **FastAPI**.

> Task spec (verbatim, from the hiring brief): see [`TASK_BRIEF.md`](./TASK_BRIEF.md).

---

## Features

- `POST /detect` — upload an image, get JSON detections
- Standalone CLI inference (`python -m src.infer`)
- Fine-tuning script for any Roboflow-format YOLOv8 grocery dataset
- **Bonus:** Real-time webcam + video file detection (`python -m src.webcam`)
- Works out of the box on **CPU / NVIDIA CUDA / Apple Silicon MPS**
- No Docker required — `pip install` and run

---

## Project structure

```
rakib_job_project/
├── README.md                <- this file
├── TASK_BRIEF.md            <- the hiring task description
├── requirements.txt
├── data/                    <- (your dataset goes here)
│   └── data.yaml
├── models/
│   └── best.pt              <- created by training; the API auto-loads it
├── samples/                 <- put test images here
└── src/
    ├── detector.py          <- shared model wrapper
    ├── api.py               <- FastAPI app (POST /detect)
    ├── infer.py             <- CLI: detect on one image
    ├── train.py             <- fine-tune YOLOv8n on your dataset
    └── webcam.py            <- bonus: webcam / video stream detection
```

---

## 1. Setup

```bash
# clone the repo, then:
cd rakib_job_project

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

That's it — no Docker, no CUDA setup required. The first time you run the
service, Ultralytics will auto-download the pretrained `yolov8n.pt` weights
(~6 MB).

---

## 2. Run the API

```bash
uvicorn src.api:app --host 0.0.0.0 --port 8000 --reload
```

The service is now live at <http://localhost:8000>.

- `GET /`         — service info and which model is loaded
- `GET /health`   — health check
- `GET /docs`     — interactive Swagger UI (great for the demo video)
- `POST /detect`  — upload an image, get detections

### Test it

**With curl:**

```bash
curl -X POST http://localhost:8000/detect \
  -F "file=@samples/test.jpg"
```

**With Python:**

```python
import requests
with open("samples/test.jpg", "rb") as f:
    r = requests.post("http://localhost:8000/detect", files={"file": f})
print(r.json())
```

**With Swagger / browser:** open <http://localhost:8000/docs>, expand
`POST /detect`, click **Try it out**, upload an image, hit **Execute**.

### Response shape

```json
{
  "detections": [
    {"class": "rice", "confidence": 0.92, "bbox": [142.1, 88.0, 401.3, 320.5]},
    {"class": "oil",  "confidence": 0.88, "bbox": [510.0, 102.4, 690.7, 470.2]}
  ],
  "model": "best.pt",
  "custom_trained": true
}
```

The `bbox` is `[x1, y1, x2, y2]` in pixel coordinates of the input image. The
core `class` + `confidence` pair matches the response shape requested in the
task brief; `bbox` and `model` are added for usefulness and traceability.

---

## 3. CLI inference (no server)

```bash
python -m src.infer samples/test.jpg
python -m src.infer samples/test.jpg --weights models/best.pt --conf 0.4
```

Prints the same JSON payload to stdout.

---

## 4. Real-time webcam / video (bonus)

```bash
# webcam (camera index 0)
python -m src.webcam

# different camera
python -m src.webcam --source 1

# video file
python -m src.webcam --source path/to/video.mp4
```

A window opens with live bounding boxes and an FPS counter. Press **`q`** to
quit.

---

## 5. Training on your own grocery dataset

### Dataset format

YOLOv8 / Roboflow export format works directly. Drop your dataset into
`data/`:

```
data/
├── data.yaml
├── train/
│   ├── images/*.jpg
│   └── labels/*.txt
└── valid/
    ├── images/*.jpg
    └── labels/*.txt
```

`data.yaml` looks like:

```yaml
path: ./data
train: train/images
val: valid/images

nc: 4
names: ["rice", "oil", "soap", "packet"]
```

### Where to get a dataset

- **Roboflow Universe** — <https://universe.roboflow.com> → search "grocery
  detection" or "supermarket products". Export as **YOLOv8** and unzip into
  `data/`.
- **SKU-110K** — large-scale retail shelf dataset (heavier, optional).

### Train

```bash
python -m src.train --data data/data.yaml --epochs 50 --imgsz 640
```

Useful flags:

```bash
--epochs 100         # more for higher accuracy
--imgsz 640          # input image size
--batch 16           # reduce if you OOM
--base yolov8n.pt    # or yolov8s.pt / yolov8m.pt for more capacity
--device mps         # Apple Silicon, or "cpu", or "0" for CUDA
```

When training finishes, the best weights are automatically copied to
`models/best.pt`. The API and CLI auto-detect this file and prefer it over the
COCO pretrained model — **just restart `uvicorn` and you're using your trained
model**.

---

## Model details

| Aspect          | Value                                                             |
| --------------- | ----------------------------------------------------------------- |
| Architecture    | YOLOv8n (Ultralytics)                                             |
| Default weights | `yolov8n.pt` (COCO, auto-downloaded on first run)                 |
| Custom weights  | `models/best.pt` (produced by `src.train`, auto-loaded if present)|
| Input size      | 640×640                                                           |
| Output          | bounding boxes + class + confidence (JSON)                        |
| Device          | Auto: CUDA → MPS → CPU                                            |

### A note about classes when running with the default COCO weights

COCO does not include grocery-specific classes like *rice* or *oil*. To make
the demo meaningful **without** any training, `src/detector.py` contains a
small alias map (e.g. COCO `bottle` → `oil`, COCO `bowl` → `rice`). This is
purely cosmetic for the pretrained-only flow.

Once you train on a real grocery dataset and `models/best.pt` is present, the
alias map is bypassed and the model's actual learned class names are
returned. **For a real submission, train on a grocery dataset.**

---

## Quick smoke test (no dataset required)

```bash
# 1. install
pip install -r requirements.txt

# 2. grab any test image
curl -L -o samples/test.jpg https://ultralytics.com/images/bus.jpg

# 3. CLI test
python -m src.infer samples/test.jpg

# 4. API test
uvicorn src.api:app --port 8000 &
sleep 5
curl -s -X POST http://localhost:8000/detect -F "file=@samples/test.jpg" | python -m json.tool
```

---

## What to record for the demo video (3–10 min)

1. Walk through the project structure (`README.md`, `src/`).
2. Open `src/detector.py` and `src/api.py`; explain the YOLO call and the API.
3. Start the API (`uvicorn src.api:app --port 8000`).
4. Open <http://localhost:8000/docs>, upload an image to `/detect`, show the
   JSON response.
5. (Optional) Run `python -m src.webcam` and show real-time detection.
6. (Optional) Show `python -m src.train --data data/data.yaml --epochs 1` to
   prove training works end-to-end.

---

## Troubleshooting

- **`ModuleNotFoundError: ultralytics`** — you didn't activate the venv or
  install requirements. `source .venv/bin/activate && pip install -r requirements.txt`.
- **First request to `/detect` is slow** — Ultralytics is downloading
  `yolov8n.pt` (~6 MB). Subsequent requests are fast.
- **Webcam window doesn't open on macOS** — Terminal needs *Camera* permission
  (System Settings → Privacy & Security → Camera).
- **`MPS backend out of memory`** — pass `--device cpu` or reduce `--batch`.
