# Getting Started

Get up and running with Material Complexity Visualizer in under 5 minutes.

---

## Prerequisites

- **Unreal Engine** 5.0 – 5.7
- **Platform:** Windows 64-bit
- A project with at least one Material or Material Function

---

## Installation

### From Fab / Epic Marketplace

1. Purchase or add **Material Complexity Visualizer** from the [Fab Marketplace](https://www.fab.com/)
2. Open the **Epic Games Launcher** → Library → Vault
3. Click **Install to Engine** and select your UE version
4. Open your project — the plugin is enabled automatically

### Manual Installation

1. Download the plugin
2. Copy the `MaterialComplexityVisualizer` folder into your project's `Plugins` directory:
   ```
   YourProject/
   └── Plugins/
       └── MaterialComplexityVisualizer/
   ```
3. Launch the editor — the plugin is detected and loaded automatically

### Verify Installation

1. Open the editor
2. Go to **Edit → Plugins**
3. Search for "Material Complexity Visualizer"
4. Confirm the plugin is enabled (checkbox is checked)
5. Restart the editor if prompted

---

## Your First Analysis

### Step 1 — Open a Material

Open any Material or Material Function in the **Material Editor**. You'll notice a new **MCV** button in the toolbar.

### Step 2 — Activate the Visualizer

Click the **MCV** button in the toolbar. A dropdown menu appears with these sections:

- **Mode** — what aspect of shader cost to visualize
- **Normalization** — how cost values map to colors
- **Wires** — toggle wire coloring and cost labels on/off
- **Legend** — toggle the floating Legend panel

The default settings (**Total** mode + **P95** normalization, Wires enabled) are a great starting point.

### Step 3 — Read the Wires

Every wire in your material graph is now colored:

| Color | Meaning |
|:---:|:---|
| ⬜ White | Low cost — this connection is cheap |
| 🟣 Violet | Below average cost |
| 🟪 Magenta | Moderate cost — worth investigating |
| 🩷 Hot Pink | High cost — optimization candidate |
| 🟥 Red | Maximum cost — prioritize optimization here |

Numeric cost values appear directly on the wires when **Show Labels** is enabled.

### Step 4 — Hover for Details

Hover your mouse over any wire to see a **Tooltip** with:

- A mini heatbar showing where this wire falls on the cost scale
- Per-mode metric breakdown (ALU, Samples, Dependent, Flow, Feature Penalty)
- Source and destination node names
- A status tag (OK / Warm / Hot) with a hint about why the wire is expensive

### Step 5 — Show the Legend

Press **Ctrl+L** (or enable it from the MCV dropdown) to display the **Legend** in the top-right corner. The Legend shows:

- Current mode and normalization
- The full color gradient with 18 discrete segments
- Numeric tick labels that correspond to actual cost values
- **Hotspots** — a ranked list of the most expensive nodes, sorted by how much cost each node *adds* (incremental cost). Click any row to navigate directly to that node

---

## Understanding the Modes

### Total (Default)

A weighted combination of all metrics. Best for a general overview of material complexity.

The Total score is calculated as:

```
Total = ALU × W_alu + Samples × W_samp + Dependent × W_dep + Flow × W_flow + FeaturePenalty × W_fp
```

Default weights emphasize Dependent reads (×12) and Flow (×10) because these tend to have the highest GPU impact.

### ALU

Shows the cost of arithmetic and math operations — additions, multiplications, power, sine, lerp, dot products, etc. Use this mode when you suspect heavy math is slowing down your shader.

### Samples

Counts texture sampling operations. Each `TextureSample` node costs 1 base sample, but different sampler types have different weights. Use this to find materials with too many texture fetches.

### Dependent

Highlights **dependent texture reads** — situations where UV coordinates are calculated at runtime before being used to sample a texture. Dependent reads force the GPU to wait for the UV computation to finish before it can start fetching the texture, breaking parallelism. This is one of the most impactful optimization targets.

### Flow

Measures control flow complexity from static switches, if-statements, and feature permutations. High flow cost means the shader has many possible execution paths.

### Feature Penalty

Shows the overhead from material-level features like **Translucency**, **Subsurface Scattering**, **Refraction**, **Two-Sided** rendering, and others. These features add per-pixel cost regardless of graph complexity.

---

## Understanding Normalization

Normalization controls how raw cost numbers map to the white-to-red color gradient.

### Percentile 95 (P95) — Recommended

Uses the 95th percentile of all wire costs in the current material as the "red" end of the scale. This means 95% of wires fall within the gradient range, and only extreme outliers exceed it.

**Best for:** Most materials. Robust against single very-expensive wires that would otherwise compress the rest of the gradient into white.

### Global Max

Uses the maximum wire cost as the "red" end. Every wire fits within the gradient, but a single expensive wire can make everything else look white by comparison.

**Best for:** Materials where you want to see the full range including outliers.

### Absolute

Uses fixed per-mode budgets (configurable in settings). The scale doesn't change between materials, making it possible to compare complexity across different materials.

**Best for:** Cross-material comparison. If Material A and Material B both show orange wires in Absolute mode, they have similar cost levels.

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

## Default State Settings

All initial values when opening a new Material Editor tab are controlled in:

**Editor Preferences → Plugins → Material Complexity Visualizer → Default State**

| Setting | Description |
|---------|-------------|
| View Mode | Which mode the visualizer starts in for new tabs |
| Normalization Mode | Which normalization mode new tabs start with |
| Wires Enabled by Default | Whether wire coloring is on when you open a material |
| Legend Visible by Default | Whether the Legend panel is open by default |
| Default Hotspots Top N | How many nodes the Hotspots list shows (5/10/20/50) |

> **Note:** These settings apply only when a **new** Material Editor tab is opened. Changing them does not affect already-open tabs.

---

## Next Steps

- **[Usage Guide](Usage.md)** — deep dive into all features, settings, and workflows
- **[Troubleshooting](Troubleshooting.md)** — solutions for common issues

---

## Quick Settings Access

All plugin settings are in:

**Editor Preferences → Plugins → Material Complexity Visualizer**

Here you can change:
- Default view mode and normalization (per new tab)
- Wire label font size and shadow
- Gradient colors and presets
- Metric weights for Total mode
- Absolute scale budgets
