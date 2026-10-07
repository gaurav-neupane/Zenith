# Zenith

**Zenith** is a high-performance desktop workstation for 3D terrain reconstruction, dense point clouds, and geospatial elevation estimation from satellite and aerial optical imagery using local AI monocular depth estimation.

Built with **Tauri v2**, **Rust**, **React 19**, **TypeScript**, **Tailwind CSS v4**, **Zustand**, **Three.js WebGL2**, and a bundled **CPython 3.12** runtime for fully offline, private, and local execution.

---

## Architecture & Design

```text
                           ZENITH DESKTOP
                                 │
           ┌─────────────────────┴─────────────────────┐
           ▼                                           ▼
  React 19 Frontend                           Rust Backend (Tauri v2)
  (Three.js, WebGL2,                          (IPC, Process Manager,
   Zustand, Tailwind CSS)                      Storage, Integrity Verifier)
           │                                           │
           │                                           ▼
           │                             Bundled CPython 3.12 Subprocess
           │                             (%LOCALAPPDATA%/Zenith/runtime/)
           │                                           │
           │                                           ▼
           │                             PyTorch / Depth Anything V2 (ViT-G)
           │                             (CUDA GPU Acceleration / CPU Fallback)
           │                                           │
           └───────────────────┬───────────────────────┘
                               ▼
                Interactive 3D Viewport & Renderer
                - 3D Terrain Mesh (.glb)
                - High-Density Point Cloud (.ply / .glb)
```

- **Zero System Python Requirement**: Zenith bundles an isolated CPython 3.12 runtime. End users do not need Python, CUDA toolkits, or pip installed on their systems.
- **Isolated User Storage**: All AI environments, model weights, and inference caches reside in `%LOCALAPPDATA%/Zenith/` without modifying system `PATH` or global registry settings.
- **Subprocess IPC**: Tauri manages Python inference via isolated child subprocesses over standard JSON IPC with live progress streaming and instant cancellation support.

---

## Key Features

- **Depth Anything V2 Monocular AI Pipeline**:
  - Offline local inference using Depth Anything V2 Giant (`vitg` 1.3B parameters).
  - CUDA GPU acceleration with automatic CPU fallback.
  - Generates normalized depth maps (`depth.png`), high-fidelity point clouds (`terrain.ply`), and 3D terrain meshes (`terrain.glb`).
- **In-App Model Manager**:
  - One-click model acquisition with live bandwidth speed, ETA, and progress bar.
  - Streaming SHA-256 checksum verification and disk space pre-flight checks.
  - Automated dependency setup into an isolated local environment (`%LOCALAPPDATA%/Zenith/environment/`).
- **Split-View Optical & Depth Comparison**:
  - Interactive side-by-side split viewer between original optical imagery and AI depth maps.
  - Single-click **Maximize** full-screen preview on the toolbar for pixel-level inspection.
  - Automatic split-window reveal upon generation completion.
- **Interactive 3D Viewport**:
  - **3D Render**: Elevation-displaced 3D terrain mesh with wireframe toggle, vertical relief scaling, base floor thickness, and directional lighting.
  - **Point Cloud**: High-density 3D point cloud with real-time point radius/size slider.
- **Dedicated Render Workspace**:
  - Independent 3D file explorer supporting `.glb`, `.ply`, and `.obj` formats.
  - Automatic opening and rendering of newly generated 3D models when switching to the Render workspace.
  - Point radius slider support for `.glb` point clouds.
- **CAD-Inspired Dark Aesthetic**:
  - High-information-density dark palette (`#0b0d13`), custom frameless desktop titlebar, and keyboard shortcuts.
- **Multi-Format Export**:
  - Export reconstructed 3D assets to `.glb` and `.ply`.

---

## Desktop Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Space` / `Enter` | Run AI 3D Reconstruction |
| `R` | Reset 3D Viewport Camera |
| `G` | Toggle Reference Ground Grid |
| `W` | Toggle Wireframe Overlay (3D Render mode) |
| `F` | Toggle Viewport Fullscreen |
| `Ctrl + E` | Open 3D Asset Export Dialog |
| `Left Click + Drag` | Orbit 3D Camera |
| `Right Click + Drag` | Pan Viewport |
| `Scroll Wheel` | Zoom In / Out |

---

## Development Environment Setup

### Prerequisites

Ensure you have the following installed on your development machine:
1. **Node.js** (v18 or later) & **pnpm**:
   ```powershell
   npm install -g pnpm
   ```
2. **Rust & Cargo** (stable): Install via [rustup.rs](https://rustup.rs/).
3. **C++ Build Tools**: Visual Studio Build Tools with the **Desktop development with C++** workload (required for Tauri on Windows).

### Step 1: Clone the Repository

```bash
git clone https://github.com/CodeForFun7/Zenith.git
cd Zenith
```

### Step 2: Install Frontend Dependencies

```bash
pnpm install
```

### Step 3: Setup the Bundled Python 3.12 Runtime

Zenith runs Python via a bundled runtime located in `src-tauri/runtime/python`. Download and extract the standalone CPython 3.12 environment by running:

```powershell
pnpm setup:runtime
```

*(Alternatively, run the script directly:)*
```powershell
powershell -ExecutionPolicy Bypass -File scripts/setup_runtime.ps1
```

> **Note**: This is a one-time step for developers to initialize the local runtime. The runtime folder is gitignored to keep the repository lightweight.

### Step 4: Run in Development Mode

Launch the full Tauri desktop application with hot-reloading:

```bash
pnpm tauri dev
```

If you only want to work on the UI in a web browser without the desktop shell:
```bash
pnpm dev
```
*(Available at `http://localhost:1420`)*

---

## Building the Final Application (Production Release)

To package Zenith into a standalone Windows installer (`.exe` and `.msi`):

```bash
pnpm tauri build
```

### What This Does:
1. Compiles and minifies the React 19 / TypeScript frontend.
2. Compiles the Rust backend in release mode (`--release`) with full optimization.
3. Automatically bundles the Python 3.12 runtime (`src-tauri/runtime/python`), inference package, requirements, and model manifests directly into the installer (configured in `tauri.conf.json` under `"bundle" -> "resources"`).
4. Generates standalone installers in:
   ```text
   src-tauri/target/release/bundle/nsis/   (.exe installer)
   src-tauri/target/release/bundle/msi/    (.msi installer)
   ```

### End-User Experience
- End users simply download and run the installer.
- **No Python or dependencies are required on the user's machine.**
- The bundled Python runtime is extracted directly into `<Zenith Installation Directory>/runtime/python/` during installation and runs completely silently in the background.

---

## First Run & AI Model Setup

1. Launch Zenith.
2. Click **Models** in the top navigation bar to open the Model Manager.
3. Click **Get** next to **Depth Anything V2**:
   - Zenith checks the bundled Python runtime.
   - It sets up the isolated virtual environment in `%LOCALAPPDATA%/Zenith/environment/` and installs required packages (`torch`, `safetensors`, `opencv-python`, `numpy`, `Pillow`).
   - It downloads the 5.03 GB model weights to `%LOCALAPPDATA%/Zenith/models/` with real-time bandwidth and progress tracking.
   - It verifies the SHA-256 checksum integrity.
4. Once marked as **Ready**, import any satellite or aerial image, click **Generate**, and view your reconstructed 3D terrain and point cloud.

---

## Project Structure

```text
Zenith/
├── models/                     # Model manifests (depth-v2.json)
├── requirements/               # Python requirements (depth-v2.txt)
├── scripts/                    # Automation scripts (setup_runtime.ps1)
├── src/                        # React 19 Frontend
│   ├── components/
│   │   ├── generation/         # Loading modals & progress tracking
│   │   ├── image-input/        # Image dropzone, comparison & toolbar
│   │   ├── layout/             # Window titlebar, navigation, workspace
│   │   ├── models/             # Model manager modal & cards
│   │   ├── renderer/           # Dedicated 3D file explorer & workspace
│   │   └── viewer/             # Three.js viewport, controls & tabs
│   ├── services/               # Tauri IPC & image/model/generation services
│   ├── stores/                 # Zustand state stores (viewer, renderer, ui, etc.)
│   └── types/                  # TypeScript definitions
├── src-tauri/                  # Rust Backend & Tauri Config
│   ├── capabilities/           # Tauri v2 security capabilities
│   ├── icons/                  # Application icons
│   ├── python/                 # Python inference scripts & depth_anything_v2
│   ├── runtime/                # Bundled Python 3.12 runtime (gitignored)
│   ├── src/
│   │   ├── commands/           # Tauri IPC commands (model, inference, filesystem)
│   │   ├── models/             # Downloader, verifier, installer, registry
│   │   ├── python/             # Runtime resolution & subprocess runner
│   │   └── lib.rs              # App builder & plugin initialization
│   ├── Cargo.toml              # Rust crate dependencies
│   └── tauri.conf.json         # Tauri v2 bundle and window configuration
└── package.json                # Frontend scripts and dependencies
```

---

## License

This project is licensed under the MIT License.

