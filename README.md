# PhotoCurator - Product Documentation

This directory contains the complete product requirements, architecture, and implementation planning for **PhotoCurator**, a professional photo ingestion, evaluation, and culling system.

## Quick Navigation

### For Decision Makers & Product Stakeholders
- **Start here**: [SUMMARY.md](SUMMARY.md) — 2-minute executive overview
- **Full spec**: [PRD.md](PRD.md) — Complete product requirements with use cases, features, and roadmap

### For Architects & Tech Leads
- **Architecture**: [ARCHITECTURE.md](ARCHITECTURE.md) — System design, data flow, database schema, API spec
- **Implementation**: [IMPLEMENTATION.md](IMPLEMENTATION.md) — Project structure, tech stack decisions, phase breakdown

### For Developers
- **Project setup**: `DEVELOPMENT.md` (coming soon)
- **API reference**: `docs/API.md` (coming soon)
- **Deployment**: `docs/DEPLOYMENT.md` (coming soon)

---

## What is PhotoCurator?

**PhotoCurator** is an end-to-end photo workflow solution for photographers:

1. **Ingest** (macOS App) - Import photos from memory cards with one click
2. **Analyze** (Server) - Automatically score sharpness and detect subjects
3. **Cull** (Web UI) - Rapidly review and select best images with keyboard-driven workflow

**Key insight**: Photographers spend 30-40% of time on mechanical tasks (transfer, organize, filter). PhotoCurator automates these, freeing photographers to focus on artistic decisions.

---

## Why This Project?

### The Problem
- Professional photographers shoot 1000+ images per event
- Manual transfer from cards is tedious and error-prone
- No built-in quality filtering (which images are sharp?)
- No intelligent subject tagging (what's in each image?)
- Culling workflow takes hours with existing tools

### The Solution
- **1-click ingestion** from any directory
- **Automatic sharpness scoring** removes obviously blurry images
- **Subject detection** (people, flowers, rings, food, venue, etc.) with embedded labels
- **Web-based culling** optimized for photographer workflow: <3 sec per image decision
- **Metadata preserved** through entire pipeline (EXIF, custom tags, ratings)

### Success Metrics
- Ingest 100+ images in <5 minutes ✓
- Sharpness accuracy >90% ✓
- Subject detection >95% recall ✓
- Cull workflow <3 seconds per image ✓
- Wedding shoot (1000+ images) → final selection in <1 hour ✓

---

## Document Overview

### [SUMMARY.md](SUMMARY.md)
**Purpose**: High-level project overview for stakeholders  
**Length**: ~5 minutes  
**Contents**:
- Project name and elevator pitch
- Core features (ingestion, analysis, culling)
- Key differentiators vs. existing tools
- 3-tier architecture diagram
- Timeline and success metrics
- Document index

### [PRD.md](PRD.md) ⭐ **START HERE**
**Purpose**: Complete product specification  
**Length**: ~20 minutes  
**Contents**:
- Executive summary
- Problem statement
- Goals & success criteria
- Market analysis & use cases
- Functional requirements (detailed):
  - Ingest app: UI, config, error handling
  - Server: pipeline, storage schema, metadata format
  - Web UI: pages, features, shortcuts, filters
- Technical architecture overview
- Implementation roadmap (4 phases)
- Non-functional requirements (performance, security, etc.)
- Risk analysis
- Success definition

### [ARCHITECTURE.md](ARCHITECTURE.md)
**Purpose**: Deep technical design for architects  
**Length**: ~30 minutes  
**Contents**:
- System architecture diagram
- Data flow diagrams (3 flows: ingest, analysis, culling)
- Database schema (PostgreSQL)
- API endpoints (FastAPI specification)
- Sharpness scoring algorithm (Laplacian variance)
- Subject detection (YOLO v8 integration)
- Performance targets and tuning
- Security considerations
- Monitoring & observability

### [IMPLEMENTATION.md](IMPLEMENTATION.md)
**Purpose**: Roadmap and project structure for developers  
**Length**: ~15 minutes  
**Contents**:
- Complete project file structure (3 components)
- Key implementation decisions (tech stack, architecture patterns)
- Phase-by-phase breakdown (MVP → full feature)
- Getting started guide

---

## Technology Stack (Quick Reference)

| Component | Technology | Why |
|-----------|-----------|-----|
| **Ingest App** | Swift + SwiftUI (macOS) | Native integration, best image handling |
| **Server** | FastAPI (Python) | Rapid dev, ML ecosystem, async built-in |
| **Web UI** | React + TypeScript | Type safety, component reuse, community |
| **Database** | PostgreSQL | ACID, JSON support, scalability |
| **Image Processing** | OpenCV + PIL | Established, no cloud deps, fast |
| **ML Models** | YOLO v8 (ONNX) | Embedded, local, <1s inference |
| **Task Queue** | Celery + Redis | Distributed, retries, priority |
| **Deployment** | Docker + Compose | Reproducible, multi-service |

---

## Development Phases

### Phase 1: MVP (4–6 weeks)
✓ Basic ingest app  
✓ Server receive & store  
✓ Sharpness scoring  
✓ Simple web UI + Keep/Reject

### Phase 2: Analysis (2–3 weeks)
✓ YOLO v8 integration  
✓ Subject detection & embedded labels  
✓ Thumbnail generation  
✓ Advanced filtering

### Phase 3: UX (2–3 weeks)
✓ Keyboard shortcuts  
✓ Filmstrip navigation  
✓ Bulk operations  
✓ Real-time progress

### Phase 4: Scale & Polish (1–2 weeks)
✓ Performance optimization  
✓ Error handling & logging  
✓ Documentation  

**Total: 10–14 weeks to full feature**

---

## Next Steps

1. **Review PRD.md** — Share with stakeholders, get buy-in on scope
2. **Create GitHub repo** — `github.com/TesseractWorks/photo-curator`
3. **Set up monorepo** — Subprojects: ingest-app/, server/, web-ui/
4. **Begin Phase 1** — Start with ingest app (simplest to validate)
5. **Iterate** — Share early builds with photographers for feedback

---

## FAQ

### Q: Why not use existing photo management tools (Lightroom, Capture One, etc.)?
**A**: Those are post-processing tools. PhotoCurator is specialized for the **ingestion + culling workflow**:
- Automated sharpness filtering (saves 30-40% of time)
- Subject detection for fast categorization
- Keyboard-optimized culling (not mouse-heavy)
- Local-first (no cloud sync delays)
- Photographer-focused (not designed for editors)

### Q: Will this support cloud upload?
**A**: v1 is local network only (home/office server). Cloud support is Phase 2+, with optional S3 backend.

### Q: Can we support Windows/Linux for ingest?
**A**: v1 is macOS only (SwiftUI + native APIs). Windows/Linux would require Qt or Electron rewrite, adding 3-4 weeks.

### Q: What about AI-generated subject suggestions?
**A**: v1 uses YOLO v8 (deterministic). Optional: v2+ could add OpenAI/Claude for natural-language suggestions.

### Q: How do we handle RAW files?
**A**: We store originals + generate JPEG proxies for web display. EXIF/metadata extracted from RAW before conversion.

### Q: Can photographers customize detection classes?
**A**: v1 has hardcoded 15 classes. v2+ will support custom YOLO fine-tuning per photographer.

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-05-22 | Kenneth | Initial PRD: complete spec + architecture |

---

## Contact & Questions

For questions or feedback on PhotoCurator:
1. File an issue in the GitHub repo
2. Contact the product lead: kenneth@tesseractworks.com

---

**Status**: 📋 PRD Complete | ✅ Ready for Development | 🚀 Next: Phase 1 Kickoff

