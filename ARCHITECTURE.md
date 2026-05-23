# PhotoCurator v2 - Architecture & Design Details

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      LOCAL NETWORK / CLOUD                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────┐      ┌──────────────────┐                  │
│  │   Ingest CLI     │      │   Web Browsers   │                  │
│  │  (macOS, manual) │      │  (Next.js + TS)  │                  │
│  │                  │      │                  │                  │
│  │ • File selector  │      │ • Session dash   │                  │
│  │ • Batch upload   │      │ • Cull interface │                  │
│  │ • Progress       │      │ • Browse/filter  │                  │
│  │ • Status polling │      │ • Export         │                  │
│  └────────┬─────────┘      └────────┬─────────┘                  │
│           │                         │                            │
│           │  HTTP/HTTPS             │                            │
│           ├────────────────────────┬┘                            │
│           │                        │                             │
│           v                        v                             │
│  ┌────────────────────────────────────────┐                      │
│  │         CURATOR SERVER (Go)            │                      │
│  │    REST API + Image Processing         │                      │
│  ├────────────────────────────────────────┤                      │
│  │                                        │                      │
│  │  API Endpoints:                        │                      │
│  │  ├─ POST   /api/sessions               │                      │
│  │  ├─ POST   /api/sessions/{id}/upload   │                      │
│  │  ├─ GET    /api/sessions               │                      │
│  │  ├─ GET    /api/sessions/{id}          │                      │
│  │  ├─ GET    /api/sessions/{id}/images   │                      │
│  │  ├─ PATCH  /api/images/{id}/cull       │                      │
│  │  └─ GET    /api/export/{session_id}    │                      │
│  │                                        │                      │
│  └────────┬─────────────┬────────┬────────┘                      │
│           │             │        │                               │
│    ┌──────v──┐   ┌──────v───┐   └────────────┐                  │
│    │ Storage │   │  Analysis │               │                  │
│    │ Service │   │  Queue    │      ┌────────v──────────┐       │
│    ├─────────┤   ├───────────┤      │ Image Processors  │       │
│    │Filesystem   │ Task pool │      ├───────────────────┤       │
│    │data/        │ (goroutines)     │                   │       │
│    │images/      │ • Analyze │      │ • Sharpness       │       │
│    │sessions/    │ • Export  │      │   (Laplacian)     │       │
│    │thumbs/      │           │      │ • Taxonomies      │       │
│    └─────────┘   └───────────┘      │   (hierarchical)  │       │
│       │                             │ • Metadata embed  │       │
│       └──────────────────────┐      │ • Thumbnails      │       │
│                              │      │                   │       │
│                   ┌──────────v──────────┐                │       │
│                   │     SQLite DB       │ (or PostgreSQL)│       │
│                   ├─────────────────────┤                │       │
│                   │ • Sessions          │                │       │
│                   │ • Images            │                │       │
│                   │ • Analysis results  │                │       │
│                   │ • Curation state    │                │       │
│                   │ • Taxonomies        │                │       │
│                   └─────────────────────┘                │       │
│                                                           │       │
│                          ┌────────────────────────────────┘       │
│                          └────────────────────────────────┐       │
│                                                          │       │
│                   (REST endpoints ←────────────────────┘       │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### Flow 1: Photo Ingestion (CLI)

```
User runs: photo-curator ingest --source ~/Photos

         ├─ [Select/scan source directory]
         │  └─ Find JPEG, RAW, PNG, HEIC, TIFF
         │     └─ Display file count & total size
         │
         ├─ [Create session on server]
         │  ├─ POST /api/sessions
         │  │  { name: "...", description: "..." }
         │  └─ Receive session_id
         │
         ├─ [Batch upload 1-8 files in parallel]
         │  ├─ POST /api/sessions/{session_id}/upload
         │  │  multipart: [image files + metadata]
         │  │
         │  ├─ Server receives, stores locally
         │  ├─ Enqueues analysis tasks
         │  └─ Returns: [image IDs, status]
         │
         ├─ [Poll for analysis progress]
         │  ├─ GET /api/sessions/{session_id}
         │  │  └─ totalImages: 247, analyzedImages: 156
         │  │
         │  └─ Display progress bar & ETA
         │
         └─ [Ingest complete]
            └─ Display session ID
            └─ Show: [Open Web UI] [New Session] [Quit]
```

### Flow 2: Server-Side Analysis

```
Server receives image batch

         ├─[1] Store Image
         │    └─ Write to data/images/{session_id}/
         │
         ├─[2] Extract Metadata
         │    ├─ Read EXIF (camera, ISO, shutter, etc.)
         │    ├─ Read creation timestamp
         │    └─ Store in .json sidecar
         │
         ├─[3] Enqueue Analysis Tasks (async goroutines)
         │    ├─ Task: compute_sharpness
         │    ├─ Task: categorize_subjects
         │    ├─ Task: generate_thumbnails
         │    └─ Task: embed_metadata
         │
         ├─[4] Async Processing
         │    │
         │    ├─ Sharpness Scorer
         │    │  ├─ Load image
         │    │  ├─ Apply Laplacian filter
         │    │  ├─ Compute variance
         │    │  └─ Normalize to 0-100 scale
         │    │     └─ Result: score 87, passing=true
         │    │
         │    ├─ Subject Categorizer
         │    │  ├─ Load image (or use cached features)
         │    │  ├─ Run hierarchical taxonomy classifier
         │    │  │  └─ Mammal → Dog → Labrador (confidence 0.96)
         │    │  └─ Extract multi-level categories
         │    │     └─ Result: [Mammal/0.99, Dog/0.98, Labrador/0.96]
         │    │
         │    ├─ Thumbnail Generator
         │    │  ├─ Generate 150px version
         │    │  ├─ Generate 400px version
         │    │  └─ Store in data/thumbnails/
         │    │
         │    └─ Metadata Embedder
         │       ├─ Write subjects to XMP tags
         │       ├─ Write sharpness metadata
         │       └─ Write taxonomy hierarchy
         │
         ├─[5] Update Database
         │    └─ INSERT/UPDATE images, analysis results
         │       └─ Queryable metadata for UI
         │
         └─ [Analysis Complete]
            └─ WebSocket notify CLI/UI of progress
```

### Flow 3: Web UI Culling

```
User navigates to web UI

         ├─ Load Sessions Dashboard
         │  └─ GET /api/sessions
         │     └─ Display all sessions with stats
         │
         ├─ [User clicks session]
         │  └─ GET /api/sessions/{id}
         │     └─ Display overview + filters
         │
         ├─ [Cull Interface Loads]
         │  ├─ GET /api/sessions/{id}/images?limit=50
         │  ├─ Display main image
         │  ├─ Load filmstrip (25% visible)
         │  ├─ Show sharpness bar
         │  ├─ Show taxonomy breadcrumb (Mammal > Dog > Labrador)
         │  └─ Show confidence scores per level
         │
         ├─ [User navigates / culls]
         │  │
         │  ├─ → Arrow Right
         │  │  └─ Load next undecided image
         │  │
         │  ├─ K (Keep)
         │  │  └─ PATCH /api/images/{id}/cull
         │  │     { decision: "keep", rating: 5 }
         │  │
         │  ├─ X (Reject)
         │  │  └─ PATCH /api/images/{id}/cull
         │  │     { decision: "reject" }
         │  │
         │  ├─ Z (Undo)
         │  │  └─ Revert last decision from DB
         │  │
         │  └─ [1-5] Star Rating
         │     └─ Update rating without changing decision
         │
         ├─ [Filter & Browse]
         │  ├─ Sharpness >= 70 ✓
         │  ├─ Taxonomy: [Mammal ✓] [Dog ✓]
         │  └─ GET /api/sessions/{id}/images?filter=...
         │
         └─ [Export]
            ├─ GET /api/export/{session_id}?format=jpeg
            │  └─ Download ZIP of culled images
            ├─ GET /api/export/{session_id}?format=csv
            │  └─ Download metadata CSV with taxonomy
            └─ GET /api/export/{session_id}?format=pdf
               └─ Download contact sheet
```

---

## Database Schema (SQLite or PostgreSQL)

### Tables

#### `sessions`
```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  name VARCHAR(255),
  photographer_id VARCHAR(255),
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status VARCHAR(50),  -- "ingesting", "analyzing", "complete"
  total_images INT,
  images_analyzed INT,
  metadata JSON  -- custom fields
);
```

#### `images`
```sql
CREATE TABLE images (
  id TEXT PRIMARY KEY,
  session_id TEXT REFERENCES sessions(id) ON DELETE CASCADE,
  original_filename VARCHAR(255),
  storage_path VARCHAR(512) UNIQUE,
  mime_type VARCHAR(50),
  file_size_bytes BIGINT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ingested_at TIMESTAMP,
  
  -- Analysis results
  sharpness_score INT,  -- 0-100
  sharpness_passing BOOLEAN,
  taxonomy_data JSON,  -- [{"level": "Mammal", "label": "Mammal", "confidence": 0.99}, ...]
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
  id TEXT PRIMARY KEY,
  image_id TEXT REFERENCES images(id) ON DELETE CASCADE,
  job_type VARCHAR(50),  -- "sharpness", "taxonomy", "thumbnail", "embed"
  status VARCHAR(50),  -- "pending", "running", "complete", "failed"
  result JSON,
  error_message TEXT,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  
  INDEX (image_id),
  INDEX (status),
  INDEX (started_at)
);
```

#### `taxonomies`
```sql
CREATE TABLE taxonomies (
  id TEXT PRIMARY KEY,
  name VARCHAR(255),
  description TEXT,
  config JSON,  -- hierarchical levels, classification model, calibration
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX (name)
);
```

---

## API Endpoints (Go REST API)

### Sessions API

```
POST /api/sessions
├─ Request: JSON
│  ├─ name: str
│  ├─ description: str
│  └─ taxonomy_id: str (optional, default: standard)
│
├─ Response: 200 OK
│  ├─ id: UUID
│  ├─ name: str
│  ├─ created_at: timestamp
│  └─ status: "creating"
│
└─ Error: 400, 500
```

```
GET /api/sessions
├─ Query: ?limit=50&offset=0
├─ Response: 200 OK
│  └─ sessions: [
│     { id, name, total_images, culled_count, status, created_at, ... }
│  ]
└─ Auth: Bearer token
```

```
GET /api/sessions/{session_id}
├─ Response: 200 OK
│  ├─ id: UUID
│  ├─ name: str
│  ├─ totalImages: int
│  ├─ imagesAnalyzed: int
│  ├─ statsSharpness: { passing: 220, failing: 27 }
│  ├─ statsTaxonomy: [
│  │   { level: "Mammal", top_categories: [{ label: "Mammal", count: 215 }, ...] }
│  │ ]
│  ├─ statsCulling: { kept: 125, rejected: 122, pending: 0 }
│  └─ createdAt: timestamp
└─ Auth: Bearer token
```

### Upload API

```
POST /api/sessions/{session_id}/upload
├─ Request: multipart/form-data
│  ├─ files: File[]
│  └─ metadata: JSON (EXIF, timestamps, etc.)
│
├─ Response: 200 OK
│  └─ uploaded: [
│     { imageId: UUID, status: "stored", analysisQueued: bool }
│  ]
│
└─ Error: 400, 413, 500
```

### Images API

```
GET /api/sessions/{session_id}/images
├─ Query: 
│  ├─ limit=50
│  ├─ offset=0
│  ├─ filter.sharpness_min=70
│  ├─ filter.taxonomy_path=Mammal.Dog
│  ├─ filter.curation_status=pending
│  └─ sort_by=created_at
│
├─ Response: 200 OK
│  └─ images: [
│     {
│       id, originalFilename, storagePath,
│       sharpnessScore, taxonomy: [{ level, label, confidence }, ...],
│       exif: { camera, iso, shutter, aperture },
│       curation: { decision, rating, decidedAt },
│       thumbnails: { small: url, medium: url }
│     }
│  ]
└─ Auth: Bearer token
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
└─ Auth: Bearer token or session cookie
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

```go
func ComputeSharpnessScore(imagePath string, cameraModel string) int {
    // Load image
    img, err := imaging.Open(imagePath)
    if err != nil {
        return 0
    }
    
    // Convert to grayscale
    gray := imaging.Grayscale(img)
    
    // Apply Laplacian filter
    laplacian := applyLaplacian(gray)
    
    // Compute variance
    variance := computeVariance(laplacian)
    
    // Calibration: camera-specific thresholds
    calibration := getCameraCalibration(cameraModel)
    
    // Normalize to 0-100
    normalized := math.Max(0, math.Min(100,
        ((variance - calibration.Min) / 
         (calibration.Max - calibration.Min)) * 100,
    ))
    
    return int(normalized)
}

// Example calibration per camera model
var CameraCalibrations = map[string]struct{
    Min float64
    Max float64
}{
    "Canon EOS R5": {Min: 150.0, Max: 12000.0},
    "Nikon Z9": {Min: 120.0, Max: 9500.0},
    "Sony A7RV": {Min: 140.0, Max: 11000.0},
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

## Subject Categorization (Hierarchical Taxonomy)

### Flexible Taxonomy System

PhotoCurator v2 uses a **configurable hierarchical taxonomy** instead of fixed object detection.

```
Taxonomy Structure Example:

Living Things
├─ Mammal
│  ├─ Dog
│  │  ├─ Labrador
│  │  ├─ GoldenRetriever
│  │  └─ GermanShepherd
│  ├─ Cat
│  └─ Human
├─ Bird
└─ Fish

Inanimate
├─ Building
├─ Vehicle
│  ├─ Car
│  ├─ Truck
│  └─ Motorcycle
└─ Landscape
   ├─ Mountain
   ├─ Forest
   └─ Beach
```

### Taxonomy Configuration (JSON)

```json
{
  "id": "default-2026",
  "name": "Default Taxonomy",
  "root": {
    "label": "Things",
    "levels": ["Category", "Subcategory", "Specific"],
    "children": [
      {
        "label": "Living Things",
        "children": [
          {
            "label": "Mammal",
            "children": [
              {
                "label": "Dog",
                "children": [
                  { "label": "Labrador" },
                  { "label": "GoldenRetriever" }
                ]
              }
            ]
          }
        ]
      }
    ]
  },
  "classifier": {
    "type": "hierarchical",
    "model": "resnet50-imagenet-hierarchy",
    "confidence_threshold": 0.70
  }
}
```

### Classification Output

```json
{
  "image_id": "uuid...",
  "taxonomy_path": "Living Things > Mammal > Dog > Labrador",
  "hierarchy": [
    { "level": 0, "label": "Living Things", "confidence": 0.99 },
    { "level": 1, "label": "Mammal", "confidence": 0.98 },
    { "level": 2, "label": "Dog", "confidence": 0.97 },
    { "level": 3, "label": "Labrador", "confidence": 0.96 }
  ],
  "alternatives": [
    { "path": "...", "confidence": 0.12 },
    { "path": "...", "confidence": 0.08 }
  ]
}
```

### Integration with UI

The Next.js frontend displays:
1. **Taxonomy breadcrumb**: Living Things > Mammal > Dog > Labrador
2. **Confidence per level**: Shows individual confidence scores
3. **Filtering**: Filter by any taxonomy level (e.g., "all Dogs")
4. **Drill-down**: Click to expand/collapse hierarchy

---

## Deployment Architecture

### Development (Single Machine)

```
┌─ Go Server (localhost:8080)
├─ SQLite DB (data/curator.db)
├─ File storage (data/images/, data/thumbnails/)
├─ Next.js dev server (localhost:3000)
└─ macOS CLI (binary)
```

### Production (Containerized)

```
Docker Compose:
├─ curator-server (Go)
├─ curator-db (PostgreSQL)
├─ curator-web (Next.js)
├─ nginx (reverse proxy)
└─ volumes for data persistence
```

---

## Performance Targets

| Operation | Target | Notes |
|-----------|--------|-------|
| Sharpness analysis | <100ms | Per image, async |
| Taxonomy classification | <200ms | Per image, async |
| Upload batch (8 files) | <5s | Network + storage |
| Web UI page load | <2s | Dashboard, filters |
| Culling speed | 5-10 images/min | Keyboard-driven |

---

## Technology Stack Summary

| Layer | Component | Technology |
|-------|-----------|-----------|
| **Frontend** | Web UI | Next.js + TypeScript + Tailwind CSS |
| **API** | REST Server | Go (Gin or Echo framework) |
| **Database** | SQLite (dev) / PostgreSQL (prod) | SQL |
| **Storage** | File system | Local disk or S3 |
| **Image Processing** | Sharpness, thumbnails | Go imaging libraries (go-image, imaging) |
| **Taxonomy Classification** | Hierarchical categorization | TensorFlow Lite or ONNX Runtime (Go bindings) |
| **Ingest** | CLI tool | Go (standalone binary for macOS) |
| **Deployment** | Docker | Docker Compose |

---

*Architecture documentation for PhotoCurator v2 (2026). For API details, see API.md.*
