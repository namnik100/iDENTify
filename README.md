# iDENTify

**iDENTify** is an iOS app that uses on-device machine learning to detect dental cavities in real time. Point your camera at an intraoral photo, tap Analyze, and the app returns bounding-box detections with per-cavity severity ratings and treatment urgency scores — all without sending any image to a server.

---

## How it works

1. **Capture** — take a photo with the camera or pick one from your library
2. **Preprocess** — the image is letterbox-resized to 640 × 640 and normalized for YOLO input
3. **Infer** — a YOLOv11n model runs on-device via TensorFlow Lite
4. **Parse** — YOLO output tensors (`[1, 84, 8400]`) are decoded and filtered with non-maximum suppression
5. **Review** — detected cavities are shown with bounding boxes, confidence scores, severity levels, and recommended next steps

---

## Tech stack

| Layer | Details |
|---|---|
| Language | Swift 5.0 |
| UI | SwiftUI |
| ML runtime | TensorFlow Lite (CocoaPods) |
| Model | YOLOv11n — converted to `.tflite` |
| Architecture | MVVM |
| Min deployment | iOS 17.0 |

---

## Project structure

```
iDENTify/
├── iDENTify/                        # App source
│   ├── iDENTifyApp.swift            # Entry point
│   ├── ContentView.swift            # Home screen
│   ├── CameraViewModel.swift        # Central state + ML orchestration
│   ├── ImagePicker.swift            # Camera / photo library bridge
│   ├── ImagePreviewView.swift       # Review screen before analysis
│   ├── ResultsView.swift            # Detection results + bounding boxes
│   ├── CavityDetectionCard.swift    # Per-cavity UI card
│   ├── CavityDetectionService.swift # TFLite inference engine
│   ├── CavityDetectionModels.swift  # Data models
│   ├── ImageProcessingUtils.swift   # Letterboxing, NMS, normalization
│   ├── NavigationState.swift        # App navigation state machine
│   └── aviScan-YOLOv11n-v1.0.tflite
├── iDENTify.xcodeproj/
├── Podfile
└── README.md
```

---

## ML model

The app loads **aviScan-YOLOv11n-v1.0.tflite**, a YOLOv11n model trained on intraoral dental imagery and exported to TensorFlow Lite for on-device inference. The model outputs an `[1, 84, 8400]` tensor (4 bbox coords + 80 class scores × 8400 anchors) which `CavityDetectionService` decodes, confidence-filters, and deduplicates with NMS before surfacing results.

---

## Getting started

### Prerequisites

- Xcode 15+
- CocoaPods (`brew install cocoapods`)
- iPhone or iOS Simulator running iOS 17+

### Install & run

```bash
git clone https://github.com/yourusername/iDENTify.git
cd iDENTify
pod install
open iDENTify.xcworkspace
```

Select a target device and press **Run**.

---

## Architecture

```
ContentView
    └── CameraViewModel  ──────────────────────────┐
         ├── ImagePicker (camera / library)         │
         ├── ImagePreviewView                       │
         │    └── triggers analysis                 │
         └── ResultsView                            │
              └── bounding box overlay              │
                                                    ▼
                              CavityDetectionService
                               ├── TFLite Interpreter
                               ├── ImageProcessingUtils
                               │    ├── letterbox resize → 640×640
                               │    ├── pixel buffer → float tensor
                               │    └── NMS
                               └── CavityDetectionModels
                                    ├── CavityDetection
                                    ├── DetectionResult
                                    └── CavitySeverity / UrgencyLevel
```
