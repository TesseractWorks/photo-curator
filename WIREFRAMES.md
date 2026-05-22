# PhotoCurator - Visual Guides

This document contains wireframes, user flows, and visual architecture diagrams for PhotoCurator.

---

## Wireframe 1: Ingest App (macOS)

```
╔════════════════════════════════════════════════════════╗
║                 PhotoCurator Ingest                     ║
║                                                          ║
║  📁 Select Source Directory                             ║
║  ┌──────────────────────────────────────────────────┐  ║
║  │ /Volumes/Canon_SD_Card              [Browse...]  │  ║
║  └──────────────────────────────────────────────────┘  ║
║                                                          ║
║  ✓ Scan Results                                         ║
║  • Found 247 images (356 GB total)                      ║
║  • File types: JPEG (180), CR3 RAW (67)                 ║
║  • Date range: 2026-05-22 → 2026-05-22                 ║
║                                                          ║
║  ┌──────────────────────────────────────────────────┐  ║
║  │ ⚙️  Upload Settings                               │  ║
║  │ • Parallel uploads: 4                             │  ║
║  │ • Destination: https://curator.local:8443        │  ║
║  │ • Local staging: ~/PhotoCurator/staging/          │  ║
║  └──────────────────────────────────────────────────┘  ║
║                                                          ║
║  [Start Upload]                    [Cancel]             ║
║                                                          ║
║  Status: Ready to upload                                ║
║                                                          ║
╚════════════════════════════════════════════════════════╝
```

### Upload Progress Screen

```
╔════════════════════════════════════════════════════════╗
║                 PhotoCurator Ingest                     ║
║                                                          ║
║  Uploading: 2026-05-22 Wedding Reception               ║
║                                                          ║
║  Progress: 156/247 (63%)                                ║
║  ████████████████░░░░░░░░░░░░░░░░░░░░░░░                ║
║                                                          ║
║  Current: IMG_1156.CR3 (45 MB)                          ║
║  Speed: 2.8 MB/s                                        ║
║  Time elapsed: 18 minutes                               ║
║  Est. time remaining: 11 minutes                        ║
║                                                          ║
║  Uploaded queue:                                        ║
║  • IMG_1153.JPG ✓                                       ║
║  • IMG_1154.CR3 ✓                                       ║
║  • IMG_1155.JPG ✓                                       ║
║  • IMG_1156.CR3 [uploading...]                          ║
║  • IMG_1157.JPG [queued]                                ║
║  • IMG_1158.JPG [queued]                                ║
║                                                          ║
║  [Pause]  [Cancel]                                      ║
║                                                          ║
║  Session ID: 2026-05-22-wedding-001                     ║
║  Server: https://curator.local:8443 ✓                   ║
║                                                          ║
╚════════════════════════════════════════════════════════╝
```

---

## Wireframe 2: Web UI - Sessions Dashboard

```
╔════════════════════════════════════════════════════════════════════╗
║                       PhotoCurator Web                              ║
║                    https://curator.local:8443                       ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  📋 Sessions                              [+ New Ingest]  🔍 Search ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐ ║
║  │ 📆 2026-05-22 Wedding Reception         Status: ✓ COMPLETE    │ ║
║  │ 247 images • 3.2 GB • 2 days ago       Analyzed: 247/247     │ ║
║  │                                                                 │ ║
║  │ Quality:  Sharp ████████░ 89% (220/247)                       │ ║
║  │ Subjects: 👤 person (215), 🌸 flowers (180), 💍 rings (95)    │ ║
║  │                                                                 │ ║
║  │ Culling:  ✓ Kept (125)  ✗ Rejected (122)  ⏳ Pending (0)      │ ║
║  │ Decision rate: 98% | Time: 1h 22m avg per 100 imgs           │ ║
║  │                                                                 │ ║
║  │ [Open Cull]  [Browse]  [Export...]  [Archive]  [Delete]      │ ║
║  └────────────────────────────────────────────────────────────────┘ ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐ ║
║  │ 📆 2026-05-21 Reception Cocktail Hour  Status: ✓ COMPLETE     │ ║
║  │ 156 images • 2.1 GB • 3 days ago       Analyzed: 156/156     │ ║
║  │                                                                 │ ║
║  │ Quality:  Sharp ██████░░░░ 76% (119/156)                      │ ║
║  │ Subjects: 👥 group (98), 🍽️ food (112), 💡 lighting (75)     │ ║
║  │                                                                 │ ║
║  │ Culling:  ✓ Kept (89)  ✗ Rejected (67)  ⏳ Pending (0)        │ ║
║  │                                                                 │ ║
║  │ [Open Cull]  [Browse]  [Export...]  [Archive]  [Delete]      │ ║
║  └────────────────────────────────────────────────────────────────┘ ║
║                                                                      ║
║  Pagination: [1] [2] [3] ... [>]                                    ║
║                                                                      ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Wireframe 3: Web UI - Culling Interface (Main)

```
╔════════════════════════════════════════════════════════════════════╗
║  PhotoCurator Cull  |  Wedding Reception (247 images)              ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Progress: Culled 125 ✓ | 122 ✗ | 0 ⏳   (50% complete)           ║
║  ████████████████████░░░░░░░░░░░░░░░░░░░░░                         ║
║                                                                      ║
║  ┌─────────────────────────────────────────────────────────────┐   ║
║  │                                                             │   ║
║  │                                                             │   ║
║  │                  [← PREV IMAGE]                            │   ║
║  │                                                             │   ║
║  │               ┌──────────────────────┐                      │   ║
║  │               │                      │                      │   ║
║  │               │                      │                      │   ║
║  │               │                      │                      │   ║
║  │               │    MAIN IMAGE        │                      │   ║
║  │               │    (1600 x 1067)     │                      │   ║
║  │               │                      │                      │   ║
║  │               │    Canon EOS R5      │                      │   ║
║  │               │    Portrait couple   │                      │   ║
║  │               │                      │                      │   ║
║  │               │                      │                      │   ║
║  │               └──────────────────────┘                      │   ║
║  │                                                             │   ║
║  │                  [NEXT IMAGE →]                            │   ║
║  │                                                             │   ║
║  └─────────────────────────────────────────────────────────────┘   ║
║                                                                      ║
║  Image Details:                                                      ║
║  • Name: 2026-05-22T14-30-15Z_IMG_1156.CR3                          ║
║  • Camera: Canon EOS R5  • ISO: 400  • Shutter: 1/500  • f/2.8      ║
║  • Focal length: 85mm                                               ║
║                                                                      ║
║  🎯 Sharpness: ██████░░ 87/100 (PASSING ✓)                          ║
║  📌 Subjects: 👤 person (0.98) | 💐 flowers (0.76)                  ║
║  ⭐ Your rating: ○ ○ ○ ○ ○  [Click to rate 1-5 stars]              ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐ ║
║  │ [✓ KEEP]      [✗ REJECT]      [? UNDECIDED]   [ℹ️ More Info]   │ ║
║  └────────────────────────────────────────────────────────────────┘ ║
║                                                                      ║
║  Keyboard shortcuts: K=Keep, X=Reject, Z=Undo, →/←=Nav, [1-5]=Rate║
║                                                                      ║
║  ┌─ Filmstrip ─────────────────────────────────────────────────────┐ ║
║  │ [1154]⬜[1155]⬜[1156]🟦[1157]⬜[1158]⬜[1159]⬜[1160]⬜[...]       │ ║
║  │  ✓    ✗    ⏳   ✓    ✓    ✗    ✗              (showing 25%)    │ ║
║  └─────────────────────────────────────────────────────────────────┘ ║
║                                                                      ║
╚════════════════════════════════════════════════════════════════════╝
```

**Legend:**
- 🟦 = Current image (highlighted)
- ⬜ = Undecided
- ✓ = Kept (green in actual UI)
- ✗ = Rejected (red in actual UI)

---

## Wireframe 4: Web UI - Browse & Filter

```
╔════════════════════════════════════════════════════════════════════╗
║  PhotoCurator Browse  |  Wedding Reception                          ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  📊 Filter & Search                                                 ║
║  ┌─────────────────────────────────────────────────────────────┐   ║
║  │ Sharpness: [========●──] 65+  [Show all]                    │   ║
║  │ Subjects:  [✓] Person  [✓] Flowers  [✓] Rings  [✗] Food    │   ║
║  │            [✗] Group   [✓] Venue    [✗] Other               │   ║
║  │ Status:    [✓] Kept  [✓] Rejected  [○] Undecided            │   ║
║  │ Rating:    [○] 5★  [○] 4★  [○] 3★  [○] 2★  [○] 1★  [○] Any  │   ║
║  │ Search:    [_________________]  Tags: [____________]         │   ║
║  │                                                              │   ║
║  │ [Apply Filters]  [Reset]  [Save View]                       │   ║
║  └─────────────────────────────────────────────────────────────┘   ║
║                                                                      ║
║  Results: 47 of 247 images match                                    ║
║  [Grid] [List] [Comparison] | Sort by: Date ▼                      ║
║                                                                      ║
║  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐                 ║
║  │ [1]⬜│ [2]✓│ [3]✗│ [4]⬜│ [5]⬜│ [6]⬜│ [7]⬜│ [8]⬜│                 ║
║  │ 87% │ 91% │ 54% │ 78% │ 82% │ 69% │ 95% │ 71% │                 ║
║  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘                 ║
║                                                                      ║
║  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐                 ║
║  │ [9]⬜│[10]⬜│[11]✗│[12]⬜│[13]✓│[14]⬜│[15]⬜│[16]⬜│                 ║
║  │ 88% │ 79% │ 45% │ 92% │ 84% │ 73% │ 89% │ 76% │                 ║
║  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘                 ║
║                                                                      ║
║  Selected: 6 images                                                 ║
║  [✓ Mark all as Keep] [✗ Mark all as Reject] [⭐ Bulk Rate...]     ║
║  [📥 Export Selected] [+ Add Tag] [🔗 Create Album]                 ║
║                                                                      ║
║  Pagination: [1] [2] [3] [4] [5] [>]                                ║
║                                                                      ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Wireframe 5: Export Dialog

```
╔════════════════════════════════════════════════════════════════════╗
║              Export Culled Images                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Session: Wedding Reception  |  Total images: 247  |  Kept: 125    ║
║                                                                      ║
║  📋 What to export?                                                 ║
║  ☑ Only "Kept" images (125 images)                                  ║
║  ☐ All images + decisions (247 images)                              ║
║  ☐ Only "Rejected" images (122 images)                              ║
║                                                                      ║
║  📂 Format & Options                                                ║
║  ┌─────────────────────────────────────────────────────────────┐   ║
║  │ Format:                                                     │   ║
║  │ ☑ JPEG + Metadata (XMP tags)        ← Recommended          │   ║
║  │ ☐ Original Files (RAW + JPEG pairs)                         │   ║
║  │ ☐ Contact Sheet (PDF, 20/page)                              │   ║
║  │ ☐ Metadata Only (CSV spreadsheet)                           │   ║
║  │                                                             │   ║
║  │ File structure:                                             │   ║
║  │ ☑ Organize by sharpness rating                              │   ║
║  │ ☑ Include EXIF data                                         │   ║
║  │ ☑ Embed subject tags                                        │   ║
║  │ ☑ Embed your star ratings                                   │   ║
║  │                                                             │   ║
║  │ Size: ~2.3 GB (estimated)                                   │   ║
║  │                                                             │   ║
║  └─────────────────────────────────────────────────────────────┘   ║
║                                                                      ║
║  💾 Destination                                                     ║
║  ☑ Download as ZIP file (local)                                    ║
║  ☐ Upload to cloud (Google Drive, Dropbox, etc.)                   ║
║  ☐ Copy to external drive: [Select disk...]                        ║
║                                                                      ║
║  [Export]  [Cancel]                                                 ║
║                                                                      ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## User Flow: End-to-End Culling

```
Start
  │
  └─> Photographer returns from shoot
       │
       ├─ [Run Ingest App on Mac]
       │   │
       │   ├─ Select memory card directory
       │   ├─ View scan results (247 images)
       │   ├─ Click [Start Upload]
       │   │
       │   └─ Watch progress bar
       │       (18 min upload time)
       │
       └─> Images now on server, analyzing...
            │
            ├─ [Open web UI in browser]
            │   https://curator.local:8443
            │
            ├─ See sessions dashboard
            │   (shows upload progress in real-time)
            │
            ├─ Click [Open Cull] when ready
            │   (analysis might still be running)
            │
            ├─ Start culling with keyboard:
            │   │
            │   ├─ Arrow Right → next image
            │   │
            │   ├─ View image
            │   │   • Sharpness: 87/100 ✓
            │   │   • Subjects: person, flowers
            │   │
            │   ├─ Press K → KEEP (move to next)
            │   │
            │   └─ Repeat 247 times (~1.5 hours)
            │       (250 images ÷ 3 sec/image = 12 min actual decision time)
            │       (+ browsing, reviewing, second-guessing = 1-2 hours typical)
            │
            ├─ [Arrow Left] to review previous decisions
            │
            ├─ [Z] to undo mistakes
            │
            ├─ Use filter to show only "kept" + high sharpness
            │
            ├─ Final review: spot-check 10-20 images
            │
            └─> Ready to export
                 │
                 ├─ Click [Export...]
                 │
                 ├─ Select options:
                 │   • Format: JPEG + Metadata
                 │   • What: Only "Kept" images
                 │
                 ├─ Click [Export]
                 │   (ZIP file downloads)
                 │
                 └─> Done! (~2.3 GB of final images)
                     Ready to post-process in Lightroom
```

---

## Data Flow: Sharpness Scoring

```
Raw Image File (IMG_1234.CR3)
         │
         ├─ [Extract metadata]
         │   └─ Camera: Canon EOS R5
         │   └─ ISO: 400
         │   └─ Timestamp: 2026-05-22T14:30:15Z
         │
         ├─ [Convert to working format]
         │   └─ CR3 → JPEG (internal)
         │
         ├─ [Sharpness Scorer]
         │   ├─ Load image
         │   ├─ Convert to grayscale
         │   ├─ Apply Laplacian filter
         │   │   └─ Detects high-frequency content
         │   ├─ Compute variance
         │   │   └─ variance = 8500.0
         │   ├─ Look up camera calibration
         │   │   └─ Canon EOS R5: min=150, max=12000
         │   ├─ Normalize to 0-100
         │   │   └─ score = ((8500 - 150) / (12000 - 150)) * 100 = 87
         │   └─ Check threshold (default: 65)
         │       └─ 87 >= 65 → PASS ✓
         │
         └─> Result
             ├─ score: 87
             ├─ passing: true
             ├─ threshold: 65
             └─ Stored in metadata.json + database
```

---

## Data Flow: Subject Detection

```
JPEG Image
    │
    ├─ [YOLO v8 Model (preloaded)]
    │   ├─ Resize to 640x640
    │   ├─ Normalize pixel values
    │   └─ Forward pass through model
    │
    ├─ [Extract Detections]
    │   ├─ Box 1: person (0.98 confidence)
    │   │   └─ Bounding box: (100, 200, 350, 600)
    │   ├─ Box 2: person (0.95 confidence)
    │   │   └─ Bounding box: (400, 150, 600, 550)
    │   ├─ Box 3: flowers (0.76 confidence)
    │   │   └─ Bounding box: (620, 400, 750, 700)
    │   └─ [No other detections]
    │
    ├─ [Aggregate by class]
    │   ├─ person: count=2, max_confidence=0.98
    │   └─ flowers: count=1, max_confidence=0.76
    │
    └─> Result
        ├─ subjects: [
        │   {"label": "person", "confidence": 0.98, "count": 2},
        │   {"label": "flowers", "confidence": 0.76, "count": 1}
        │ ]
        └─ Stored in metadata.json + EXIF XMP tags
```

---

## Keyboard Navigation Reference

```
╔════════════════════════════════════════════════════════════════════╗
║                    Keyboard Shortcuts                               ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Navigation:                          Culling:                      ║
║  ───────────────────────────────────  ───────────────────────────  ║
║  →  / Right Arrow    Next image       K  Keep / Approve             ║
║  ←  / Left Arrow     Previous image   X  Reject / Delete            ║
║  Home             First image         Z  Undo last decision         ║
║  End              Last image          ?  Undecided (clear decision) ║
║                                                                      ║
║  Rating:                              View:                        ║
║  ───────────────────────────────────  ───────────────────────────  ║
║  1  ★☆☆☆☆  (1 star)                  F  Fullscreen                 ║
║  2  ★★☆☆☆  (2 stars)                 G  Grid/Browse view           ║
║  3  ★★★☆☆  (3 stars)                 D  Dashboard                  ║
║  4  ★★★★☆  (4 stars)                 H  Help / Shortcuts           ║
║  5  ★★★★★  (5 stars)                 E  Export options             ║
║  0  ☆☆☆☆☆  (Clear rating)           Q  Quick filter               ║
║                                                                      ║
║  Filter:                              Other:                        ║
║  ───────────────────────────────────  ───────────────────────────  ║
║  Shift+F   Open filter panel         Ctrl+S Save session state     ║
║  Shift+K   Show only "Kept"          Ctrl+Z Global undo            ║
║  Shift+X   Show only "Rejected"      /   Search by filename        ║
║  Shift+A   Show all                  Esc  Deselect / Close          ║
║                                                                      ║
║  Pro tip: Most actions can be performed without touching mouse!     ║
║           Master the keyboard → culling becomes a rhythm activity.  ║
║                                                                      ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Performance Timeline: Processing 1000 Images

```
0 min        Ingest starts (upload from camera)
  ├─ 10-15 min: Upload complete (1000 × 3.5 MB ÷ 2.5 MB/s)
  │
15 min       Analysis begins
  │
  ├─ +15 min: Sharpness scoring (1000 images)
  │            ├─ 1000 × 2 sec = 33 min single-threaded
  │            └─ 8 parallel workers = ~4 min
  │
  ├─ +5 min:  Subject detection (YOLO v8)
  │            ├─ 1000 × 1 sec (GPU) = 17 min
  │            └─ 8 parallel workers = ~2 min
  │
  ├─ +3 min:  Thumbnail generation
  │            ├─ 1000 × 0.5 sec per version
  │            └─ 8 parallel workers = ~1 min
  │
  └─ +2 min:  Database updates + EXIF embedding
               └─ Bulk insert + metadata write
30 min       ✓ All analysis complete
             ✓ Ready for culling

User starts culling:
30 min       Open web UI, see dashboard
             Images ready to cull
35 min       Begin culling first 50 images
             (learn keyboard shortcuts: ~6 sec/image)
45 min       Hit stride, culling at 3 sec/image
             ├─ 1000 images × 3 sec = 50 minutes culling time
             └─ Reality: ~1-2 hours (with review/hesitation)

1.5-2 hours  ✓ Culling complete
             ✓ 300 "Kept" images selected
             ✓ Ready for export

Export:      3-5 minutes
             ├─ Prepare images
             ├─ Generate ZIP
             └─ Download (depends on disk speed)

Total wall-clock time from shoot → final images: 2-2.5 hours
Total active photographer time: ~1.5 hours
Photographer benefit: Saved 2-3 hours vs. manual culling
```

---

*End of Visual Guides Document*

