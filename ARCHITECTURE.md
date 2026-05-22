# PhotoCurator - Architecture & Design Details

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      LOCAL NETWORK / CLOUD                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────┐      ┌──────────────────┐                  │
│  │   Ingest App     │      │   Web Browsers   │                  │
│  │  (macOS SwiftUI) │      │   (React + TS)   │                  │
│  │                  │      │                  │                  │
│  │ • Dir selector   │      │ • Session dash   │                  │
│  │ • Upload queue   │      │ • Cull interface │                  │
│  │ • Progress UI    │      │ • Browse/filter  │                  │
│  │ • Metadata cache │      │ • Export         │                  │
│  └────────┬─────────┘      └────────┬─────────┘                  │
│           │                         │                            │
│           │  HTTP/HTTPS             │                            │
│           ├────────────────────────┬┘                            │
│           │                        │                             │
│           v                        v                             │
│  ┌────────────────────────────────────────┐                      │
│  │         CURATOR SERVER                 │                      │
│  │    (FastAPI + Python)                  │                      │
│  ├────────────────────────────────────────┤                      │
│  │                                        │                      │
│  │  API Endpoints:                        │                      │
│  │  ├─ POST /api/ingest/upload            │                      │
│  │  ├─ GET  /api/ingest/status/{id}       │                      │
│  │  ├─ GET  /api/sessions                 │                      │
│  │  ├─ GET  /api/sessions/{id}/images     │                      │
│  │  ├─ PATCH /api/images/{id}/cull        │                      │
│  │  └─ GET  /api/export/{session_id}      │                      │
│  │                                        │                      │
│  └────────┬─────────────┬────────┬────────┘                      │
│           │             │        │                               │
│    ┌──────v──┐   ┌──────v───┐   └──────────────┐                 │
│    │ Storage │   │  Tasks   │                  │                 │
│    │ Service │   │  Queue   │          ┌───────v─────────┐       │
│    ├─────────┤   ├──────────┤          │ Image Processors│       │
│    │Filesystem   │ Celery   │          ├────────────────┤       │
│    │data/        │ Redis    │          │                │       │
│    │images/      │          │          │ • Sharpness    │       │
│    │sessions/    │ • Analyze│          │ • YOLO v8      │       │
│    │thumbs/      │ • Export │          │ • Thumbnails   │       │
│    └─────────┘   └──────────┘          │ • Metadata     │       │
│       │                                 └────────────────┘       │
│       └─────────────────────────────┐                            │
│                                     │                            │
│                          ┌──────────v─────────┐                  │
│                          │  PostgreSQL DB     │                  │
│                          ├────────────────────┤                  │
│                          │ • Sessions         │                  │
│                          │ • Images           │                  │
│                          │ • Analysis results │                  │
│                          │ • Curation state   │                  │
│                          └────────────────────┘                  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### Flow 1: Photo Ingestion

```
User opens Ingest App
         │
         ├─ [Select Source Directory]
         │
         ├─ [Scan for Images]
         │  └─ Find JPEG, RAW, PNG, HEIC, TIFF
         │     └─ Display count & size
         │
         ├─ [Start Upload]
         │  ├─ Read metadata from each file
         │  ├─ Upload 4-8 in parallel (chunked if large)
         │  │  └─ POST /api/ingest/upload
         │  │     └─ Server receives, stores locally, enqueues analysis
         │  │
         │  └─ Poll GET /api/ingest/status/{session_id}
         │     ├─ Show progress: 45/247
         │     ├─ Show speed: 2.3 MB/s
         │     └─ Show ETA: 20 minutes
         │
         └─ [Ingest Complete]
            ├─ Display session ID
            ├─ Show QR code to web UI
            └─ Offer: [Open Web UI] [Start New] [Quit]
```

### Flow 2: Server-Side Analysis

```
Server receives image batch
         │
         ├─[1] Store Image
         │    └─ Write to data/images/{session_id}/
         │
         ├─[2] Extract Metadata
         │    ├─ Read EXIF (camera, ISO, shutter, etc.)
         │    ├─ Read creation timestamp
         │    └─ Store in .json sidecar
         │
         ├─[3] Enqueue Analysis Tasks (Celery)
         │    ├─ Task: sharpness_scorer
         │    ├─ Task: subject_detector
         │    └─ Task: generate_thumbnails
         │
         ├─[4] Async Processing (worker pool)
         │    │
         │    ├─ Sharpness Scorer
         │    │  ├─ Load image
         │    │  ├─ Apply Laplacian filter
         │    │  ├─ Compute variance
         │    │  └─ Normalize to 0-100 scale
         │    │     └─ Result: score 87, passing=true
         │    │
         │    ├─ Subject Detector (YOLO v8)
         │    │  ├─ Load model (preloaded in GPU)
         │    │  ├─ Run inference
         │    │  └─ Extract detections
         │    │     └─ Result: [person (0.98), flowers (0.76)]
         │    │
         │    └─ Thumbnail Generator
         │       ├─ Generate 150px version
         │       ├─ Generate 400px version
         │       └─ Store in data/thumbnails/
         │
         ├─[5] Embed Labels
         │    ├─ Write subjects to EXIF XMP tags
         │    └─ Write sharpness metadata
         │
         ├─[6] Update Database
         │    └─ INSERT INTO images (...) VALUES (...)
         │       └─ Queryable metadata for UI
         │
         └─ [Analysis Complete]
            └─ WebSocket notify client of progress
```

### Flow 3: Web UI Culling

```
User navigates to web UI
         │
         ├─ Load Sessions Dashboard
         │  └─ GET /api/sessions
         │     └─ Display all sessions with stats
         │
         ├─ [User clicks "Wedding Reception"]
         │  └─ GET /api/sessions/{id}/images?limit=50
         │
         ├─ [Cull Interface Loads]
         │  ├─ Display main image
         │  ├─ Load filmstrip (25% visible)
         │  ├─ Show sharpness bar
         │  └─ Show detected subjects
         │
         ├─ [User navigates / culls]
         │  │
         │  ├─ → Arrow Right
         │  │  └─ Current image = next undecided image
         │  │     └─ Load main image
         │  │     └─ Load metadata
         │  │
         │  ├─ K (Keep)
         │  │  └─ PATCH /api/images/{id}/cull
         │  │     { decision: "keep", rating: 5 }
         │  │     └─ Mark in DB
         │  │     └─ Show visual feedback
         │  │
         │  ├─ X (Reject)
         │  │  └─ PATCH /api/images/{id}/cull
         │  │     { decision: "reject" }
         │  │
         │  ├─ Z (Undo)
         │  │  └─ Revert last decision from DB
         │  │
         │  └─ [1-5] Star Rating
         │     └─ Update rating without changing keep/reject
         │
         ├─ [Filter & Browse]
         │  ├─ Sharpness >= 70 ✓
         │  ├─ Subjects: [person ✓] [flowers ✓]
         │  └─ GET /api/sessions/{id}/images?filter=...
         │
         └─ [Export]
            ├─ GET /api/export/{session_id}?format=jpeg
            │  └─ Download ZIP of culled images
            ├─ GET /api/export/{session_id}?format=csv
            │  └─ Download metadata CSV
            └─ GET /api/export/{session_id}?format=pdf
               └─ Download contact sheet PDF
```

---

## Database Schema (PostgreSQL)

### Tables

#### `sessions`
```sql
CREATE TABLE sessions (
  id UUID PRIMARY KEY,
  name VARCHAR(255),
  photographer_id VARCHAR(255),
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  status VARCHAR(50),  -- "ingesting", "analyzing", "complete"
  total_images INT,
  images_analyzed INT,
  metadata JSONB  -- custom fields
);
```

#### `images`
```sql
CREATE TABLE images (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  original_filename VARCHAR(255),
  storage_path VARCHAR(512) UNIQUE,
  mime_type VARCHAR(50),
  file_size_bytes BIGINT,
  created_at TIMESTAMP DEFAULT NOW(),
  ingested_at TIMESTAMP,
  
  -- Analysis results
  sharpness_score INT,  -- 0-100
  sharpness_passing BOOLEAN,
  subjects JSONB,  -- [{"label": "person", "confidence": 0.98}, ...]
  analysis_completed_at TIMESTAMP,
  
  -- Curation decisions
  curation_decision VARCHAR(50),  -- "keep", "reject", null
  curation_rating INT,  -- 0-5 stars
  curation_notes TEXT,
  curation_decided_by VARCHAR(255),
  curation_decided_at TIMESTAMP,
  
  -- EXIF metadata
  exif_camera VARCHAR(255),
  exif_iso INT,
  exif_shutter VARCHAR(50),
  exif_aperture VARCHAR(50),
  exif_focal_length VARCHAR(50),
  
  INDEX (session_id),
  INDEX (curation_decision),
  INDEX (sharpness_score),
  INDEX (analysis_completed_at)
);
```

#### `analysis_jobs`
```sql
CREATE TABLE analysis_jobs (
  id UUID PRIMARY KEY,
  image_id UUID REFERENCES images(id) ON DELETE CASCADE,
  job_type VARCHAR(50),  -- "sharpness", "subject_detect", "thumbnail"
  status VARCHAR(50),  -- "pending", "running", "complete", "failed"
  result JSONB,
  error_message TEXT,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  
  INDEX (image_id),
  INDEX (status),
  INDEX (started_at)
);
```

#### `export_jobs`
```sql
CREATE TABLE export_jobs (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES sessions(id),
  format VARCHAR(50),  -- "jpeg", "csv", "pdf"
  status VARCHAR(50),  -- "queued", "running", "complete", "failed"
  file_url VARCHAR(512),
  created_at TIMESTAMP,
  completed_at TIMESTAMP,
  
  INDEX (session_id),
  INDEX (status)
);
```

---

## API Endpoints (FastAPI)

### Ingest API

```
POST /api/ingest/upload
├─ Request: multipart/form-data
│  ├─ image: File
│  ├─ session_id: str
│  └─ metadata: JSON (EXIF, etc.)
│
├─ Response: 200 OK
│  ├─ imageId: UUID
│  ├─ status: "stored"
│  └─ analysisQueued: bool
│
└─ Error: 400, 413, 500
```

```
GET /api/ingest/status/{session_id}
├─ Response: 200 OK
│  ├─ totalImages: 247
│  ├─ uploadedImages: 156
│  ├─ analyzedImages: 142
│  ├─ speedMbps: 2.3
│  ├─ estimatedMinutesRemaining: 20
│  └─ sessionId: "..."
│
└─ Useful for polling from Ingest app during upload
```

### Sessions API

```
GET /api/sessions
├─ Query: ?limit=50&offset=0
├─ Response: 200 OK
│  └─ sessions: [
│     { id, name, total_images, culled_count, status, ... }
│  ]
└─ Auth: API token (bearer)
```

```
GET /api/sessions/{session_id}
├─ Response: 200 OK
│  ├─ id: UUID
│  ├─ name: str
│  ├─ totalImages: int
│  ├─ statsSharpness: { passing: 220, failing: 27 }
│  ├─ statsSubjects: [{ label: "person", count: 215 }, ...]
│  ├─ statsCulling: { kept: 125, rejected: 122, pending: 0 }
│  └─ createdAt: timestamp
└─ Auth: API token
```

### Images API

```
GET /api/sessions/{session_id}/images
├─ Query: 
│  ├─ limit=50
│  ├─ offset=0
│  ├─ filter.sharpness_min=70
│  ├─ filter.subjects=person,flowers
│  ├─ filter.curation_status=pending
│  └─ sort_by=created_at
│
├─ Response: 200 OK
│  └─ images: [
│     {
│       id, originalFilename, storagePath,
│       sharpnessScore, subjects,
│       exif: { camera, iso, shutter, aperture },
│       curation: { decision, rating, decidedAt },
│       thumbnails: { small: url, medium: url }
│     }
│  ]
└─ Auth: API token
```

```
PATCH /api/images/{image_id}/cull
├─ Request: JSON
│  ├─ decision: "keep" | "reject"
│  ├─ rating: 0-5
│  └─ notes: str (optional)
│
├─ Response: 200 OK
│  └─ image: { ... updated fields ... }
│
└─ Auth: API token (or session cookie)
```

### Export API

```
GET /api/export/{session_id}
├─ Query:
│  ├─ format: "jpeg" | "csv" | "pdf"
│  ├─ filter.decision: "keep" | "reject" | "all"
│  └─ include_originals: bool
│
├─ Response: 200 OK (streaming)
│  └─ Content-Type: application/zip
│     └─ [images with metadata]
│
└─ Spawns background export job
```

---

## Sharpness Scoring Algorithm

### Approach: Laplacian Variance

The Laplacian operator detects edges. A sharp image has high-frequency content (lots of edges). A blurry image is smooth (low-frequency).

```python
def compute_sharpness_score(image_path: str, camera_model: str = None) -> int:
    """
    Compute sharpness score 0-100 using Laplacian variance.
    
    Args:
        image_path: Path to image file
        camera_model: Optional for camera-specific calibration
    
    Returns:
        Sharpness score 0-100
    """
    # Load image
    image = cv2.imread(image_path)
    if image is None:
        return 0
    
    # Convert to grayscale
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    
    # Apply Laplacian filter
    laplacian = cv2.Laplacian(gray, cv2.CV_64F)
    
    # Compute variance
    variance = laplacian.var()
    
    # Calibration: camera-specific thresholds
    # (collected from training set)
    calibration = CAMERA_CALIBRATIONS.get(camera_model, {
        "min": 100.0,    # Minimum variance observed
        "max": 10000.0   # Maximum variance observed
    })
    
    # Normalize to 0-100
    normalized = max(0, min(100, 
        ((variance - calibration["min"]) / 
         (calibration["max"] - calibration["min"])) * 100
    ))
    
    return int(normalized)


# Example calibration per camera model
CAMERA_CALIBRATIONS = {
    "Canon EOS R5": {
        "min": 150.0,
        "max": 12000.0
    },
    "Nikon Z9": {
        "min": 120.0,
        "max": 9500.0
    },
    "Sony A7RV": {
        "min": 140.0,
        "max": 11000.0
    }
}
```

### Calibration Process

1. Collect 100+ images from target camera across various scenes
2. Manually rate each as "sharp" or "blurry"
3. Compute Laplacian variance for each
4. Plot histogram and identify threshold
5. Store min/max in calibration table
6. Monitor false positives and retrain quarterly

### Expected Accuracy

- **Sharp (score >70)**: ~92% precision
- **Blurry (score <40)**: ~88% precision
- **Gray zone (40-70)**: ~70% precision (user should review)

---

## Subject Detection (YOLO v8)

### Model Selection

- **Model**: YOLOv8-m (medium) or YOLOv8-s (small)
- **Input**: 640×640 images
- **Output**: Bounding boxes + confidence scores
- **Execution**: ~1 second per image (GPU), ~5 seconds (CPU)
- **Memory**: ~500MB GPU VRAM

### Detection Classes (v1)

```
Classes (15 core)
├─ person
├─ people_group
├─ hand_single
├─ hand_multiple
├─ food
├─ cake_dessert
├─ drink_beverage
├─ rings_jewelry
├─ flowers_bouquet
├─ decorations_venue
├─ landscape_outdoor
├─ architecture_indoor
├─ backlighting_artistic
├─ motion_blur_effect
└─ other
```

### YOLO Integration

```python
from ultralytics import YOLO

def detect_subjects(image_path: str) -> list[dict]:
    """
    Detect subjects in image using YOLOv8.
    
    Returns:
        [{"label": "person", "confidence": 0.98, "count": 2}, ...]
    """
    model = YOLO("yolov8m.pt")  # Preloaded
    
    results = model(image_path, conf=0.5)
    
    detections = {}
    for result in results:
        for box in result.boxes:
            class_name = result.names[int(box.cls)]
            conf = float(box.conf)
            
            if class_name not in detections:
                detections[class_name] = {"confidence": conf, "count": 1}
            else:
                detections[class_name]["count"] += 1
                detections[class_name]["confidence"] = max(
                    detections[class_name]["confidence"], conf
                )
    
    # Return sorted by confidence
    return [
        {"label": k, "confidence": v["confidence"], "count": v["count"]}
        for k, v in sorted(detections.items(), 
                          key=lambda x: x[1]["confidence"], 
                          reverse=True)
    ]
```

---

## Performance Targets

| Operation | Target | Notes |
|-----------|--------|-------|
| **Ingest Upload** | 2–3 MB/s | Gigabit network, parallel 4-8 |
| **Sharpness Scoring** | <2 sec/img | CPU-based, single-threaded |
| **Subject Detection** | <1 sec/img (GPU), <5 sec/img (CPU) | YOLOv8-m |
| **Thumbnail Gen** | <0.5 sec/img | Parallel, 8-16 workers |
| **Web UI Page Load** | <1 sec | Lazy-load images + filmstrip |
| **Cull Action** | <100 ms | PATCH API response + DB write |
| **Filter Query** | <500 ms | PostgreSQL on 100k+ images |

---

## Security Considerations

### Authentication
- API token via header: `Authorization: Bearer {token}`
- Session cookies for web UI (secure, httpOnly)
- Token rotation every 30 days

### Authorization
- Ingest app authenticated as photographer (one photographer per session)
- Web UI checks session ownership before serving images

### Data Protection
- HTTPS only (cert from Let's Encrypt)
- Encrypt at-rest (AES-256) for sensitive metadata
- No unencrypted transmission of image paths or metadata

### Input Validation
- Validate file types (MIME check + magic bytes)
- Limit upload size (max 500MB per image)
- Sanitize filenames
- Validate JSON payloads against schema

---

## Monitoring & Observability

### Logging
- Structured logging (JSON) to centralized log store
- Levels: DEBUG, INFO, WARN, ERROR
- Metrics: request latency, queue depth, error rates

### Metrics
- Ingest throughput (MB/s, images/sec)
- Analysis completion rate
- Database query latency
- API endpoint response times

### Alerting
- Queue depth >1000 → page on-call
- Sharpness scoring error rate >5% → alert
- Database connection pool exhausted → alert
- API error rate >1% → alert

---

*For additional details, see IMPLEMENTATION.md, API.md, and DEPLOYMENT.md*

