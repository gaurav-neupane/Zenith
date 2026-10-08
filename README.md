<p align="center">
  <img src="Media/Zenith Screenshot/Custom 3D Mesh.png" alt="Zenith Banner" width="100%"/>
</p>

<h1 align="center">🛰️ Zenith — Satellite Depth & 3D Reconstruction</h1>

<p align="center">
  <b>Transform a single satellite image into an interactive, navigable 3D terrain model</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/SIH_2026-PS_26175-blue?style=for-the-badge" alt="SIH 2026"/>
  <img src="https://img.shields.io/badge/Theme-Disaster_Management-red?style=for-the-badge" alt="Theme"/>
  <img src="https://img.shields.io/badge/Category-Software-green?style=for-the-badge" alt="Category"/>
  <img src="https://img.shields.io/badge/MAE-1.54_m-brightgreen?style=for-the-badge" alt="MAE"/>
  <img src="https://img.shields.io/badge/RMSE-1.89_m-brightgreen?style=for-the-badge" alt="RMSE"/>
</p>


---

## 📖 Table of Contents

- [Overview](#-overview)
- [The Problem](#-the-problem)
- [Key Features](#-key-features)
- [Application Screenshots](#-application-screenshots)
- [End-to-End Pipeline Architecture](#-end-to-end-pipeline-architecture)
- [Stage 1 — Input Handling & Automatic Detection](#-stage-1--input-handling--automatic-detection)
- [Stage 2 — Depth Anything V2 (Relative Depth Prior)](#-stage-2--depth-anything-v2-relative-depth-prior)
- [Stage 3 — Metric Calibration](#-stage-3--metric-calibration-analyzer-router-urban--terrain-branches)
  - [Mathematical Analyzer (Router)](#mathematical-analyzer-label-free-router)
  - [Urban Branch](#urban-branch)
  - [Metric Calibrator Model (Deep Dive)](#-metric-calibrator--custom-metric-height-model-deep-dive)
  - [Terrain Branch](#terrain-branch)
- [Results & Validation](#-results--validation)
- [Stage 4 — 3D Reconstruction & Rendering Modes](#-stage-4--3d-reconstruction--rendering-modes)
- [Interactive Viewer & User Experience](#-interactive-viewer--user-experience)
- [Export Formats](#-export-formats)
- [Technology Stack](#-technology-stack)
- [Impact & Benefits](#-impact--benefits)
- [Design Decisions](#-design-decisions)
- [Future Scope](#-future-scope)
- [Team](#-team)
- [References](#-references)

---

## 🌟 Overview

**Zenith** is an AI-powered desktop application that transforms a **single satellite or aerial image** into an interactive, navigable 3D terrain model — no stereo pair, no multi-pass, no extra sensor required.

It has three major technical contributions:

1. **Relative depth → metric heights.** A pretrained, frozen **Depth Anything V2 (DA2)** produces a relative DSM with sharp object contours but no scale. A custom, lightweight **Metric Calibrator** (~5.4M parameters) fuses the image with the rDSM using dual encoders, CBAM attention, bidirectional cross-attention and a PixelShuffle decoder to predict a metric **nDSM** — object heights above ground in metres.

2. **Routed relative → absolute elevation.** A label-free **Mathematical Analyzer** examines the shape statistics of the rDSM and decides, patch by patch, whether the area is urban (use the **Metric Calibrator** nDSM + SRTM ground) or terrain (stretch the rDSM onto **SRTM**). Neural networks supply the shape; SRTM supplies the real-world scale.

3. **Elevation → interactive 3D environment.** The elevation map becomes a full 3D scene inside a lightweight **Tauri + Rust** desktop app, rendered with **Three.js (WebGL2)**, with three visualization modes: Point Cloud, 3D Mesh and Object Segmentation.

> *Can a single optical satellite image — the cheapest and most widely available form of remote-sensing data — be sufficient to reconstruct an accurate, interactive 3D elevation model of the terrain?*
>
> **Zenith answers: Yes.**

---

## 🎯 The Problem

Accurate Digital Elevation Models (DEMs) and Digital Surface Models (DSMs) are fundamental to **urban planning, disaster management and military reconnaissance**. Elevation data is traditionally acquired via:

- **Stereo-imaging pairs** — expensive, sensor-dependent
- **LiDAR** — cost-prohibitive at scale
- **Interferometric SAR (InSAR)** — computationally heavy

Single-view height estimation is an agile alternative, but foundational monocular depth models are trained mostly on natural, ground-level imagery and predict **relative** depth. On remote-sensing data they suffer from:

| Challenge | Description |
|---|---|
| **Domain Gap** | Models trained on natural images struggle with satellite imagery scale, textures and viewing geometry |
| **Structural Variation** | Buildings, trees and terrain behave very differently — a single calibration strategy is insufficient |
| **No Absolute Scale** | Monocular depth models predict only relative depth; a single image cannot inherently determine metric elevation |

### Required Deliverable

| Input type | Required output |
|---|---|
| **Non-georeferenced RGB** (PNG / JPG) | **Relative DSM (rDSM)** |
| **Georeferenced RGB** (GeoTIFF) | **Absolute DSM** with metric height values |

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🖼️ **Single-image input** | No stereo pair, no multi-pass, no extra sensor |
| 🔍 **Automatic file-type detection** | One entry point; the file handler decides RGB vs GeoTIFF |
| 🧠 **Custom Metric Calibrator** | ~5.4M params, ~54 GFLOPs, ~44.6 FPS at 1024×1024 — **MAE 1.54 m** on DFC 2019 |
| 🗺️ **Routed metric calibration** | Per-patch choice between urban (Metric Calibrator + SRTM) and terrain (SRTM-calibrated rDSM) |
| 🎨 **Three render modes** | Point Cloud, 3D Mesh, Object Segmentation |
| 🏙️ **RGB texture projection** | Original satellite image ortho-projected onto the reconstructed surface |
| 🎮 **Interactive viewer** | Orbit, flythrough, first-person navigation, sun lighting, height/slope tools |
| 📦 **Export** | OBJ, GLTF/GLB, STL, PLY, GeoTIFF |
| 💻 **Lightweight desktop app** | Tauri (no bundled Chromium), small binary, runs on CPU or light GPU |

---

## 📸 Application Screenshots

### 3D Terrain — Custom Mesh with Texture Projection
<p align="center">
  <img src="Media/Zenith Screenshot/Custom 3D Mesh.png" alt="Custom 3D Mesh" width="90%"/>
</p>

> The main workspace showing the original satellite image, its depth map, and the reconstructed 3D textured mesh with orbit navigation controls and real-time system monitoring.

### Orbital View
<p align="center">
  <img src="Media/Zenith Screenshot/Orbital View.png" alt="Orbital View" width="90%"/>
</p>

> Bird's-eye orbital view of the reconstructed 3D cityscape. Buildings, roads and vegetation are clearly distinguishable with accurate relative heights.

### Mesh Wireframe View
<p align="center">
  <img src="Media/Zenith Screenshot/Mesh View.png" alt="Mesh View" width="90%"/>
</p>

> Wireframe overlay showing the underlying mesh topology — including the curtain-wall extrusion that creates watertight building facades.

### Raw Point Cloud
<p align="center">
  <img src="Media/Zenith Screenshot/Raw Point Cloud.png" alt="Raw Point Cloud" width="90%"/>
</p>

> Dense point cloud rendering where each DSM pixel is back-projected to a 3D point with its original RGB colour. Adjustable point size and density controls.

### Street View (First-Person Navigation)
<p align="center">
  <img src="Media/Zenith Screenshot/Street View.png" alt="Street View" width="90%"/>
</p>

> First-person street-level navigation through the reconstructed 3D scene — experience the terrain from ground level with WASD + mouse controls.

---

## 🏗️ End-to-End Pipeline Architecture

<p align="center">
  <img src="Media/main pipeline ( urban + terrain ).png" alt="Main Pipeline Architecture" width="100%"/>
</p>

The complete Zenith pipeline operates as follows:

```
User (Tauri + Rust desktop app)
   │
   │  Upload image (PNG / JPG / TIFF / GeoTIFF)
   ▼
File Handler — automatic type detection & metadata extraction
   │                                   │
   │ plain RGB (no CRS)                │ GeoTIFF (CRS + GSD present)
   │                                   │  reproject → tile 256×256 (50% overlap)
   │                                   │  fetch SRTM via `elevation` → resample to patches
   ▼                                   ▼
        ┌──────────────────────────────────────────┐
        │  Depth Anything V2 (pretrained, frozen)  │   ← same model for both inputs
        │  relative-depth prior                    │
        └──────────────────────┬───────────────────┘
                               ▼
                       rDSM (relative depth)
                               │
              ┌────────────────┴──────────────────────────────┐
              ▼                                               ▼
      RGB → rDSM (as is)                  GeoTIFF → Mathematical Analyzer (router)
              │                           5 features + SRTM relief → P_urban per patch
              │                                 │                       │
              │                          urban patches            terrain patches
              │                      Metric Calibrator            a·rDSM + b fitted to SRTM
              │                      (image + rDSM → nDSM)       (Huber / RANSAC)
              │                      + smoothed SRTM ground
              │                                 └───────┬───────────────┘
              │                                         ▼
              │                       blend by P_urban → Absolute DSM (m above sea level)
              └────────────────┬────────────────────────┘
                               ▼
                        DSM Generation
                               ▼
                    3D Reconstruction
        ┌──────────────┬───────────────┬──────────────────┐
        ▼              ▼               ▼
   1) Point Cloud   2) 3D Mesh    3) Object Segmentation
                               ▼
                   RGB Texture Projection
                               ▼
                 Three.js WebGL2 Viewer
                               ▼
        Interactive flythrough · height/slope analysis · export
```

---

## 📥 Stage 1 — Input Handling & Automatic Detection

The user never has to specify which kind of image they are loading. The **file handler** automatically:

1. Accepts **PNG, JPG, TIFF, GeoTIFF**
2. Inspects the file for geospatial metadata (CRS, geotransform / GSD, extent) using **rasterio / GDAL**
3. For GeoTIFFs, reprojects to a metric CRS (e.g., UTM), tiles into 256×256 patches with overlap, and fetches matching **SRTM** elevation
4. Routes accordingly:

| Detected as | Metadata found | Output | Render scale |
|---|---|---|---|
| Plain RGB (PNG/JPG) | none | **Relative DSM (rDSM)** | Relative height (no metric units) |
| GeoTIFF | CRS + GSD | **Absolute DSM** (metres above sea level) | **Metric scale shown in render** |

> **Both paths run through the same pretrained DA2 model.** Detection decides what happens *after* DA2.

---

## 🔬 Stage 2 — Depth Anything V2 (Relative Depth Prior)

**Depth Anything V2 (DA2)** is used as a **pretrained, frozen** monocular depth model. Zenith never retrains or modifies it. It acts purely as a **prior generator**: for every input image it predicts a dense relative depth map — the **rDSM**.

### What DA2 provides vs. what it cannot

| ✅ Provides | ❌ Cannot Provide |
|---|---|
| Good object contours and local structure | Metric scale — values are dimensionless |
| Sharp edges and smooth slopes | Absolute height reference |
| Which things are higher relative to others | Real-world metres |

For a satellite sensor ~506 km away (e.g., Gaofen-7), a 5–10 m building is only ~0.1–0.2 ‰ of the sensor distance, so global scale is lost entirely. The rDSM is treated as **shape without units**. Real metres come from the Metric Calibrator (object heights) and SRTM (ground elevation).

### Consumers of the rDSM

| Consumer | Use |
|---|---|
| Plain RGB path | rDSM used directly for visualisation |
| Mathematical Analyzer | Shape statistics (HFR, κ, THR, H) for routing |
| Metric Calibrator | Depth prior fused with image to predict metric nDSM |
| Terrain branch | Shape to be scaled onto SRTM |

---

## 🧭 Stage 3 — Metric Calibration: Analyzer Router, Urban & Terrain Branches

This stage turns the relative DSM from Stage 2 into an **Absolute DSM** in metres above sea level. It runs **only for GeoTIFF inputs**.

> **Central idea:** Neural networks supply the *shape*; SRTM supplies the *real-world scale*; a lightweight router chooses how to combine them for each area.

### Why a Router is Needed

| Scene | What matters | Best source of metres |
|---|---|---|
| **Urban** (buildings, trees, houses) | Heights of objects above ground | **Metric Calibrator** — trained on exactly this |
| **Hilly / rural terrain** | Slope and elevation of the land | **SRTM** — measures terrain directly |

The Metric Calibrator excels in cities but was validated on urban/suburban scenes only. SRTM is blind to individual buildings (~30 m resolution) but reliable for terrain. The router decides **which branch produces the metric result**, per patch.

---

### Mathematical Analyzer (Label-Free Router)

<p align="center">
  <img src="Media/Mathematical analyzer flowchart.png" alt="Mathematical Analyzer Flowchart" width="100%"/>
</p>

For every patch the analyzer decides urban vs. terrain **without any labelled data**.

**Step 1 — Normalise the rDSM per patch** (scale-free):

$$\hat D=\text{clip}\left(\frac{D-p_2}{p_{98}-p_2},\,0,\,1\right)$$

**Step 2 — Extract five scale-invariant features:**

| Symbol | Feature | Definition | Direction |
|---|---|---|---|
| **HFR** | High-frequency energy ratio | Var(D̂ − G_σ∗D̂) / Var(D̂), σ ≈ one building size | high → urban |
| **κ** | Gradient kurtosis (edge sharpness) | E[(g−μ)⁴]/σ⁴ with g = \|∇D̂\| | high → urban |
| **THR** | Top-hat ratio (object-above-ground) | Mean top-hat / mean range | high → urban |
| **H** | Hurst exponent (smoothness) | slope/2 of log-variogram vs. log-lag | high → terrain |
| **R_s** | SRTM relief | log(1 + max S − min S) | high → terrain |

> Urban areas have many sharp, frequent height jumps (0 m, 40 m, 0 m, 10 m). Hills have smooth, large, slow changes.

**Step 3 — Z-score with fixed reference statistics.** z(x) = (x − μ_ref) / σ_ref, clipped to [−3, 3]. Reference statistics are computed **once** from a small, diverse set (~20–40 images: Delhi, Mumbai, Himalayas, Western Ghats, farmland) and reused forever. No per-patch annotation needed.

**Step 4 — Score and probability:**

$$s=\frac{z_{HFR}+z_{\kappa}+z_{THR}-z_{H}-z_{R_s}}{5}, \quad P_{urban}=\frac{1}{1+e^{-k\,s}}, \quad k\approx3$$

**Step 5 — Spatial smoothing.** Gaussian smoothing over the patch grid prevents neighbouring patches from flickering.

**Step 6 — Route or blend:**

| P_urban | Action |
|---|---|
| > 0.65 | **Urban branch** |
| < 0.35 | **Terrain branch** |
| 0.35 – 0.65 | **Blend:** P · DSM_urban + (1−P) · DSM_terrain |

Soft blending avoids visible seams at boundaries such as hill towns.

---

### Urban Branch

1. The **Metric Calibrator** takes the RGB patch **and** its rDSM and predicts a **metric nDSM**: object height above local ground in metres (e.g., building = 18 m, road = 0 m).
2. **Add ground elevation from SRTM:**

$$DSM_{urban}(x,y) = nDSM(x,y) + Ground(x,y)$$

`Ground` is a **smoothed** SRTM (low-pass or morphological opening) to avoid double-counting objects already present in raw SRTM.

---

### 🧠 Metric Calibrator — Custom Metric Height Model (Deep Dive)

<p align="center">
  <img src="Media/Caliberatopr model.png" alt="Metric Calibrator Architecture" width="85%"/>
</p>

The Metric Calibrator is a **custom model designed and trained by the Zenith team**. Its design is inspired by RDAH-Net (Jiang et al., *Remote Sensing* 2026).

#### Task

Predict a **normalised DSM (nDSM)** — above-ground object heights in metres — from a **single** image plus its DA2 rDSM. No stereo pair or LiDAR is needed at inference.

#### Core Idea

DA2 gives a relative depth map with sharp structure but no metric scale. The Metric Calibrator learns to fuse that depth prior with the image and convert it into **absolute height**. DA2 stays **frozen**; only the calibrator is trained.

#### Architecture

Two parallel encoders (image and depth, **not weight-shared**) feed a cross-modal fusion stage; a PixelShuffle decoder then outputs the nDSM.

```
RGB image (3 ch) ─► Conv 3→32 ─► Image Encoder (MobileViT-S-Light) ──┐  multi-scale features
                                                                       ├─► Feature Enhancement Module
DA2 rDSM  (1 ch) ─► Conv 1→32 ─► Depth Encoder (MobileViT-S-Light) ──┘  (3 scales, coarse → fine)
                                                                            per scale:
                                                                            CBAM → LightCrossAttention I→D
                                                                                 + LightCrossAttention D→I → sum → Conv
                                                                                     │
                       Decoder: PixelShuffle Upsampler ×4 (one per fusion scale + final),
                                fed by positional encoding + Light Transformer
                                                                                     ▼
                                                               Conv 8→1  →  nDSM (m above ground)
```

#### Component Breakdown

| Component | Role |
|---|---|
| **Dual Encoder** | Lightweight `MobileViT-S-Light` per modality, giving multi-scale features. Three stages (32→64, 64→128, 128→256 channels); each output reduced to 32 channels by Conv before fusion. |
| **CBAM (Convolutional Block Attention Module)** | Placed after each encoder stage: channel attention (MaxPool + AvgPool → shared MLP → sigmoid) followed by spatial attention (Conv → sigmoid). Emphasises height-relevant features and suppresses background. |
| **Bidirectional Cross-Attention** | At three scales (coarse to fine). **D→I:** depth is Query, image is Key/Value. **I→D:** the reverse. Uses fixed sinusoidal 2D positional encoding. Both directions are summed. |
| **PixelShuffle Decoder** | Cascaded Conv → PixelShuffle ×2 → BatchNorm → ReLU. Avoids checkerboard artefacts of transposed convolutions. Preserves sharp building edges. |
| **Light Transformer** | LightCrossAttention → Norm → FFN → Dropout for refined feature aggregation. |

<p align="center">
  <img src="Media/convolutional block attention moduel.png" alt="CBAM Block" width="65%"/>
</p>
<p align="center"><i>CBAM: Channel attention followed by spatial attention, applied at each encoder stage.</i></p>

#### Sub-Block Details

| Sub-Block | Structure |
|---|---|
| **PixelShuffle Upsampler** | Conv N→2N → PixelShuffle ×2 → BN → ReLU |
| **Light Transformer** | LightCrossAttention → Norm → FFN → Dropout |
| **LightCrossAttention** | Q/K/V projections → scaled attention → softmax → dropout → output projection + residual |
| **CBAM** | Channel: MaxPool + AvgPool → shared MLP → sigmoid → multiply. Spatial: Conv → sigmoid → multiply |

#### Training Recipe

| Parameter | Value |
|---|---|
| **Loss** | L1 |
| **Optimiser** | Adam, lr = 1e-5 |
| **Batch size** | 8 |
| **Epochs** | 100 |
| **Hardware** | Single RTX 4080-class GPU |
| **Labels** | nDSM stored as nDSM × 500 in uint16 |
| **Inputs** | Image + frozen-DA2 depth map (no DA2 weights updated) |

#### Model Specifications

| Metric | Value |
|---|---|
| **Parameters** | ~5.37 M |
| **FLOPs** | ~54.4 G |
| **Inference Speed** | ~44.6 FPS at 1024×1024 |
| **Size Comparison** | ~10× fewer FLOPs than heavier baselines |

#### Why It Fits Zenith

- **Small and fast** — suits a desktop app on CPU or light GPU
- **Depth prior matters** — removing it roughly doubles the error (1.54 → 3.04 m)
- **Robust to radiometric shift** — the depth prior is less sensitive to raw image intensity than image-only networks
- **Validated on urban/suburban** — terrain is handled by the SRTM branch (§ Terrain Branch)

---

### Terrain Branch

The rDSM already captures the land's shape; it only needs units. A linear map is fitted to SRTM:

$$D_{abs}(x,y) = a \cdot \hat{D}(x,y) + b$$

- **a** converts relative units to metres (scale); **b** shifts to the correct elevation (offset)
- Estimated by **robust regression** (Huber or RANSAC) — errors in either source won't skew the fit
- **Correlation sign check** — if rDSM is inverted relative to SRTM, *a* simply becomes negative

### Final Merge

$$DSM(x,y) = P(x,y) \cdot DSM_{urban}(x,y) + (1 - P(x,y)) \cdot DSM_{terrain}(x,y)$$

Soft probabilities (not a hard switch) avoid visible seams. Overlapping patches are averaged with feathered, distance-weighted windows. Output: a georeferenced GeoTIFF DSM in metres above sea level.

---

## 📊 Results & Validation

### Model Metrics

| Metric | Value | Dataset |
|---|---|---|
| **MAE** | **1.54 m** | DFC 2019 (Jacksonville + Omaha) |
| **RMSE** | **1.89 m** | DFC 2019 / US3D |

<p align="center">
  <img src="Media/result comaparison.png" alt="Result Comparison" width="75%"/>
</p>
<p align="center"><i>Visual comparison across Jacksonville (JAX) and Omaha (OMA) test scenes. Row 1: Orthophoto. Row 2: Ground Truth nDSM. Row 3: Our Model (Metric Calibrator). Row 4: MLP Baseline. Row 5: IM2ELEVATION. Our model produces the closest match to ground truth with sharp building boundaries.</i></p>

### Training & Evaluation Dataset

| Dataset | Sensor | Resolution | Patch Size | Train / Test |
|---|---|---|---|---|
| **DFC2019-Track1** (Jacksonville, Omaha) | WorldView-3 | 0.35 m | 1024² | 2226 / 557 |

### In-Domain Results (MAE in metres) — Best in **bold**

| Model | Params | FLOPs (G) | FPS (1024²) | DFC19 |
|---|---|---|---|---|
| Baseline-MLP | 10 K | 5.4 | 216 | 5.11 |
| IM2HEIGHT | 7.4 M | 502 | 18.7 | 3.08 |
| SynRS3D | 366.5 M | 586 | 2.7 | 4.46 |
| NRF | – | – | – | 3.93 |
| IM2ELEVATION | 158.9 M | 535 | 5.3 | 1.67 |
| **Metric Calibrator (Ours)** | **5.37 M** | **54.4** | **44.6** | **1.54** |

> **Our Metric Calibrator achieves the best MAE on DFC2019** while being ~10× more efficient than heavier baselines.

### Ablation Study (MAE in metres, DFC2019)

| Variant | Params | DFC19 |
|---|---|---|
| **Full model** | 5.37 M | **1.54** |
| w/o MobileViT | 0.43 M | 2.38 |
| w/o CBAM | 5.37 M | 1.74 |
| w/o Bi-Cross-Attention | 5.36 M | 2.00 |
| w/o PixelShuffle | 5.24 M | 1.90 |
| **w/o Depth prior** | 5.37 M | **3.04** |

**Key Takeaways:**
- Removing the **depth prior** nearly doubles the error (1.54 → 3.04 m) — confirming its critical role
- Removing **MobileViT** causes the next-largest degradation (→ 2.38 m)
- Removing **PixelShuffle** blurs building edges; removing **CBAM** causes over-prediction
- The full model is the best overall configuration

---

## 🎮 Stage 4 — 3D Reconstruction & Rendering Modes

### Mode 1 — Point Cloud

- Each DSM pixel is back-projected to a 3D point (x, y, h) with RGB colour
- Built with **Open3D / LAStools**
- Fastest, most faithful view of the raw prediction

### Mode 2 — 3D Mesh

<p align="center">
  <img src="Media/3D mesh flowchart.png" alt="3D Mesh Pipeline" width="100%"/>
</p>

The mesh pipeline converts a point cloud into a **watertight, artifact-free, textured 3D model** in three modules:

```
Raw Point Cloud P ⊂ ℝ³ (+ colours)
   │
   ▼  Module 1 · mesh_using_triangulation.py
   │   Height-axis detection → O(N) grid binning → EDT void inpainting
   │   → quad triangulation → stretched-edge pruning
   ▼  Raw surface mesh
   │
   ▼  Module 2 · mesh_postprocess.py
   │   Boundary/steep edge detection → KD-tree local drop height
   │   → shared-vertex curtain-wall extrusion → Laplacian smoothing
   ▼  Repaired watertight mesh
   │
   ▼  Module 3 · mesh_texture_projection.py
   │   Orthographic UV mapping → Lanczos-3 resampling
   │   → hybrid multi-primitive assembly (top texture + wall colours)
   ▼  Final textured GLB
```

#### Module 1 — 2.5D Grid Triangulation
- **Height-axis auto-detection:** The axis with the smallest bounding-box extent is identified as the height axis
- **O(N) direct binning** instead of O(N log N) Delaunay — points quantised onto a uniform R×R grid
- **Void inpainting** via exact Euclidean Distance Transform — empty cells take the height/colour of their nearest valid neighbour
- **Quad triangulation** — each grid quad split into two triangles
- **Stretched-edge rejection** — triangles spanning empty regions are removed

#### Module 2 — Post-Processing & Curtain Walls
- **Hash-based boundary edge detection** using bijective 64-bit hashing
- **Steep discontinuity detection** for building edges
- **Local drop-target estimation** via KD-tree ball query (avoids deep skirts)
- **Shared-vertex curtain-wall extrusion** — no cracks or T-junctions
- **Boundary-constrained Laplacian smoothing** — only wall vertices are smoothed

#### Module 3 — Planar Texture Projection
- **Orthographic nadir projection** with affine UV normalisation
- **Lanczos-3 resampling** when source exceeds GPU budget
- **Hybrid multi-primitive composition** — surface faces get UV-mapped texture; vertical walls get per-vertex colours (solves texture smearing on walls)

### Mode 3 — Object Segmentation

Bounding boxes are drawn around every **building, tree, road and structure**. The purpose is analysis and detection — users can spot even the smallest objects that a human eye could easily miss.

- Supports **surface graph networks** (MeshCNN, DiffusionNet), **surface point sampling** (PointNet++, KPConv) and **multi-view 2D lifting** (SAM, Mask2Former)
- **Graph-cut / CRF** post-processing snaps label boundaries to real geometric edges
- Per-instance components grouped and enclosed in bounding boxes

---

## 🖥️ Interactive Viewer & User Experience

Rendering is done in **Three.js WebGL2** inside the Tauri window.

### Navigation
- **Orbit** — free rotation around the scene
- **Flythrough** — smooth cinematic movement
- **First-Person / Street View** — WASD + mouse, ground-level exploration

### Analysis Tools
- Height and slope inspection of structures
- Metric scale display for GeoTIFF inputs
- Sun lighting control
- Relief control panel (vertical exaggeration)
- Depth-map viewer and image comparison viewer

### Workflow UI
- Drag-and-drop image dropzone and gallery
- Generate button with progress display and history panel
- Batch queue bar for multiple images
- Model Manager for downloading / selecting inference models
- Settings, shortcuts and about dialogs
- System stats panel (GPU, CPU, RAM, VRAM monitoring)
- Render-mode tabs: **Point Cloud · 3D Terrain · Object Segmentation**

---

## 📦 Export Formats

| Format | Use |
|---|---|
| **OBJ** | Universal mesh interchange |
| **GLTF / GLB** | Web/real-time, carries textures and multiple primitives |
| **STL** | 3D printing / CAD |
| **PLY** | Point clouds and meshes with colour |
| **GeoTIFF** | Elevation output for GIS |

---

## ⚙️ Technology Stack

| Layer | Technology | Role |
|---|---|---|
| Desktop shell | **Tauri v2 + Rust** | Lightweight native container & IPC bridge |
| Frontend UI | **React 19 + Zustand** | Component layer & state management |
| 3D viewer | **Three.js WebGL2** | Point cloud, mesh, segmentation overlay rendering |
| Relative depth prior | **Depth Anything V2** (pretrained, frozen) | rDSM: shape without units |
| Metric height model | **Metric Calibrator** (custom, PyTorch) | Image + rDSM → metric nDSM |
| Router | **NumPy, SciPy, scikit-image** | Five scale-free features, P_urban |
| Elevation reference | **SRTM** via Python `elevation` | Ground height and real-world scale |
| Calibration | **scikit-learn** (Huber/RANSAC) | Robust fit in terrain branch |
| Geospatial I/O | **rasterio / pyproj / GDAL** | GeoTIFF parsing, CRS/GSD, reprojection |
| Point cloud | **Open3D / LAStools** | 3D reconstruction from depth |
| Meshing | Custom grid triangulation + curtain walls | Surface mesh from point cloud |
| Texture | Orthographic UV + Lanczos-3 | Photorealistic projection |
| Segmentation | Mesh/point/multi-view + CRF | Object labels & bounding boxes |
| Model weights | **safetensors / .pth** | Local weight cache |

---

## 🌍 Impact & Benefits

### Speed & Cost

| Metric | Zenith | Traditional Methods |
|---|---|---|
| **Speed** | Minutes | Months |
| **Cost** | Near-zero | Millions (LiDAR surveys, flights) |
| **Comparison** | **~4000× faster** | — |
| **Cost Comparison** | **~10,000,000× cheaper** | — |

> *A 3D model of a city. Minutes, not months. Costs less than a packet of chips. That's Zenith.*

### Application Domains

| Domain | Use Cases |
|---|---|
| 🏗️ **Urban Planning** | Building-height analysis, 3D city modelling, infrastructure planning, smart city & zoning |
| 🌋 **Disaster Management** | Rapid terrain assessment, hazard analysis (landslide, wildfire), evacuation route planning, post-disaster mapping |
| 🌾 **Land Management** | Terrain and drainage analysis, land-suitability assessment, vegetation & landscape analysis, agriculture & watershed planning |
| 🎖️ **Military Reconnaissance** | Rapid terrain visualization, elevation & slope analysis, remote-area situational awareness, route & accessibility assessment |

### Broader Impact

| Dimension | Impact |
|---|---|
| **Social** | Saving lives — faster situational awareness enables timely disaster response decisions |
| **Economic** | Much faster and cheaper than LiDAR, stereo imagery or dedicated sensors |
| **Environmental** | Reduces need for airborne LiDAR surveys (planes → carbon emissions) |
| **Climate** | Strengthens disaster preparedness and climate-resilience through accurate elevation analysis |

---

## 🧩 Design Decisions

| Decision | Reason |
|---|---|
| Same DA2 prior for both input types | Simplifies the pipeline; single pretrained DA2 serves all |
| Automatic file-type detection | Removes user error; single upload entry point |
| Frozen DA2; learning confined to a small calibrator | DA2 provides sharp structure. A ~5M-parameter calibrator is cheap to train and fast at inference |
| Dual-stream calibrator with cross-attention | Depth prior + image context; removing depth prior doubles MAE |
| PixelShuffle decoder | Avoids checkerboard artefacts of transposed convolutions |
| Router instead of single calibration | Metric Calibrator is urban-only; SRTM is blind to buildings — neither alone is good everywhere |
| Label-free routing with fixed reference stats | No labelled data needed; fixed μ/σ keep score absolute |
| Soft blending by P_urban | Avoids seams at hill-town boundaries |
| Grid binning for meshing | O(N) vs. O(N log N) on millions of points |
| Local drop heights for curtain walls | Avoids deep skirts clipping uneven terrain |
| Hybrid texture + vertex-colour mesh | Planar projection collapses vertical walls in UV space |
| Tauri instead of Electron | No bundled Chromium; smaller binary; lighter resource use |

---

## 🔮 Future Scope

- 📦 **Standalone `.exe` build** — one distributable executable, no Node.js/Rust/pnpm required
- 🐍 **Python + Tauri integration** — bundle Python inference inside the Tauri runtime
- 🎯 **Segmentation refinement** — broader class coverage and accuracy tuning
- 📏 **Routed-pipeline benchmark** — full Absolute DSM error split by urban/terrain against LiDAR
- 📍 **Ground Control Point calibration** — optional additional anchor for georeferenced inputs
- 🤖 **Supervised router** — with ~50 labelled patches, upgrade from equal weights to logistic regression
- 🌿 **Vegetation awareness** — NDVI / green-band check to separate trees from buildings
- 🗺️ **Router heat-map overlay** — show P_urban in the viewer for transparency
- 📊 **In-app validation tools** — error maps and per-terrain statistics against reference DSMs

---

## 👥 Team

**Team Oculus** — Smart India Hackathon 2026

| Role | Name | Branch & Batch | Contribution |
|---|---|---|---|
| 🎯 **Leader** | Disha Shukla | BT 2028 | Lead, ML research & experimentation |
| 👨‍💻 Member | Gaurav Neupane | CSE 2027 | Lead, desktop application (Tauri and Rust) |
| 🔬 Member | Sidhanth Mandal | CSE 2027 | ML model implementation — object segmentation and 3D rendering |
| 🎨 Member | Omkar Mahabole | CSE 2027 | UI/UX design and video editing |
| 💻 Member | Ujjawal Kumar | CSE 2027 | Frontend development — UI designs to TypeScript |
| 🌐 Member | Mehak Rana | BT 2028 | 3D rendering research — CesiumJS and Three.js integration |

---

## 📚 References

### Research Papers
1. Disentangling Geometry and Texture: RGB–DSM Pre-training for 3D Urban Understanding
2. Sat3R — Satellite 3D Reconstruction
3. Sat-DN — Satellite Depth Estimation
4. IM2ELEVATION — Single images for building height estimation
5. **RDAH-Net** — Relative depth to absolute height mapping (Jiang et al., *Remote Sensing* 2026) — *Adapted and implemented as our Metric Calibrator*
6. **Depth Anything V2** — Strong relative-depth estimation — *Implemented as DA2 depth backbone*
7. Depth Anything V3 — Newer monocular depth capabilities
8. SAM 3 — Segment Anything with Concepts

### Datasets
- [DFC 2019 / US3D](https://github.com) — Urban Semantic 3D Dataset
- GAMUS Dataset
- Swiss & Hong Kong (self-built from open LiDAR + Gaofen-7)

### Tools & Frameworks
- [SRTM](https://www2.jpl.nasa.gov/srtm/) — Shuttle Radar Topography Mission elevation data
- [Copernicus DEM](https://www.copernicus.eu/) — Alternative elevation source
- [Tauri](https://tauri.app/) — Lightweight desktop application framework
- [Elevation (Python)](https://pypi.org/project/elevation/) — SRTM data access
- [Trimesh](https://trimesh.org/) — 3D mesh processing

---

<p align="center">
  <b>Built with ❤️ by Team Oculus for Smart India Hackathon 2026</b>
</p>


