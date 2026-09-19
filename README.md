# BLUR

**Automatic privacy protection for images — blurring faces, license plates, QR codes, and sensitive text.**

BLUR is a single Google Colab notebook that takes an image, detects personally identifiable information (faces, license plates, QR codes, and sensitive text like the fields on an ID card), and automatically blurs it before the image is saved or returned.

---

## Why BLUR?

Manually blurring personal information in photos and videos doesn't scale, and getting it wrong has real consequences:

- **Legal compliance** — laws like Korea's PIPA, the EU's GDPR, and California's CCPA regulate how faces, names, and license plates can be shown.
- **Platform policy** — sites like YouTube can restrict or remove content that exposes private information without consent.
- **Preventing misuse** — unblurred images/video can be repurposed for deepfakes, impersonation, and phishing.
- **Ethical AI/data use** — de-identifying data before it's used for training helps avoid privacy violations.

BLUR automates this so privacy protection isn't a manual, error-prone afterthought.

## Who it's for

| User | Use case |
|---|---|
| **General users** | Auto-blur faces, plates, and personal data before posting on social media |
| **Content creators** | Blur unexpected faces/backgrounds during livestreams to stay within platform rules |
| **Businesses & government agencies** | Anonymize people and private details in security footage to comply with privacy laws |

## How it works

The pipeline runs in two detection stages so that both *visual* objects and *embedded text* get caught:

```
1. Upload a user image
        │
2. Object detection (YOLOv11, trained on Roboflow datasets)
   → detects faces, license plates, and QR codes
   → apply first-stage blur
        │
3. Save intermediate (first-stage) result
        │
4. OCR (Google Vision API)
   → extract all text in the image + its bounding-box location
        │
5. Sensitive-text classification (LLM prompt)
   → decide whether extracted text contains personal/sensitive
     information (e.g. names, addresses, ID numbers)
        │
6. Second-stage blur (OpenCV)
   → blur only the text regions flagged as sensitive
        │
7. Visualize / save / return the final, fully anonymized image
```

**Example (ID card):**

1. Original → 2. YOLO detects face / plate / QR (bounding boxes) → 3. YOLO-based blur applied → 4. OCR detects text regions (name, address, ID#, expiry) → 5. Final result: face, plate, QR, and sensitive text fields are all blurred, while non-sensitive fields (e.g. "Sex: Male", "Height") remain readable.

## Core technologies

| Component | Purpose |
|---|---|
| **YOLOv11** (custom-trained, via Roboflow) | Object detection for faces, license plates, and QR codes |
| **Google Vision OCR API** | Extracts text and its location from the image |
| **LLM prompting** (Gemini) | Classifies extracted text as sensitive vs. non-sensitive |
| **OpenCV** | Applies the actual blur/pixelation to detected regions |

**Development environment:** Google Colab, TPU v2-8 runtime.

## Dataset overview

Three custom YOLOv11 models were trained on Roboflow-hosted datasets:

| | Face | License Plate | QR Code |
|---|---|---|---|
| **Total photos** | 2,859 | 2,560 | 3,211 |
| **Train / Valid / Test** | 2,318 / 356 / 185 | 2,294 / 159 / 107 | 2,790 / 245 / 176 |
| **Variety captured** | Front-facing, side view, distant/small faces, various angles | Multiple states, custom-designed plates, side angles | Digital, printed, damaged, and stylized QR codes |

## Model performance

Face detection: YOLOv11 vs. Google Cloud Vertex AI AutoML vs. Google Vision API

| Metric | YOLOv11 | Vertex AI AutoML | Google Vision API |
|---|---|---|---|
| mAP@50 | **84.0%** | 67.1% | — |
| Recall | **75.2%** | 52.7% | — |
| Precision | 90.8% | **98.1%** | Both strong |
| Speed | Moderate (local, GPU) | Moderate (cloud) | Fast (cloud API) |
| Cost | Free (local execution) | Expensive (usage-based) | Expensive (API-based) |

**Takeaway:** the custom-trained YOLOv11 model gave the best balance of accuracy (mAP, recall) and cost, since it's trained specifically on the target classes and runs locally. Vertex AI AutoML is faster to prototype with (no-code) but is less accurate and costlier to run. Google Vision API is fast and easy to integrate but isn't customizable and doesn't expose standard detection metrics.

*(Loss curves for the face, QR code, and license plate detection models — box loss, class loss, object loss — are tracked over training epochs to monitor convergence and check for overfitting.)*

## Metrics glossary

- **Precision** = TP / (TP + FP) — of everything the model flagged, how much was actually correct.
- **Recall** = TP / (TP + FN) — of everything that should have been flagged, how much did the model catch.
- **mAP (mean Average Precision)** = average AP across all classes, integrating the precision-recall curve — a single number summarizing overall detection quality.

## What makes BLUR different

Most existing tools only recognize faces and require the full video/image to be uploaded ahead of time, with no support for other sensitive elements (plates, address text, etc.). BLUR aims to:

- **Detect comprehensively** — faces, license plates, QR codes, *and* sensitive text, not just faces.
- **Run on flexible, scalable infrastructure** — built on GCP so it can be adapted to different deployment environments.
- **Process near real-time** — more practical than conventional auto-blur systems that require a full pre-upload.

## Getting started

1. Open the notebook in Google Colab.
2. Add your API credentials (Google Vision OCR API key / LLM API key) as needed.
3. Upload an image when prompted.
4. Run all cells — the notebook will output the original image alongside intermediate detection/blur stages and the final anonymized result.
