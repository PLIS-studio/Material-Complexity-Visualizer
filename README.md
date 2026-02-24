# Material Complexity Visualizer

**Real-time shader complexity analysis for the Unreal Engine Material Editor**

Material Complexity Visualizer (MCV) helps developers identify and understand shader complexity hotspots directly inside the Material Editor — at the wire and node level — without needing RenderDoc, shader compilation logs, or GPU profilers.

---

## Overview

MCV colors every wire in your material graph with a heatmap gradient (white → violet → magenta → red) that represents estimated shader cost. Cheap connections stay white; expensive ones turn red. Cost numbers are printed directly on wires, and a floating **Legend** shows the current scale along with **Hotspots** — a ranked list of the most expensive nodes.

### Key Features

- **Heatmap Wires** — every connection colored by estimated cost
- **Cost Labels** — numeric values displayed directly on wires
- **6 Analysis Modes** — Total, ALU, Samples, Dependent, Flow, Feature Penalty
- **3 Normalization Modes** — Percentile 95, Global Max, Absolute (fixed budgets)
- **Display Toggle** — per-tab Wires and Legend on/off switches in the dropdown
- **Interactive Tooltip** — hover any wire for a full cost breakdown
- **Legend with Hotspots** — color scale + ranked expensive nodes with click-to-navigate
- **Configurable Top N** — show Top 5, 10, 20, or 50 hotspots; cycle with one click
- **Material Function Support** — full inline traversal with correct cost propagation
- **Customizable** — gradient colors, metric weights, absolute budgets, label styling
- **Editor Preferences Defaults** — configure the initial state for each new tab
- **Non-Destructive** — never modifies your materials; editor-only, zero runtime overhead

---

## Supported Versions

| Unreal Engine | Status |
|:---:|:---:|
| 5.0 – 5.7 | ✅ Supported |

**Platform:** Windows 64-bit (Editor only)

> ⚠️ **Substrate Materials:** MCV runs with Substrate enabled but results are not reliable. When working on Substrate materials, hide the visualization instead of disabling the plugin: open the MCV dropdown and set **Wires = off** and **Show Legend = off**. Re-enable instantly when switching back to standard materials.

---

## Quick Start

1. Install the plugin from [Fab Marketplace](https://www.fab.com/) or copy to `YourProject/Plugins/`
2. Open any Material in the Material Editor
3. Click the **MCV** button in the toolbar
4. Select a **Mode** (start with Total) and **Normalization** (P95 recommended)
5. Follow the heatmap — red wires are expensive
6. Press **Ctrl+L** to show the Legend with Hotspots
7. Click a Hotspot row to jump directly to that node

---

## Documentation

| Document | Description |
|----------|-------------|
| **[Getting Started](Documentation/GettingStarted.md)** | Installation, first analysis, modes overview |
| **[Usage Guide](Documentation/Usage.md)** | Complete reference for all features, settings, workflows |
| **[Troubleshooting](Documentation/Troubleshooting.md)** | Solutions for common issues |

---

## Analysis Modes

| Mode | What It Measures |
|------|-----------------|
| **Total** | Weighted combination of all metrics |
| **ALU** | Arithmetic/math operations |
| **Samples** | Texture sampling operations |
| **Dependent** | Dependent texture reads (UV computation before sampling) |
| **Flow** | Control flow complexity (switches, branches) |
| **Feature Penalty** | Material feature overhead (translucency, refraction, etc.) |

---

## Normalization Modes

| Mode | Description |
|------|-------------|
| **P95** | Scale based on 95th percentile — recommended for most materials |
| **Max** | Scale based on maximum wire cost |
| **Absolute** | Fixed budgets per mode — for cross-material comparison |

Default absolute budgets:

| Mode | Budget |
|:---:|:---:|
| Total | 300 |
| ALU | 250 |
| Samples | 64 |
| Flow | 80 |
| Dependent | 64 |
| Feature Penalty | 100 |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+L** | Toggle Legend |

---

## Settings

All settings are in **Editor Preferences → Plugins → Material Complexity Visualizer**:

- **Default State** — initial View Mode, Normalization, Wires, Legend visibility, and Hotspots Top N for each new tab
- Wire label font size and shadow
- Gradient colors and presets (Neon, Base, Custom)
- Metric weights for Total mode
- Absolute scale budgets

> **Note:** Default State settings apply only when a new Material Editor tab is opened. Already-open tabs keep their per-tab state independently.

---

## Support

- **Documentation:** See [Getting Started](Documentation/GettingStarted.md), [Usage](Documentation/Usage.md), [Troubleshooting](Documentation/Troubleshooting.md)

---

## License

Distributed through [Fab Marketplace](https://www.fab.com/) under the standard Epic Games Marketplace license.
