# PhotoCurator - Product Requirements Document

## Executive Summary

**PhotoCurator** is an end-to-end photo ingestion, evaluation, categorization, and culling system designed for photographers and content creators. It consists of two primary components:

1. **PhotoCurator Ingest** (macOS app) - Imports photos from memory cards or directories
2. **PhotoCurator Server** (dedicated server) - Processes, categorizes, and hosts a web UI for review and culling

The system automatically evaluates each imported image for **sharpness** and **subject matter**, then provides a web-based interface for rapid culling and selection workflows.

---

## Problem Statement

Professional and enthusiast photographers often face time-consuming workflows when processing large photo shoots:

- **Manual transfer** of images from camera/cards is tedious and error-prone
- **No automated quality filtering** (blurry/sharp detection) before manual review
- **Manual categorization** and tagging is labor-intensive
- **No centralized review** interface for culling decisions
- **Metadata loss** during ingestion and sorting

PhotoCurator solves these problems by automating ingestion, evaluation, and providing a purpose-built UI for culling.

---

## Goals & Success Criteria

### Primary Goals
1. **Automate ingestion** - one-click import from any source directory
2. **Intelligent filtering** - automatically score sharpness and identify subjects
3. **Streamlined culling** - intuitive web UI for rapid accept/reject decisions
4. **Scale efficiently** - handle 1,000+ images per shoot without degradation
5. **Preserve metadata** - retain EXIF, ratings, and custom labels through the pipeline

### Success Metrics
- Ingest 100+ images in <5 minutes
- Sharpness classification >90% accuracy (vs. manual review)
- Subject detection covers >95% of common scenarios
- Culling workflow <3 seconds per image decision
- Web UI responsive on desktop and tablet

---

## Market / Use Cases

### Primary Users
- **Wedding photographers** - rapid sorting of 1,000+ images per event
- **Event photographers** - categorize by moment/group/lighting
- **Product photographers** - consistent quality checking before delivery
- **Content creators** - organize photo/video shoots by subject and quality
- **Photo libraries** - batch ingest and metadata enrichment

### Example Workflows

#### Workflow 1: Wedding Reception
1. Photographer imports memory card via PhotoCurator Ingest on site
2. System uploads to server, begins analysis
3. That evening, photographer logs into web UI
4. Views images pre-scored for sharpness + grouped by detected subjects (faces, rings, décor, cake, etc.)
5. Culls to final 200 images in 20 minutes
6. Exports final set with metadata intact

#### Workflow 2: Product Shoot
1. Product photography session generates 500 images
2. Photographer runs Ingest app, selects shoot folder
3. Images queue on server within 2 minutes
4. Sharpness scores help identify best technical shots
5. Subject detection flags product placement, lighting, and angle variations
6. Photographer culls to 50 final images using smart filters

---

## Functional Requirements

### Component 1: PhotoCurator Ingest (macOS App)

#### 1.1 Core Behavior
- **Launch**: Native macOS SwiftUI app, single focused UI
- **Input Source**: User selects any directory (memory card, external drive, folder)
- **File Discovery**: Recursively scan for image files (JPEG, RAW, PNG, HEIC, TIFF)
- **Validation**: Skip corrupted or incomplete files; log warnings
- **Transfer**: Copy images to a local staging directory with consistent naming
- **Metadata Preservation**: Retain EXIF, creation date, camera model
- **Upload Trigger**: Once transferred, notify server to begin processing

#### 1.2 User Interface
```
PhotoCurator Ingest
┌─────────────────────────────────┐
│ Select Source Directory          │
│ ┌─────────────────────────────┐ │
│ │ /Volumes/SD_CARD            │ │
│ └─────────────────────────────┘ │
│ [Browse...]                     │
│                                 │
│ ✓ Found 247 images              │
│ ✓ Total size: 3.2 GB            │
│                                 │
│ ┌─────────────────────────────┐ │
│ │ Ingesting: 45/247 (18%)     │ │
│ │ ████████░░░░░░░░░░░░░░     │ │
│ │ Speed: 2.3 MB/s              │ │
│ │ Est. time: 20 minutes         │ │
│ └─────────────────────────────┘ │
│                                 │
│ Server: https://curator.local   │
│ Session: [unique-id]            │
│                                 │
│            [Start]              │
└─────────────────────────────────┘
```

#### 1.3 Configuration
- **Server URL**: Configurable via CLI or UI (defaults to local network)
- **Image Formats**: JPEG, RAW (CR3, NEF, ARW), PNG, HEIC, TIFF (user-configurable priority)
- **Batch Size**: Parallel upload (default 4-8 concurrent)
- **Staging Path**: Configurable local cache directory

#### 1.4 Error Handling
- Gracefully skip corrupted files and log them
- Retry transient network errors (3 attempts)
- Resume interrupted uploads
- Clear UI messaging for failures

---

### Component 2: PhotoCurator Server

#### 2.1 Ingestion Queue
- Receive images from Ingest app
- Store in organized directory structure: `images/[session-id]/[timestamp]-[original-name]`
- Immediately enqueue for analysis

#### 2.2 Image Analysis Pipeline

##### 2.2.1 Sharpness Scoring
- **Tool**: Laplacian variance or local feature density (using OpenCV/PIL)
- **Output**: Score 0–100 (0 = blurry, 100 = sharp)
- **Threshold**: User-configurable threshold for "acceptable" sharpness (default: 65)
- **Execution**: Per-image, <2 seconds

##### 2.2.2 Subject Detection & Embedded Labels
- **Tool**: YOLO v8 or similar embedded model (runs locally, no cloud API)
- **Detection Classes** (extensible):
  - Person / faces
  - Multiple people / group
  - Hand(s) / gesture
  - Food / cake / drinks
  - Rings / jewelry
  - Flowers / décor
  - Landscape / scenery
  - Architecture / indoor
  - Sunlight / backlighting
  - Motion / blur artistic effect
  - Pet / animal
  - Vehicle / transportation
  - Product / still life

- **Output**: List of detected subjects with confidence scores (0–100)
- **Embedded Labels**: Store directly in image EXIF/sidecar as subject tags
- **Execution**: Per-image, <1 second with GPU, <5 seconds CPU

##### 2.2.3 Metadata Enrichment
- Extract EXIF: camera model, ISO, shutter speed, aperture, focal length
- Compute: histogram, color temperature estimate, aspect ratio
- Add ingestion timestamp and session ID

#### 2.3 Storage Schema

```
curator-server/
  data/
    images/
      [session-id]/
        2026-05-22T14-30-15Z_IMG_1234.jpg
        2026-05-22T14-30-15Z_IMG_1234.json  (metadata + scores)
        2026-05-22T14-30-16Z_IMG_1235.jpg
        2026-05-22T14-30-16Z_IMG_1235.json
    thumbnails/
      [session-id]/
        IMG_1234_thumb_small.jpg    (150px)
        IMG_1234_thumb_medium.jpg   (400px)
    sessions/
      [session-id].json             (ingest metadata, user info)
  logs/
    ingest.log
    analysis.log
```

#### 2.4 Metadata File Format (`.json` sidecar)

```json
{
  "sessionId": "2026-05-22-wedding-reception",
  "originalFilename": "IMG_1234.JPG",
  "storagePath": "images/2026-05-22-wedding-reception/2026-05-22T14-30-15Z_IMG_1234.jpg",
  "ingestedAt": "2026-05-22T14:30:15Z",
  "analysis": {
    "sharpness": {
      "score": 87,
      "threshold": 65,
      "passing": true
    },
    "subjects": [
      { "label": "person", "confidence": 0.98, "count": 2 },
      { "label": "flowers", "confidence": 0.76 },
      { "label": "backlighting", "confidence": 0.64 }
    ],
    "analyzedAt": "2026-05-22T14:30:25Z",
    "duration_ms": 1200
  },
  "exif": {
    "camera": "Canon EOS R5",
    "iso": 400,
    "shutter": "1/500",
    "aperture": "f/2.8",
    "focalLength": "85mm",
    "lensModel": "Canon RF 85mm f/1.2L"
  },
  "curation": {
    "rating": null,
    "decision": null,
    "cullReason": null,
    "decidedBy": null,
    "decidedAt": null
  },
  "tags": [
    "portrait",
    "couple"
  ]
}
```

---

### Component 3: PhotoCurator Web UI

#### 3.1 Core Pages

##### 3.1.1 Sessions Dashboard
```
PhotoCurator Web
┌─────────────────────────────────────────┐
│ Sessions                                 │
├─────────────────────────────────────────┤
│ [+ New Ingest]                          │
│                                         │
│ Session: 2026-05-22 Wedding             │
│ Status: ✓ Complete (247 images)         │
│ Sharpness: 89% passing                  │
│ Top subjects: person, flowers, rings    │
│ Culled: 125 ✓ 122 ✗ (3 pending)         │
│ [Open]                                  │
│                                         │
│ Session: 2026-05-21 Reception           │
│ Status: ✓ Complete (156 images)         │
│ Sharpness: 76% passing                  │
│ Top subjects: group, food, lights       │
│ Culled: 89 ✓ 67 ✗ (0 pending)           │
│ [Open]                                  │
└─────────────────────────────────────────┘
```

##### 3.1.2 Image Culling Interface
```
PhotoCurator Cull
┌─────────────────────────────────────────┐
│ Session: Wedding Reception (247 images) │
│ Culled: 125 ✓ 122 ✗ (3 pending)         │
├─────────────────────────────────────────┤
│                                         │
│           [← PREV]                      │
│                                         │
│      ┌─────────────────┐                │
│      │   [Image]       │                │
│      │    Main View    │                │
│      │   (high-res)    │                │
│      └─────────────────┘                │
│                                         │
│      Sharpness: ████░░ 87%              │
│      Subjects: person (0.98), flowers   │
│      Camera: Canon EOS R5, f/2.8, 1/500 │
│      Rating: ○○○○○ (click to rate)      │
│                                         │
│      [✓ Keep]      [✗ Reject]      [?]  │
│                                         │
│           [NEXT →]                      │
├─────────────────────────────────────────┤
│ Filmstrip (25% visible):                │
│ [○][○][●][○][○][○][○][○][○]              │
│ Cursor shows pending (3) in different   │
│ color; already decided (✓/✗) greyed out │
└─────────────────────────────────────────┘
```

##### 3.1.3 Filter & Browse
```
PhotoCurator Browse
┌─────────────────────────────────────────┐
│ Session: Wedding Reception              │
│ Filter:                                 │
│   Sharpness: [>=70]                     │
│   Subjects:  [person ✓][flowers ✓][..] │
│   Status:    [All ✓][Pending ✓][..] │
│   [Apply]                               │
├─────────────────────────────────────────┤
│ Grid view: 20 thumbnails                │
│ [○][○][✓][○][✓][✗][✗][○][○][.....]     │
│                                         │
│ Selected: 3 of 20 shown                 │
│ [Export Selected] [Bulk Reject] [Tag]   │
└─────────────────────────────────────────┘
```

#### 3.2 Key Features
- **Rapid Navigation**: Arrow keys, spacebar, number row shortcuts
- **Filmstrip**: Show 25% of session images; click to jump
- **Keyboard Shortcuts**:
  - `→` / `←` : Next/Previous
  - `K` : Keep / Rate up
  - `X` : Reject
  - `Z` : Undo last decision
  - `[1-5]` : Star rating
  - `T` : Add custom tag
  - `F` : Full-screen view

- **Bulk Operations**: Select multiple images via Shift-click, apply decisions
- **Filter by**:
  - Sharpness range (slider)
  - Subject detection (checkbox list)
  - Curation status (pending, approved, rejected)
  - Custom tags

- **Export Options**:
  - High-res JPEG with embedded metadata
  - Original file (RAW + JPEG)
  - Contact sheet / proof PDF
  - Metadata CSV

#### 3.3 Session Ingest URL
- Generate unique URL for each session: `https://curator.example.com/ingest/[session-id]`
- Display QR code in Ingest app; photographer can scan to confirm
- Shows real-time progress in browser

---

## Technical Architecture

### Technology Stack

#### Ingest App (macOS)
- **Language**: Swift
- **UI Framework**: SwiftUI
- **Image Handling**: AVFoundation, CoreImage
- **Networking**: URLSession
- **Deployment**: Xcode + code signing

#### Server
- **Language**: Python (FastAPI) or Go (fiber)
- **Database**: PostgreSQL (metadata), filesystem (image storage)
- **Image Processing**: OpenCV, PIL, numpy
- **ML Models**: YOLO v8 (embedded), or ort-runtime for ONNX
- **Task Queue**: Celery (Python) or similar for async processing
- **Caching**: Redis (optional, for session state)

#### Web UI
- **Framework**: React / Vue.js (desktop-first, responsive)
- **State Management**: Redux or Pinia
- **Build**: Vite
- **Styling**: TailwindCSS or similar
- **Hosting**: Nginx / Caddy reverse proxy

### Processing Pipeline

```
Ingest App                  Server                        Web UI
    │                           │                           │
    ├─ [Browse Source] ─────────┼───────────────────────────┤
    │                           │                           │
    ├─ [Select Images] ─────────┼───────────────────────────┤
    │                           │                           │
    ├─ [Upload] ───────────────>│                           │
    │           (parallel 4-8)  │                           │
    │                           ├─ Receive + Store          │
    │                           ├─ Queue for Analysis       │
    │                           ├─[Analyze: Sharpness]      │
    │                           ├─[Analyze: Subjects]       │
    │                           ├─[Extract Metadata]        │
    │                           ├─[Generate Thumbnails]     │
    │                           ├─[Write .json sidecar]     │
    │                           │                           │
    │        [Progress Feed] <──┤─ Webhook/WebSocket ─────>│
    │                           │   (real-time status)      │
    │                           │                           │
    │                           ├─ Mark Complete            │
    │                           │                           │
    │                  [Access Web UI] ──────────────────>│
    │                                                       │
    │                                  [Browse + Cull]     │
    │                                  [Apply Ratings]      │
    │                                  [Export]             │
```

### Deployment Model

#### Option A: Local Network (Recommended for v1)
- Ingest app uploads to server on home/office network
- Server runs on NAS, Mac mini, or dedicated Linux box
- Web UI accessed via `https://curator.local:8443`
- All data stays on-premises

#### Option B: Cloud (Future)
- Ingest app uploads to cloud service (AWS S3, etc.)
- Server runs on cloud VM
- Web UI accessible remotely via https://curator.example.com

---

## Implementation Roadmap

### Phase 1: MVP (4–6 weeks)
- [x] Project setup + repository structure
- [ ] **Ingest App v1**: Directory selection, local staging, basic progress UI
- [ ] **Server v1**: Receive images, basic metadata extraction
- [ ] **Sharpness Scoring**: Laplacian-based scoring, configurable threshold
- [ ] **Web UI v1**: Session list, basic image browser, Keep/Reject decisions
- [ ] **Export**: Export culled images with metadata

### Phase 2: Enhanced Analysis (2–3 weeks)
- [ ] **Subject Detection**: YOLO v8 integration, 15+ classes
- [ ] **Embedded Labels**: Store subjects as EXIF/sidecar tags
- [ ] **Thumbnail Generation**: Multiple resolutions for fast browsing
- [ ] **Filter UI**: Sharpness ranges, subject checkboxes, status filters

### Phase 3: Advanced Culling UX (2–3 weeks)
- [ ] **Keyboard Shortcuts**: Full keyboard-driven culling workflow
- [ ] **Filmstrip Navigation**: Visual timeline of all images
- [ ] **Bulk Operations**: Select multiple, apply decisions at once
- [ ] **Custom Tags**: User-defined tags per image
- [ ] **Real-time Progress**: WebSocket updates on ingest + analysis

### Phase 4: Polish & Scale (1–2 weeks)
- [ ] **Performance**: Handle 1,000+ images without degradation
- [ ] **Error Handling**: Retry logic, graceful degradation
- [ ] **Logging**: Structured logging for debugging
- [ ] **Documentation**: README, setup guide, API docs

---

## Non-Functional Requirements

### Performance
- Ingest: 2–3 MB/s on gigabit Ethernet
- Sharpness scoring: <2 seconds per image
- Subject detection: <1 second per image (GPU), <5 seconds (CPU)
- Web UI: Image load <500ms, cull action <100ms
- Thumbnail generation: Parallel, <1 minute for 100 images

### Scalability
- Support up to 10,000 images per session
- Support up to 50 concurrent ingest sessions
- Database query response time <500ms for 100k+ images

### Reliability
- Retry transient network errors
- Resume interrupted uploads
- Preserve curation decisions even if server restarts
- Backup image files automatically

### Security
- HTTPS for all web communication
- API token authentication for server access
- CORS restrictions on web UI
- No unencrypted storage of sensitive metadata

### Accessibility
- Keyboard-driven workflow for full culling process
- High-contrast UI for outdoor viewing
- Mobile/tablet responsive (landscape primary)

---

## Risks & Mitigations

| Risk | Mitigation |
|------|-----------|
| Sharpness scoring < 90% accuracy | Calibrate threshold per camera model; allow manual override |
| Subject detection misses rare scenarios | Start with 15 common classes; user can manually tag; improve iteratively |
| Server CPU bottleneck on analysis | Use GPU for inference; implement priority queue; allow batch processing schedule |
| Network instability on set | Resume interrupted uploads; cache locally; show clear retry UI |
| Metadata loss in RAW processing | Extract EXIF before pipeline; embed in sidecar `.json`; preserve original files |
| UI slow on large sessions | Implement pagination; lazy-load thumbnails; optimize React rendering |

---

## Naming Rationale: **PhotoCurator**

### Why "Curator"?
The name emphasizes the system's core value: **curating** a large photo shoot into a refined collection. Unlike generic terms like "Photo Manager" or "Photo Reviewer," "Curator" conveys:

1. **Expertise**: Curators make intentional, thoughtful selections
2. **Automation**: The system does preliminary curation (sharpness, subjects) so the user focuses on artistic decisions
3. **Flow**: Rhymes with the culling/selection process
4. **Professionalism**: Appeals to photographers who take their craft seriously

### Alternative Names Considered
- **PhotoCull** - More literal but sounds negative
- **PhotoWorks** - Too generic
- **PhotoFlow** - Doesn't capture the curation angle
- **ShutterSort** - Clever but domain-specific to cameras
- **SelectPhoto** - Descriptive but less memorable

---

## Success Definition

PhotoCurator will be considered successful when:

1. ✓ A user can ingest 100+ images from a memory card in <5 minutes
2. ✓ Sharpness filtering reduces manual review time by 30–50%
3. ✓ Subject detection captures >90% of relevant images (e.g., "group photos")
4. ✓ Web UI culling workflow averages <3 seconds per decision
5. ✓ A full wedding shoot (1000+ images) can be culled to final selection in <1 hour
6. ✓ All curation decisions and metadata are preserved for long-term archive

---

## Appendices

### A. Supported Image Formats
- JPEG / JPG
- RAW: Canon (CR3), Nikon (NEF), Sony (ARW), Fujifilm (RAF)
- PNG
- HEIC / HEIF
- TIFF

### B. Sharpness Scoring Algorithm
```
1. Convert image to grayscale
2. Apply Laplacian filter
3. Compute variance of Laplacian
4. Normalize to 0-100 scale per camera/lens model (calibration)
5. Apply user threshold
```

### C. Subject Detection Classes (v1 & v2 roadmap)

**v1 MVP (8 classes)**:
- Person / Face
- Group (multiple people)
- Food / Cake
- Flowers / Bouquet
- Decorations / Venue
- Landscape / Outdoor
- Architecture / Indoor
- Other

**v2+ Additions**:
- Hands / Gesture
- Rings / Jewelry
- Backlight / Artistic lighting
- Motion / Blur effect
- Pet / Animal
- Vehicle
- Product / Still life

### D. Sample Export Formats

**Format 1: JPEG with EXIF**
```
IMG_1234_CURATED.jpg
  - EXIF: camera, ISO, shutter, aperture, focal length
  - XMP: subjects, sharpness score, curation rating
```

**Format 2: Metadata CSV**
```
filename,sharpness,subjects,rating,decision,camera,iso,shutter,aperture
IMG_1234.jpg,87,"person,flowers,backlight",5,keep,Canon EOS R5,400,1/500,f/2.8
```

**Format 3: Contact Sheet PDF**
```
Cover: Session name, date, photographer
Pages: Thumbnails grid (20 per page) + metadata footer
```

---

## Version History

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-05-22 | Initial PRD: Ingest, Sharpness, Subjects, Web UI, MVP roadmap |

