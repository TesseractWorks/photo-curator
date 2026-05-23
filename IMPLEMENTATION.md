# PhotoCurator v2 - Implementation Guide

This document provides the technical implementation roadmap and architecture details for PhotoCurator v2.

## Project Structure

```
photo-curator/
├── README.md                          # Project overview and quick start
├── PRD.md                             # Full product requirements
├── ARCHITECTURE.md                    # Technical architecture deep dive
│
├── ingest-cli/                        # macOS CLI (Go)
│   ├── main.go
│   ├── go.mod
│   ├── go.sum
│   ├── Makefile
│   │
│   ├── cmd/
│   │   └── photo-curator/
│   │       └── main.go                # CLI entry point
│   │
│   ├── internal/
│   │   ├── cli/
│   │   │   ├── commands.go            # Command definitions
│   │   │   ├── ingest.go              # Ingest command
│   │   │   └── flags.go               # Flag parsing
│   │   │
│   │   ├── client/
│   │   │   ├── curator.go             # HTTP client to server
│   │   │   └── session.go             # Session management
│   │   │
│   │   ├── fs/
│   │   │   ├── scanner.go             # Directory scanner
│   │   │   ├── formats.go             # Image format detection
│   │   │   └── metadata.go            # EXIF extraction
│   │   │
│   │   ├── upload/
│   │   │   ├── manager.go             # Upload orchestration
│   │   │   ├── batch.go               # Batch processing
│   │   │   ├── progress.go            # Progress tracking
│   │   │   └── retry.go               # Retry logic
│   │   │
│   │   └── ui/
│   │       ├── progress.go            # Progress bar rendering
│   │       ├── colors.go              # Terminal colors
│   │       └── formatting.go          # Text formatting
│   │
│   ├── tests/
│   │   ├── cli_test.go
│   │   ├── scanner_test.go
│   │   └── upload_test.go
│   │
│   └── build.sh                       # Build and sign script
│
├── server/                            # Go backend (Gin)
│   ├── main.go
│   ├── go.mod
│   ├── go.sum
│   ├── Dockerfile
│   ├── docker-compose.yml             # PostgreSQL, volumes
│   ├── .env.example                   # Environment config template
│   │
│   ├── cmd/
│   │   └── server/
│   │       └── main.go                # Server entry
│   │
│   ├── internal/
│   │   ├── config/
│   │   │   └── config.go              # Configuration loading
│   │   │
│   │   ├── db/
│   │   │   ├── db.go                  # Database initialization
│   │   │   ├── migration.go           # Schema setup
│   │   │   └── queries.go             # SQL helpers
│   │   │
│   │   ├── models/
│   │   │   ├── session.go             # Session model
│   │   │   ├── image.go               # Image model
│   │   │   ├── analysis.go            # Analysis job model
│   │   │   └── taxonomy.go            # Taxonomy model
│   │   │
│   │   ├── api/
│   │   │   ├── router.go              # Route setup
│   │   │   ├── middleware.go          # Auth, logging, CORS
│   │   │   │
│   │   │   ├── handlers/
│   │   │   │   ├── sessions.go        # Session endpoints
│   │   │   │   ├── upload.go          # Upload endpoints
│   │   │   │   ├── images.go          # Image endpoints
│   │   │   │   └── export.go          # Export endpoints
│   │   │   │
│   │   │   └── responses.go           # Response helpers
│   │   │
│   │   ├── services/
│   │   │   ├── session_service.go     # Session business logic
│   │   │   ├── storage_service.go     # Store/retrieve images
│   │   │   ├── upload_service.go      # Receive uploads
│   │   │   ├── analysis_service.go    # Queue/manage analysis
│   │   │   └── export_service.go      # Export job management
│   │   │
│   │   ├── processor/
│   │   │   ├── processor.go           # Processing orchestrator
│   │   │   ├── sharpness.go           # Laplacian scoring
│   │   │   ├── taxonomy.go            # Hierarchical classification
│   │   │   ├── thumbnail.go           # Image resizing
│   │   │   └── metadata.go            # EXIF embedding
│   │   │
│   │   ├── jobs/
│   │   │   ├── queue.go               # Job queue (channel-based)
│   │   │   ├── worker.go              # Worker pool
│   │   │   └── scheduler.go           # Task scheduling
│   │   │
│   │   ├── storage/
│   │   │   ├── filesystem.go          # Local FS backend
│   │   │   └── s3.go                  # (Optional) S3 backend
│   │   │
│   │   └── utils/
│   │       ├── logger.go              # Logging
│   │       ├── errors.go              # Error handling
│   │       └── constants.go           # Constants
│   │
│   ├── tests/
│   │   ├── handlers_test.go
│   │   ├── services_test.go
│   │   ├── processor_test.go
│   │   └── db_test.go
│   │
│   └── scripts/
│       ├── init_db.sql                # Database initialization
│       └── seed_taxonomies.go         # Load default taxonomies
│
├── web-ui/                            # Next.js frontend
│   ├── package.json
│   ├── package-lock.json
│   ├── next.config.js                 # Next.js config
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── .env.example
│   │
│   ├── public/
│   │   ├── icons/
│   │   └── logos/
│   │
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx              # Root layout
│   │   │   ├── page.tsx                # Home/redirect
│   │   │   ├─── sessions/
│   │   │   │    └── page.tsx           # Sessions dashboard
│   │   │   └─── session/
│   │   │        ├── [id]/
│   │   │        │  ├── page.tsx        # Session details
│   │   │        │  └── cull/
│   │   │        │     └── page.tsx     # Culling interface
│   │   │        └── layout.tsx
│   │   │
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── Header.tsx
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   └── Footer.tsx
│   │   │   │
│   │   │   ├── sessions/
│   │   │   │   ├── SessionsList.tsx
│   │   │   │   ├── SessionCard.tsx
│   │   │   │   └── SessionStats.tsx
│   │   │   │
│   │   │   ├── culling/
│   │   │   │   ├── ImageViewer.tsx     # Main display
│   │   │   │   ├── Filmstrip.tsx       # Thumbnail bar
│   │   │   │   ├── ControlPanel.tsx    # Keep/Reject buttons
│   │   │   │   ├── TaxonomyDisplay.tsx # Category breadcrumb
│   │   │   │   ├── SharpnessBar.tsx    # Score visualization
│   │   │   │   ├── MetadataPanel.tsx   # EXIF details
│   │   │   │   └── KeyboardHelp.tsx    # Hotkey overlay
│   │   │   │
│   │   │   ├── common/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   ├── Spinner.tsx
│   │   │   │   ├── Toast.tsx
│   │   │   │   └── Badge.tsx
│   │   │   │
│   │   │   └── forms/
│   │   │       ├── SessionForm.tsx
│   │   │       └── ExportForm.tsx
│   │   │
│   │   ├── hooks/
│   │   │   ├── useSession.ts
│   │   │   ├── useImages.ts
│   │   │   ├── useCulling.ts
│   │   │   ├── useKeyboard.ts
│   │   │   ├── useApi.ts
│   │   │   └── useLocalStorage.ts
│   │   │
│   │   ├── lib/
│   │   │   ├── api.ts                  # API client (fetch-based)
│   │   │   ├── api/
│   │   │   │   ├── sessions.ts
│   │   │   │   ├── images.ts
│   │   │   │   └── export.ts
│   │   │   ├── utils.ts
│   │   │   └── constants.ts
│   │   │
│   │   ├── types/
│   │   │   ├── api.ts                  # API response types
│   │   │   ├── models.ts               # Domain models
│   │   │   └── ui.ts                   # UI state
│   │   │
│   │   ├── store/
│   │   │   ├── store.ts                # Zustand store
│   │   │   ├── sessionSlice.ts
│   │   │   └── cullingSlice.ts
│   │   │
│   │   ├── styles/
│   │   │   ├── globals.css
│   │   │   └── tailwind.css
│   │   │
│   │   └── utils/
│   │       ├── formatters.ts           # Date, size formatting
│   │       ├── keyboard.ts             # Keyboard events
│   │       └── validators.ts
│   │
│   ├── tests/
│   │   ├── __tests__/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   └── lib/
│   │   │
│   │   └── setup.ts
│   │
│   └── .eslintrc.js
│
├── docs/                              # Additional documentation
│   ├── API.md                         # Full API reference
│   ├── DEPLOYMENT.md                  # Docker, production setup
│   ├── SHARPNESS_CALIBRATION.md       # Camera-specific tuning
│   ├── TAXONOMY_CONFIG.md             # Taxonomy customization
│   ├── DB_SCHEMA.md                   # Database schema details
│   └── TROUBLESHOOTING.md
│
├── .github/
│   ├── workflows/
│   │   ├── test-server.yml            # Go server CI/CD
│   │   ├── test-web-ui.yml            # Next.js CI/CD
│   │   ├── build-cli.yml              # macOS CLI build
│   │   └── deploy.yml                 # Production deployment
│   │
│   └── ISSUE_TEMPLATE/
│       └── bug_report.md
│
├── .gitignore
├── LICENSE
├── CHANGELOG.md
└── CONTRIBUTING.md
```

---

## Key Implementation Decisions

### 1. Technology Choices

| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Ingest CLI** | Go (standalone binary) | Single executable, cross-platform, minimal dependencies, fast startup |
| **Server API** | Go (Gin/Echo framework) | Lightweight, fast, native HTTP/2, excellent concurrency, easy deployment |
| **Web UI** | Next.js + TypeScript | Modern React, SSR capable, strong ecosystem, type safety |
| **Database** | SQLite (dev) / PostgreSQL (prod) | ACID transactions, JSON support for metadata, scalability |
| **Image Processing** | Go imaging libraries | No external dependencies, fast, embedded in server |
| **Taxonomy Classification** | TensorFlow Lite or ONNX Runtime | Pre-trained hierarchical models, lightweight, no cloud API |
| **Task Queue** | Go goroutines + channels | Built-in concurrency, no external dependencies in dev, simple scaling |
| **Deployment** | Docker Compose | Reproducible, local development, production-ready |

### 2. Processing Architecture

**Synchronous for Upload:**
- Upload endpoint stores image immediately
- Returns HTTP 200 to CLI (quick feedback)
- Enqueues analysis tasks

**Asynchronous for Analysis:**
- Analysis tasks run in goroutine pool
- CLI polls `/api/sessions/{id}` for progress
- Server broadcasts updates via progress endpoint

### 3. Storage Strategy

**Images:**
- Store in organized filesystem: `data/images/{session_id}/{timestamp}-{filename}`
- Generate thumbnails in separate tree: `data/thumbnails/`
- Use hard links or copies (configurable)

**Metadata:**
- `.json` sidecar for each image (analysis results)
- PostgreSQL for queryable metadata + curation decisions
- In-memory cache for active session state

### 4. Frontend Architecture

**State Management:**
- Zustand for global state (sessions, images, analysis results)
- React local state for UI interactions
- localStorage for user preferences

**Keyboard-First Design:**
- All major actions accessible via keyboard shortcuts
- Configurable hotkey bindings
- Vim-style navigation (hjkl) optional

### 5. CLI Philosophy

The ingest CLI prioritizes:
- **Simplicity**: Single command, minimal flags
- **Progress visibility**: Real-time feedback without verbose output
- **Robustness**: Automatic retries, graceful error handling
- **Offline awareness**: Can detect network issues and queue for retry

---

## Development Setup

### Prerequisites

- Go 1.21+
- Node.js 18+ (Next.js)
- PostgreSQL 14+ (or SQLite for dev)
- macOS 11+ (for CLI distribution)

### Server Setup

```bash
# Clone
git clone https://github.com/TesseractWorks/photo-curator.git
cd photo-curator/server

# Setup database
go run ./scripts/init_db.go

# Run
go run ./cmd/server/main.go
```

### Web UI Setup

```bash
cd photo-curator/web-ui

# Install dependencies
npm install

# Development server
npm run dev

# Production build
npm run build
npm run start
```

### CLI Setup

```bash
cd photo-curator/ingest-cli

# Build
go build -o photo-curator ./cmd/photo-curator/main.go

# Test
./photo-curator ingest --help
```

---

## Deployment

### Docker Compose (Development & Production)

```bash
cd photo-curator/server

# Create .env from example
cp .env.example .env

# Start services
docker-compose up -d

# Run migrations
docker-compose exec curator-server go run ./scripts/init_db.go
```

### Environment Variables

```env
# Server
SERVER_HOST=0.0.0.0
SERVER_PORT=8080
DATABASE_URL=postgresql://user:pass@localhost/curator

# Storage
STORAGE_PATH=./data/images
THUMBNAIL_PATH=./data/thumbnails

# Security
JWT_SECRET=your-secret-here
API_TOKEN=your-api-token

# Processing
MAX_WORKERS=4
BATCH_SIZE=8

# Taxonomy
DEFAULT_TAXONOMY_ID=default-2026
```

---

## Performance Targets

| Operation | Target | Notes |
|-----------|--------|-------|
| Sharpness analysis | <100ms | Per image |
| Taxonomy classification | <200ms | Per image |
| Upload batch (8 files) | <5s | Network + storage |
| Web UI page load | <2s | Dashboard with 100+ sessions |
| Culling speed | 5-10 images/min | Keyboard-driven |
| Export (100 images) | <10s | ZIP creation + compression |

---

## Testing Strategy

### Server Tests
- Unit: Services, processors, models
- Integration: API endpoints with mock DB
- E2E: Full workflow (upload → analysis → cull → export)

### Web UI Tests
- Component: React component rendering
- Hook: Custom React hooks
- E2E: Playwright for full workflows

### CLI Tests
- Unit: Scanner, metadata extraction
- Integration: Mock server responses
- Manual: Real server integration

---

## Next Steps

1. ✅ Architecture & Implementation planning
2. → Phase 1: Server + CLI (ingest, analysis, basic API)
3. → Phase 2: Web UI (dashboard, culling interface)
4. → Phase 3: Export, taxonomy customization, multi-user support
5. → Phase 4: Performance optimization, cloud deployment

---

*Implementation guide for PhotoCurator v2. See ARCHITECTURE.md for detailed system design.*
