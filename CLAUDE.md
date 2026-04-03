# Skeletrack

## Project Overview
A pip-installable Python library for extracting multi-person skeleton trajectories from videos.
Originated from the VideoScreener project (ASD child screening system) but designed as a general-purpose tool.

## Core Pipeline
```
Video → Person Detection (YOLO) → Multi-Person Tracking (ByteTrack) → Pose Estimation (MediaPipe) → Skeleton Trajectories
```

## Key Design Decisions
- **Pose estimation runs AFTER tracking + filtering** to avoid wasting compute on discarded tracks
- **Synchronous API** (no threading) — users handle concurrency themselves
- **Pluggable backends** via registry pattern: detectors, trackers, pose estimators
- **Lazy imports** for optional dependencies (ultralytics, mediapipe, torch)

## API Style
```python
import skeletrack
tracks = skeletrack.extract("video.mp4")          # one-liner
tracks.filter(min_duration=2.0).save("out.npy")   # chainable
```

## Key Classes
- `TrackCollection` — main return type, wraps list of Track, supports filter/save/to_dataframe
- `Track` — single person trajectory (frames, timestamps, bboxes, skeletons)
- `Skeleton` — single frame keypoints as dict of named groups (pose, left_hand, right_hand, face)
- `Pipeline` — reusable pipeline object for processing multiple videos

## Project Structure
```
src/skeletrack/
├── core/       — Pipeline, PipelineConfig, VideoReader
├── data/       — Skeleton, Track, TrackCollection, BBox
├── detection/  — DetectorBackend ABC + YOLO implementation
├── tracking/   — TrackerBackend ABC
├── pose/       — PoseBackend ABC + MediaPipe implementation
├── filters/    — TrackFilter ABC + motion/duration/scene filters
├── repair/     — Trajectory repair/interpolation (WIP)
├── io/         — npy/coco/dataframe serialization
└── viz/        — Skeleton rendering and video output
```

## Tech Stack
- Detection: YOLOv8 via `ultralytics` (optional dep)
- Tracking: ByteTrack (bundled with ultralytics)
- Pose: MediaPipe Holistic (optional dep)
- Core: numpy + opencv only

## MVP Scope (v0.1)
Core pipeline + npy IO. Filters, repair, viz are next iteration.

## Related Project
VideoScreener at `../VideoScreener/` — the original ASD screening system this was extracted from.
