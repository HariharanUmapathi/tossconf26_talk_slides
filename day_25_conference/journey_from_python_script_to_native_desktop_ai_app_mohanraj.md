---
title: "Journey from Python Script to Native Desktop AI App"
sub_title: "Semantic Video Search for Video Editors"
author: "Mohan Raj M (@praem90)"
theme:
  name: terminal-dark
---

The Motivation
===
 * **The Goal:** Build a new Micro-SaaS product.
 * **The Foundation:** Leverage prior experience with Live Streaming solutions.
 * **The Spark:** Deep curiosity to learn and apply AI in a highly practical way.
 * **The Formula:** Chose Media as the Domain and AI as the Tool to solve real problems.

<!-- end_slide -->

The Problem: The "Flashback" Nightmare
===
 * Spoke directly with a professional **TV Serial Content Editor**.
 * **The Pain:** Finding an old scene from a TV serial across dozens of unindexed, dusty hard drives is incredibly tedious.
 * **The Workflow:** 
   * Plug in multiple physical drives.
   * Open massive, proprietary video files one by one.
   * Manually scrub timelines looking for a single scene.

<!-- speaker_note: Insert the "billion dollar idea" meme here -->
<!-- end_slide -->

The Solution: Semantic Video Search
===
 * **RAG-Powered:** Build a semantic video search engine that indexes content via description rather than static tags.
 * **High Performance:** Must handle large video files locally and provide sub-second search results.
 * **Privacy First:** Video files are large and proprietary; cloud uploads are a complete dealbreaker.
 * **Visual Context:** Generate instant thumbnails for search results to quickly identify the scene.
 * **Physical Asset Mapping:** Must tag matches with the physical hard disk name and file path.
 * **Workflow Integrated:** Support direct drag-and-drop into standard video editing software.

<!-- speaker_note: Insert the "rukko jara" meme here -->
<!-- end_slide -->

The MVP: The Elegant Python Hack
===
 * **The Stack:** OpenCV + Sentence-Transformers + LanceDB + FastAPI.
 * **Why LanceDB?** Serverless, embedded vector DB (like SQLite for vectors). No heavy server infrastructure required.
 * **The Execution:** Lives completely in a terminal, lists matching timestamps, and renders frames using an OpenCV window.

 ```text
 [ Indexing Phase ]
 Video File ---> Parse Frames ---> CLIP ViT Model ---> Vector DB

 [ Search Phase ]
 Text Query ---> CLIP Text Model ---> Cosine Similarity Check ---> Vector DB
                                                                       |
                                                                       v
                                                              Timestamps Found
                                                                       |
                                                                       v
                                                              OpenCV Render Loop
 ```

 ```text
 +-------------------+
 |    Video File     |
 +---------+---------+
           |
           v
 +-------------------+
 |   Python Script   | <--- User Types Query
 |  (OpenCV Indexer) |
 +---------+---------+
           |
           | (On Keypress)
           v
 +-------------------+
 |   OpenCV Window   | ---> Render Frames
 |   [Img1] [Img2]   |      One-by-One
 +-------------------+
 ```

<!-- end_slide -->

The MVP: Reality Check
===
 * **The UI Problem:** OpenCV image windows lack player controls, scrubbing, timeline visualization, or audio.
 * **The Math Problem:** Raw cosine similarity scores from the CLIP model were incredibly narrow.
   * Top match confidence score: **~25%**
 * **The Solution:** Hack the math for the user interface!
   * Mapped `0% - 25%` actual score -> `0% - 100%` UI Confidence.
 * **Verdict:** It beautifully proved the concept, but it wasn't a real product yet.

<!-- end_slide -->

The Pivot: Moving to Native Desktop Architecture
===
 * Editors aren't engineers. Running a CLI Python script is a massive barrier to entry.
 * **The Goal:** A clean GUI for dragging/dropping videos and instantly viewing results with full metadata.
 * **The Shell:** Chose `Tauri` over Electron due to its tiny bundle size and exceptional developer experience.
 * **The Multi-Language Stack:** React + Tailwind CSS + Rust + Python Backend.

 ```text
 +-------------------------+
 |      React Frontend     |
 +------------+------------+
              | (Tauri IPC)
              v
 +-------------------------+      (HTTP)      +-------------------------+
 |       Rust Backend      | ---------------> |  FastAPI Python Engine  |
 +-------------------------+                  +-------------------------+
 ```
 <!-- speaker_note: React+Tailwindcss -> Tauri -> Engine -->

<!-- end_slide -->

The Packaging Nightmare
===
 * **The Trap:** Tauri does not bundle Python runtimes or FastAPI components by default.
 * **The Fix:** Used `PyInstaller` to compile the FastAPI server into a standalone executable binary.
 * **The Delivery:** Deployed via Tauri's **Sidecar Pattern**.
 * **The Reality Check:**
   * **Port Conflicts:** What if the hardcoded FastAPI port is already used by another process?
   * **Race Conditions:** Slow FastAPI startup times caused Tauri's UI connections to fail on launch.
   * **Bloatware:** The bundle size exploded because we were shipping an entire packaged Python environment.

<!-- end_slide -->

The End Game: Rust All The Things
===
 * To fix the sidecar mess, we had to eliminate Python entirely from the system.
 * **The Challenge:** Minimal Rust experience. Time to learn by building and failing!
 * **The Crucial Discoveries:**
   * `LanceDB` is written in Rust (the Python SDK was just a wrapper around core Rust code).
   * `Sentence-Transformers` is just a wrapper around HuggingFace models using PyTorch.
   * **The Pivot:** Adopted `HuggingFace`'s **Candle** framework for pure, native Rust AI inference.
   * **The Video Parser:** Swapped out heavy OpenCV dependencies for lightweight, native `FFmpeg` bindings.

<!-- end_slide -->

Putting It All Together
===
 * **The Engineering Reality:** 
   * Highly limited documentation 
   * Strict compiler type systems
   * Heavy async engineering.
 * **The Result:** A highly efficient, single-binary application.

 ```text
 +--------------------------------------------------------+
 |                      TAURI APP                         |
 |                                                        |
 |   +-------------------+        +-------------------+   |
 |   |  React UI Layer   | <----> |   Rust App State  |   |
 |   +-------------------+        +---------+---------+   |
 |                                          |             |
 +------------------------------------------|-------------+
                                            v
                         +--------------------------------+
                         |      PURE RUST AI ENGINE       |
                         |                                |
                         |  [FFmpeg]   ->   [Candle ML]   |
                         |  (Frames)         (CLIP ViT)   |
                         |                        |       |
                         |                        v       |
                         |                   [LanceDB]    |
                         +--------------------------------+
 ```

<!-- end_slide -->

From Concept to Completion: Key Decisions in Architecture
===
Every architectural shift was driven by a concrete constraint: performance, distribution size, or user experience.

| The Component | Initial Approach | Final Architecture | The Core Reason |
| :--- | :--- | :--- | :--- |
| **Interface** | Python CLI | Native Desktop GUI (Tauri) | UX for non-technical editors |
| **Backend API** | FastAPI (Python Sidecar) | Native Rust Runtime | Eliminated IPC lag & port conflicts |
| **Vector DB** | LanceDB | LanceDB (Kept!) | Embedded simplicity beats heavy external DBs |
| **Video Decoding** | OpenCV Loop | FFmpeg Bindings | Massive speedups & hardware acceleration |
| **Data Pipeline** | Frame -> Disk JPEG -> Model | Raw Byte Array Buffers | Saved disk I/O thrashing & memory overhead |


<!-- end_slide -->

The Future Roadmap
===
 * **Multimodal Search:** Integrate **Whisper** to transcribe audio, linking spoken dialog timestamp ranges directly with video frame embeddings.
 * **Local Agents:** Deploy a local LLM to handle complex contextual user queries ("Find the scene where they look happy, but then suddenly get angry").
 * **Workflow Extensibility:** Build Model Context Protocol (MCP) tools for automated post-search actions like instant clip trimming, stitching, or asset exports.
