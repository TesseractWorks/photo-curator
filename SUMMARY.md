# PhotoCurator - Executive Summary

## Project Name: **PhotoCurator**

A professional photo ingestion, evaluation, and culling system consisting of:
1. **Ingest App** (macOS) - one-click photo import from memory cards
2. **Analysis Server** - automatic sharpness scoring and subject detection
3. **Web Cull Interface** - rapid culling and export workflow

---

## Why PhotoCurator?

**The Problem**: Photographers waste hours transferring, sorting, and culling large shoots (1000+ images) with no intelligent filtering.

**The Solution**: Automate ingestion and pre-qualify images by sharpness & subject, leaving the photographer to focus on artistic decisions.

**The Name**: "Curator" emphasizes **intelligent selection**, not just storage. The system curates; the photographer decides.

---

## Core Features

### 1. Ingestion (Ingest App)
- Select any directory (memory card, external drive, folder)
- Recursively find JPEG, RAW, PNG, HEIC, TIFF files
- Upload to server with progress UI
- Preserve all EXIF metadata

### 2. Analysis (Server)
**Sharpness Scoring**
- Laplacian-based algorithm: 0–100 scale
- User-configurable threshold (default 65)
- Per-image execution: <2 seconds
- Goal: Remove obviously blurry shots

**Subject Detection**
- YOLO v8 embedded model (no cloud)
- 15+ detection classes: person, group, food, flowers, rings, venue, landscape, etc.
- Confidence scores stored with image
- Embedded as EXIF tags for archive

### 3. Culling (Web UI)
- Browse all images with real-time filtering
- Keep / Reject rapid workflow (<3 sec per decision)
- Keyboard-driven (arrow keys, K=keep, X=reject, Z=undo)
- Filmstrip timeline for quick navigation
- Bulk operations (select multiple, apply decision)
- Export with metadata intact

---

## Key Differentiators

| Feature | PhotoCurator | Typical Tool |
|---------|--------------|-------------|
| **Sharpness Filtering** | ✓ Automatic scoring | Manual review only |
| **Subject Recognition** | ✓ 15+ classes embedded in image | No detection |
| **Local Processing** | ✓ No cloud APIs | Often cloud-dependent |
| **Keyboard-First** | ✓ Photographer workflow optimized | GUI-centric |
| **Metadata Preservation** | ✓ EXIF + custom tags | Often lost |

---

## Architecture (3-Tier)

```
Ingest App            Server                 Web UI
(Swift/SwiftUI)      (FastAPI/Python)      (React/TypeScript)
   ↓                      ↓                       ↓
[Directory Select] → [Receive + Store] → [Browse Images]
[Upload 4-8 par]    [Sharpness Score]     [Rate/Cull]
[Progress UI]       [Subject Detect]      [Keyboard Nav]
                    [Thumbnail Gen]       [Export]
                    [Metadata Store]
```

---

## Timeline

- **Phase 1 (MVP)**: 4–6 weeks
  - Ingest app, basic server, sharpness scoring, simple web UI
  
- **Phase 2**: 2–3 weeks
  - Subject detection (YOLO), embedded labels
  
- **Phase 3**: 2–3 weeks
  - Advanced culling UX, keyboard shortcuts, bulk ops
  
- **Phase 4**: 1–2 weeks
  - Performance, error handling, documentation

**Total: 10–14 weeks to full feature**

---

## Success Metrics

✓ Ingest 100+ images in <5 minutes  
✓ Sharpness accuracy >90%  
✓ Subject detection >95% recall on common classes  
✓ Culling workflow <3 seconds per image  
✓ Wedding shoot (1000+ images) → final selection in <1 hour

---

## Deployment

**v1 (Local Network)**
- Ingest app uploads to home/office server
- Server runs on NAS, Mac mini, or dedicated Linux box
- All data stays on-premises

**v2+ (Cloud)**
- Optional remote access via https://curator.example.com

---

## File Structure

```
photo-curator/
  ├── ingest-app/          (macOS Swift app)
  ├── server/              (Python FastAPI backend)
  ├── web-ui/              (React frontend)
  ├── docs/                (architecture, API, deployment)
  └── PRD.md               (this document)
```

---

## Getting Started

1. Clone the photo-curator repo
2. Follow `DEVELOPMENT.md` for setup
3. Start with Phase 1 tasks (see `PRD.md` roadmap)
4. Iterate based on user feedback

---

## Document Index

| Document | Purpose |
|----------|---------|
| **PRD.md** | Full product requirements, use cases, features |
| **IMPLEMENTATION.md** | Technical roadmap, architecture decisions, project structure |
| **ARCHITECTURE.md** | Deep dive on system design, data flow, tech stack rationale |
| **DEVELOPMENT.md** | Setup, contribution guidelines, coding standards |
| **API.md** | Server endpoint documentation |
| **DEPLOYMENT.md** | Docker, Kubernetes, production setup |

---

**Status**: PRD Complete ✓  
**Next Step**: Create repository and begin Phase 1 development  
**Date**: 2026-05-22

