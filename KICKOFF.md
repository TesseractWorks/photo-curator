# PhotoCurator - Project Kickoff Checklist

Complete PRD and architecture documentation created. Ready for development.

## ✅ Deliverables Complete

### Documentation (2,335 lines, 6 Markdown files)

- [x] **README.md** — Quick navigation guide for all documents
- [x] **SUMMARY.md** — 5-minute executive overview + FAQ
- [x] **PRD.md** — 20-page comprehensive product requirements
  - Executive summary, problem statement, goals
  - Use cases & workflows (wedding, product, event photography)
  - Complete functional requirements (ingest app, server, web UI)
  - Non-functional requirements (performance, security, reliability)
  - 4-phase implementation roadmap
  - Risk analysis & mitigation
  
- [x] **ARCHITECTURE.md** — Technical deep dive
  - System architecture diagram & data flows
  - PostgreSQL database schema (4 tables)
  - FastAPI endpoint specifications
  - Sharpness scoring algorithm (Laplacian variance)
  - YOLO v8 subject detection integration
  - Performance targets & monitoring
  
- [x] **IMPLEMENTATION.md** — Development roadmap
  - Complete project file structure (100+ files across 3 components)
  - Technology stack rationale
  - Key implementation decisions
  - Phase-by-phase task breakdown
  
- [x] **WIREFRAMES.md** — Visual design & UX
  - 5 detailed UI wireframes (ASCII art)
  - End-to-end user flow diagrams
  - Data flow visualizations
  - Keyboard shortcut reference
  - Performance timeline

**Total documentation**: 2,335 lines of detailed specs, architecture, and roadmap

---

## 📋 Project Structure

Location: `/home/hermes/src/github.com/TesseractWorks/photo-curator-prd/`

```
photo-curator-prd/
├── README.md              ← START HERE (navigation guide)
├── SUMMARY.md             ← 5-min executive overview
├── PRD.md                 ← Full product spec
├── ARCHITECTURE.md        ← Technical design
├── IMPLEMENTATION.md      ← Development roadmap
└── WIREFRAMES.md          ← UI/UX mockups
```

---

## 🎯 Project Name: PhotoCurator

### Why "Curator"?
- **Curator** emphasizes intelligent selection (not just storage)
- The system auto-curates; photographer decides
- Appeals to professional photographers
- More memorable than generic "Photo Manager" or "PhotoSort"

### What It Does
1. **Ingest** (macOS app) — One-click import from memory cards
2. **Analyze** (Server) — Automatic sharpness scoring + subject detection
3. **Cull** (Web UI) — Rapid selection workflow (<3 sec per image decision)

---

## 🏗️ Architecture: 3-Tier System

```
┌─────────────────┐      ┌──────────────┐      ┌──────────────┐
│  Ingest App     │      │    Server    │      │   Web UI     │
│  (Swift/macOS)  │◄────►│ (FastAPI/Py) │◄────►│ (React/TS)   │
└─────────────────┘      └──────────────┘      └──────────────┘
   • Directory select      • Receive images      • Browse images
   • Upload queue          • Sharpness score     • Rate/cull
   • Progress UI           • YOLO v8 detect      • Keyboard nav
                           • Thumbnails         • Export
```

---

## 🔑 Key Features

### Ingestion
- ✓ Select any directory (memory card, external drive, folder)
- ✓ Recursively find JPEG, RAW, PNG, HEIC, TIFF files
- ✓ Upload 4-8 in parallel, preserve all EXIF metadata
- ✓ Real-time progress UI with ETA

### Analysis (Automatic)
- ✓ **Sharpness**: Laplacian variance algorithm, 0-100 score
- ✓ **Subjects**: YOLO v8 with 15+ detection classes
- ✓ **Metadata**: Extract EXIF + embed subject tags
- ✓ **Thumbnails**: Generate multiple resolutions for fast browsing

### Culling (Workflow-Optimized)
- ✓ Rapid navigation: arrow keys, keyboard shortcuts
- ✓ Keep/Reject decisions in <3 seconds per image
- ✓ Filmstrip timeline for quick image jumping
- ✓ Bulk operations (select multiple, apply decision)
- ✓ Smart filtering (sharpness range, subject types, decision status)

### Export
- ✓ JPEG + embedded metadata
- ✓ Original files (RAW + JPEG pairs)
- ✓ Contact sheet PDF
- ✓ Metadata CSV

---

## 📊 Success Metrics

| Metric | Target | Note |
|--------|--------|------|
| **Ingest Speed** | 2–3 MB/s | Typical gigabit network |
| **Sharpness Accuracy** | >90% | vs. manual review |
| **Subject Detection Recall** | >95% | 15+ core classes |
| **Culling Speed** | <3 sec/image | Keyboard-driven |
| **Wedding Shoot Time** | <1 hour | 1000 images → final selection |
| **Metadata Preservation** | 100% | EXIF + custom tags intact |

---

## 🛠️ Tech Stack

| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Ingest** | Swift + SwiftUI | Native macOS, best file I/O |
| **Server** | FastAPI (Python) | Fast dev, ML ecosystem |
| **Web UI** | React + TypeScript | Type safety, reusable components |
| **Database** | PostgreSQL | ACID, JSON metadata, scales |
| **Image Proc** | OpenCV + PIL | Established, local, no cloud deps |
| **ML** | YOLO v8 (ONNX) | Embedded, <1 sec inference |
| **Queue** | Celery + Redis | Distributed tasks, retries |
| **Deploy** | Docker + Compose | Reproducible, multi-service |

---

## 📅 Development Roadmap

### Phase 1: MVP (4–6 weeks) ⭐
**Goal**: Basic ingest → sharpness score → web cull

- Ingest app: directory selection, upload progress
- Server: receive images, store, extract EXIF
- Sharpness scoring: Laplacian algorithm
- Web UI: session list, image browser, Keep/Reject buttons
- Export: JPEG + metadata

**Deliverable**: Photographer can shoot, ingest, cull, export

---

### Phase 2: Enhanced Analysis (2–3 weeks)
**Goal**: Subject detection + smart filtering

- YOLO v8 integration (15+ detection classes)
- Embedded labels in EXIF + sidecar JSON
- Thumbnail generation (3 sizes)
- Filter UI (sharpness ranges, subjects, status)

**Deliverable**: Smart filtering reduces manual review time by 30%

---

### Phase 3: Advanced UX (2–3 weeks)
**Goal**: Professional culling workflow

- Keyboard shortcuts (full keyboard-driven workflow)
- Filmstrip navigation (visual timeline)
- Bulk operations (select multiple, apply decision)
- Custom tagging per image
- WebSocket real-time progress

**Deliverable**: Culling workflow <3 sec per image

---

### Phase 4: Scale & Polish (1–2 weeks)
**Goal**: Production-ready

- Performance tuning (handle 1000+ images smoothly)
- Error handling & retries
- Structured logging
- Documentation & deployment guide
- Monitoring & alerting

**Deliverable**: Ready for production deployment

---

## 📈 Expected Timeline

| Phase | Duration | Cumulative |
|-------|----------|-----------|
| Phase 1 (MVP) | 4–6 weeks | 4–6 weeks |
| Phase 2 (Analysis) | 2–3 weeks | 6–9 weeks |
| Phase 3 (UX) | 2–3 weeks | 8–12 weeks |
| Phase 4 (Polish) | 1–2 weeks | 9–14 weeks |

**Total**: 10–14 weeks from kickoff to full feature set

---

## 🚀 Next Steps (Action Items)

### Immediate (This Week)
- [ ] Create GitHub repository: `github.com/TesseractWorks/photo-curator`
- [ ] Initialize monorepo structure (3 subprojects)
- [ ] Set up CI/CD pipelines (.github/workflows)
- [ ] Create development environment guide (DEVELOPMENT.md)

### Phase 1 Kickoff (Next Week)
- [ ] Ingest app: Create Swift package structure
  - [ ] Build directory selector UI (SwiftUI)
  - [ ] Implement image scanner (find files recursively)
  - [ ] Build upload manager (parallel chunks)
  
- [ ] Server: Create FastAPI skeleton
  - [ ] Set up database (PostgreSQL + migrations)
  - [ ] Build ingest API endpoints
  - [ ] Implement image storage service
  
- [ ] Web UI: Create React project structure
  - [ ] Set up Vite + TypeScript config
  - [ ] Build session dashboard page
  - [ ] Build image viewer component

### Communication
- [ ] Share PRD with team for review & feedback
- [ ] Schedule kickoff meeting to discuss risks & unknowns
- [ ] Establish coding standards & PR review process
- [ ] Set up project management (GitHub Projects or similar)

---

## ❓ FAQ for Team

### Q: Why build this vs. using Lightroom?
**A**: Lightroom is post-processing. PhotoCurator specializes in **rapid ingest + culling** (saves 2-3 hours on a wedding shoot).

### Q: How accurate is sharpness scoring?
**A**: Laplacian variance achieves ~90% accuracy with camera-specific calibration. User can adjust threshold or manually override.

### Q: What about RAW processing?
**A**: We preserve RAW originals + generate JPEG proxies. Photographer can post-process in Lightroom after culling.

### Q: Can we support video?
**A**: Phase 2+. Start with photos only (simpler scope).

### Q: Will this work offline?
**A**: Server must be available during ingest/analysis. Local network only in Phase 1. Cloud upload is Phase 2+.

### Q: How do we handle failure/retry?
**A**: Celery + Redis handle task retries. Upload can be resumed. Comprehensive error logging for debugging.

---

## 📚 Documentation Locations

| Document | Purpose | Audience |
|----------|---------|----------|
| **README.md** (in this dir) | Navigation + quick start | Everyone |
| **SUMMARY.md** | 5-minute overview | Stakeholders |
| **PRD.md** | Full specification | Product, Design |
| **ARCHITECTURE.md** | System design | Architecture, Backend |
| **IMPLEMENTATION.md** | Development roadmap | Engineers |
| **WIREFRAMES.md** | UI/UX mockups | Design, Frontend |
| **DEVELOPMENT.md** (TBD) | Setup & contributing | All engineers |
| **API.md** (TBD) | API reference | Backend, Frontend |
| **DEPLOYMENT.md** (TBD) | Production setup | DevOps, Backend |

---

## 🎓 Key Decisions & Rationale

### Why Swift for macOS ingest?
- ✓ Native SwiftUI for clean UI
- ✓ Best file system APIs
- ✓ Direct camera SDK access
- ✓ Single-app distribution

### Why FastAPI?
- ✓ Rapid development (automatic OpenAPI docs)
- ✓ Excellent async support
- ✓ Strong ML ecosystem (OpenCV, YOLO, Celery)
- ✓ Type hints for safety

### Why local-first (no cloud in v1)?
- ✓ Simpler MVP scope
- ✓ Faster development & testing
- ✓ Better privacy for photographers
- ✓ Cloud can be added later (S3 backend)

### Why embedded YOLO v8 (not API)?
- ✓ No external dependencies
- ✓ Photographer data stays local
- ✓ <1 sec inference (vs. API latency)
- ✓ Cheaper at scale

---

## ✨ Why This Project Will Be Great

1. **Solves Real Problem**: Photographers waste 30-40% of time on mechanical tasks
2. **Clear Scope**: Well-defined 3-component system, 4-phase roadmap
3. **Modern Tech**: Swift, FastAPI, React, YOLO — industry-standard tools
4. **Professional UX**: Keyboard-driven workflow, real-time feedback, smart filtering
5. **Extensible**: Easy to add new detection classes, export formats, cloud backends
6. **Testable**: Clear APIs, deterministic algorithms (Laplacian, YOLO)

---

## 🎬 Ready to Start?

The PRD is complete and comprehensive. All technical decisions have been made and documented. Architecture is solid and scalable.

**Next**: Create GitHub repository and begin Phase 1 development.

---

**Created**: 2026-05-22  
**Status**: ✅ PRD Complete | 🚀 Ready for Development  
**Questions?** Refer to SUMMARY.md (5 min) or PRD.md (full spec)

