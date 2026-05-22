# PhotoCurator - Implementation Guide

This document provides the technical implementation roadmap and architecture details for PhotoCurator.

## Project Structure

```
photo-curator/
├── README.md                          # Project overview and quick start
├── PRD.md                             # Full product requirements (this dir)
├── ARCHITECTURE.md                    # Technical architecture deep dive
├── DEVELOPMENT.md                     # Development setup and contribution guide
│
├── ingest-app/                        # macOS app (Swift/SwiftUI)
│   ├── Package.swift
│   ├── Package.resolved
│   ├── Sources/
│   │   └── PhotoCuratorIngest/
│   │       ├── App.swift              # Main app entry
│   │       ├── ContentView.swift      # Main UI
│   │       ├── Networking/
│   │       │   ├── CuratorServerClient.swift
│   │       │   └── UploadManager.swift
│   │       ├── Models/
│   │       │   ├── Image.swift
│   │       │   ├── Session.swift
│   │       │   └── UploadProgress.swift
│   │       ├── Services/
│   │       │   ├── ImageScanner.swift
│   │       │   ├── StagingManager.swift
│   │       │   └── MetadataExtractor.swift
│   │       └── Views/
│   │           ├── DirectorySelector.swift
│   │           ├── ProgressView.swift
│   │           └── CompletionView.swift
│   ├── Tests/
│   ├── .gitignore
│   └── build.sh                       # Build and codesign script
│
├── server/                            # Python/FastAPI backend
│   ├── requirements.txt
│   ├── setup.py
│   ├── docker-compose.yml             # PostgreSQL, Redis, Nginx
│   ├── Dockerfile                     # Server container
│   ├── .env.example                   # Environment config template
│   │
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI app entry
│   │   ├── config.py                  # Configuration
│   │   ├── database.py                # PostgreSQL setup
│   │   │
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── ingest.py              # POST /api/ingest routes
│   │   │   ├── images.py              # GET /api/images routes
│   │   │   ├── sessions.py            # GET /api/sessions routes
│   │   │   └── export.py              # GET /api/export routes
│   │   │
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── image.py               # SQLAlchemy Image model
│   │   │   ├── session.py             # SQLAlchemy Session model
│   │   │   └── analysis.py            # SQLAlchemy Analysis model
│   │   │
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── image_storage.py       # Store/retrieve images
│   │   │   ├── sharpness_scorer.py    # Laplacian-based scoring
│   │   │   ├── subject_detector.py    # YOLO v8 inference
│   │   │   ├── metadata_extractor.py  # EXIF extraction
│   │   │   ├── thumbnail_generator.py # Resize images
│   │   │   └── analysis_queue.py      # Task queueing
│   │   │
│   │   ├── jobs/
│   │   │   ├── __init__.py
│   │   │   ├── analyze_image.py       # Celery task
│   │   │   ├── generate_thumbnails.py
│   │   │   └── export_session.py
│   │   │
│   │   └── utils/
│   │       ├── __init__.py
│   │       ├── logging.py
│   │       ├── constants.py
│   │       └── validators.py
│   │
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py
│   │   ├── test_ingest_api.py
│   │   ├── test_sharpness_scorer.py
│   │   ├── test_subject_detector.py
│   │   └── test_export.py
│   │
│   ├── migrations/                    # Alembic database migrations
│   │   ├── alembic.ini
│   │   └── versions/
│   │
│   └── scripts/
│       ├── init_db.py                 # Database initialization
│       ├── download_models.py         # Download YOLO weights
│       └── benchmarks.py              # Performance testing
│
├── web-ui/                            # React frontend
│   ├── package.json
│   ├── vite.config.ts                 # Vite bundler config
│   ├── index.html
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── .env.example
│   │
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx                    # Router setup
│   │   │
│   │   ├── pages/
│   │   │   ├── SessionsList.tsx       # Dashboard
│   │   │   ├── CullInterface.tsx      # Main culling page
│   │   │   ├── BrowseView.tsx         # Grid/filter view
│   │   │   ├── IngestProgress.tsx     # Real-time ingest status
│   │   │   └── ExportDialog.tsx       # Export options
│   │   │
│   │   ├── components/
│   │   │   ├── ImageViewer.tsx        # Main image display
│   │   │   ├── Filmstrip.tsx          # Timeline navigation
│   │   │   ├── SubjectBadge.tsx       # Subject label display
│   │   │   ├── SharpnessBar.tsx       # Sharpness visualization
│   │   │   ├── ControlPanel.tsx       # Keep/Reject buttons
│   │   │   └── KeyboardShortcuts.tsx  # Help overlay
│   │   │
│   │   ├── hooks/
│   │   │   ├── useSession.ts          # Session data fetching
│   │   │   ├── useImages.ts           # Image list + filtering
│   │   │   ├── useCulling.ts          # Cull decision state
│   │   │   ├── useKeyboard.ts         # Keyboard event binding
│   │   │   └── useWebSocket.ts        # Real-time progress
│   │   │
│   │   ├── api/
│   │   │   ├── client.ts              # API client (axios/fetch)
│   │   │   ├── sessions.ts            # Session endpoints
│   │   │   ├── images.ts              # Image endpoints
│   │   │   └── export.ts              # Export endpoints
│   │   │
│   │   ├── types/
│   │   │   ├── api.ts                 # API response types
│   │   │   ├── models.ts              # Domain models
│   │   │   └── ui.ts                  # UI state types
│   │   │
│   │   ├── store/
│   │   │   ├── sessionSlice.ts        # Redux/Pinia session state
│   │   │   ├── cullingSlice.ts        # Culling decisions state
│   │   │   └── index.ts               # Store setup
│   │   │
│   │   ├── styles/
│   │   │   ├── index.css              # Global styles
│   │   │   └── tailwind.css           # Tailwind imports
│   │   │
│   │   └── utils/
│   │       ├── formatters.ts
│   │       ├── keyboard.ts            # Keyboard mapping
│   │       └── validators.ts
│   │
│   ├── public/
│   │   └── icons/
│   │
│   └── tests/
│       ├── unit/
│       ├── integration/
│       └── e2e/
│
├── docs/                              # Additional documentation
│   ├── DEPLOYMENT.md                  # Deployment guide (Docker, K8s)
│   ├── API.md                         # API reference
│   ├── SHARPNESS_CALIBRATION.md       # Camera-specific tuning
│   ├── YOLO_INTEGRATION.md            # Subject detection setup
│   ├── DB_SCHEMA.md                   # Database schema
│   └── TROUBLESHOOTING.md
│
├── .github/
│   ├── workflows/
│   │   ├── test-server.yml            # Server CI/CD
│   │   ├── test-web-ui.yml            # Web UI CI/CD
│   │   ├── build-ingest.yml           # macOS app CI/CD
│   │   └── deploy.yml                 # Production deployment
│   │
│   └── ISSUE_TEMPLATE/
│       └── bug_report.md
│
├── .gitignore
├── LICENSE
└── CHANGELOG.md
```

## Key Implementation Decisions

### 1. Technology Choices

| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Ingest App** | Swift + SwiftUI | Native macOS integration, best file/image handling, fast performance |
| **Server** | FastAPI + Python | Rapid development, excellent ML ecosystem, async/concurrency built-in |
| **Web UI** | React + TypeScript | Type safety, component reusability, strong community |
| **Database** | PostgreSQL | ACID transactions, good JSON support for metadata, scalability |
| **Image Processing** | OpenCV + PIL | Well-established, no cloud dependencies, fast |
| **ML Models** | YOLO v8 (ONNX) | Embedded execution, no external API calls, <1s per image |
| **Task Queue** | Celery + Redis | Distributed processing, retries, priority queues |
| **Containerization** | Docker + Compose | Reproducible deployment, multi-service orchestration |

### 2. Processing Architecture

**Synchronous for Ingest:**
- Upload endpoint blocks until image is stored
- Ensures user gets immediate confirmation
- Keeps ingest app responsive

**Asynchronous for Analysis:**
- Queue analysis tasks in Celery
- Ingest app polls `GET /api/status/{session_id}` for progress
- Allows batching and prioritization

### 3. Storage Strategy

**Images:**
- Store in organized filesystem: `data/images/{session_id}/{timestamp}-{filename}`
- Symlink originals to avoid duplication
- Generate thumbnails in separate tree

**Metadata:**
- `.json` sidecar for each image (sharpness, subjects, EXIF)
- PostgreSQL for queryable metadata + curation decisions
- Redis cache for session state during culling

### 4. Frontend Architecture

**State Management:**
- Redux for global state (sessions, images, analysis results)
- Local React state for UI interactions (selected image, keyboard focus)
- WebSocket for real-time progress updates

**Keyboard-First Design:**
- All major actions accessible via keyboard
- Full filmstrip navigation
- Customizable hotkeys

## Next Steps

1. Create GitHub repository: `github.com/TesseractWorks/photo-curator`
2. Set up monorepo structure with subproject CI/CD
3. Begin Phase 1 development (see PRD roadmap)
4. Establish coding standards and PR review process

---

*For questions or corrections, file an issue in the GitHub repo.*

