# Usage Guide

Complete reference for all Material Complexity Visualizer features, settings, and workflows.

---

## Table of Contents

- [MCV Toolbar Button](#mcv-toolbar-button)
- [Analysis Modes](#analysis-modes)
- [Normalization Modes](#normalization-modes)
- [Heatmap Wires](#heatmap-wires)
- [Wires Toggle](#wires-toggle)
- [Cost Labels](#cost-labels)
- [Tooltip](#tooltip)
- [Legend](#legend)
- [Hotspots](#hotspots)
- [Material Function Support](#material-function-support)
- [Settings Reference](#settings-reference)
- [Gradient Customization](#gradient-customization)
- [Optimization Workflows](#optimization-workflows)
- [Keyboard Shortcuts](#keyboard-shortcuts)

---

## MCV Toolbar Button

The **MCV** button appears in the Material Editor toolbar when you open any Material or Material Function.

Clicking the button opens a dropdown menu with the following sections:

| Section | Purpose |
|---------|---------|
| **Mode** | Select which cost metric to visualize (per-tab) |
| **Normalization** | Choose how raw values map to the color gradient (per-tab) |
| **Wires** | Toggle wire coloring and cost labels on/off for this tab |
| **Legend** | Toggle the Legend panel (includes color scale and Hotspots) |
| **Shortcuts** | Quick reference for keyboard shortcuts |

Mode and Normalization are **per-tab** — changing them in one Material Editor window does not affect other open windows.

---

## Analysis Modes

MCV provides six analysis modes. Each mode highlights a different dimension of shader cost on the material graph wires.

### Total

**What it measures:** Weighted combination of all five individual metrics.

**Formula:**
```
Total = ALU × W_alu + Samples × W_samp + Dependent × W_dep + Flow × W_flow + FeaturePenalty × W_fp
```

**Default weights:**

| Metric | Weight | Rationale |
|--------|:---:|-----------|
| ALU | 1 | Arithmetic is generally fast on modern GPUs |
| Samples | 8 | Texture fetches are memory-bound and costly |
| Dependent | 12 | Dependent reads break GPU parallelism — very expensive |
| Flow | 10 | Branching creates multiple shader permutations |
| Feature Penalty | 1 | Feature overhead is fixed, not cumulative |

**When to use:** Start here for a general overview. If you see expensive (red) wires, switch to individual modes to understand why.

### ALU

**What it measures:** Arithmetic and math operations — Add, Multiply, Divide, Power, Sine, Cosine, Lerp, Dot, Cross, Normalize, Fresnel, and others.

**Cost rules:**
- Basic operations (Add, Multiply, Subtract): 1 ALU each
- Intermediate operations (Lerp, Clamp, Saturate): 1–2 ALU
- Expensive operations (Power, Sine, Cosine, ArcTangent): 3–8 ALU
- Very expensive (Noise, Fresnel with complex falloff): 10+ ALU

**When to use:** When you suspect heavy math chains (e.g., procedural textures, complex Fresnel calculations, parallax mapping).

### Samples

**What it measures:** Texture sampling operations and their relative cost.

**Cost rules:**
- Standard `TextureSample`: 1 sample
- `TextureSampleParameter2D`: 1 sample
- Cubemap samples: higher weight due to trilinear filtering
- Virtual Texture samples: additional overhead

**When to use:** When a material uses many textures and you want to reduce texture fetch count. Mobile platforms are especially sensitive to sample count.

### Dependent

**What it measures:** Dependent texture reads — situations where UV coordinates are computed at runtime before sampling.

**Dependency levels:**

| Level | Meaning | Example |
|:---:|---------|---------|
| 0 | No dependency — UVs come from vertex data | Standard TexCoord → TextureSample |
| 1 | Simple offset — parameter-based UV offset | TexCoord + Param → TextureSample |
| 2 | Computed UVs — math on coordinates | TexCoord × Time → TextureSample |
| 3 | Cascading — dependent sample feeds another sample | TextureSample(computed UV) → UV for another sample |

**Why it matters:** Level 2–3 dependent reads force the GPU to serialize operations that would otherwise run in parallel. A single Level 3 chain can be more expensive than dozens of independent texture samples.

**When to use:** When materials feel slower than their node count suggests. Dependent reads are often the hidden bottleneck.

### Flow

**What it measures:** Control flow complexity from branching and permutations.

**What adds Flow cost:**
- `StaticSwitch` nodes — each switch doubles the potential permutation count
- `If` nodes (runtime branching) — the GPU evaluates both branches
- `StaticSwitchParameter` — creates shader variants
- Multiple switches in series multiply the permutation space

**When to use:** When a material has many static switches or conditional logic. High Flow cost means the shader compiler generates many variants, increasing compile time and instruction cache pressure.

### Feature Penalty

**What it measures:** Overhead from material-level feature flags.

**Penalized features:**

| Feature | Description |
|---------|-------------|
| Translucency | Multiple passes, blending overhead |
| Subsurface Scattering | Extra passes for SSS computation |
| Refraction | Screen-space distortion pass |
| Two-Sided | Double-sided rendering overhead |
| Tessellation / Displacement | Extra geometry stage |
| Custom depth / stencil | Additional render passes |

**When to use:** To understand the fixed cost of enabling material features. Unlike other modes, Feature Penalty shows *per-feature* overhead rather than cumulative graph complexity.

---

## Normalization Modes

Normalization defines how raw cost values are mapped to the 0%–100% color gradient range.

Normalization is **per-tab** — each Material Editor window remembers its own normalization setting.

### Percentile 95 (P95)

The scale maximum is set to the 95th percentile of all wire costs in the current material.

- 95% of wires fall within the gradient range
- Top 5% outliers may exceed the scale (shown as red)
- Provides excellent visual contrast for most materials
- **Recommended for daily use**

### Global Max

The scale maximum is the single highest wire cost in the material.

- Every wire fits within the gradient — no values exceed the scale
- A single very expensive wire can "compress" all other wires into white/violet range
- The last tick label on the gradient shows the exact max value

### Absolute

The scale maximum is a fixed budget per mode, defined in plugin settings.

- Scale does not change between materials
- Enables meaningful cross-material comparison
- Wires can exceed the budget (shown as saturated red)

**Default budgets:**

| Mode | Budget | Typical range |
|------|:---:|:---:|
| Total | 300 | 0–800+ |
| ALU | 250 | 0–400+ |
| Samples | 64 | 0–64+ |
| Flow | 80 | 0–200+ |
| Dependent | 64 | 0–64+ |
| Feature Penalty | 100 | 0–200+ |

All budgets are configurable in **Editor Preferences → Plugins → Material Complexity Visualizer → Advanced → Absolute Scale**.

---

## Heatmap Wires

When MCV is active and Wires are enabled, every wire in the material graph is colored based on the current mode and normalization.

### Color Gradient

The default **Neon** gradient uses 5 stops:

| Position | Color | Hex |
|:---:|:---:|:---:|
| 0% | White | `#FFFFFF` |
| 25% | Violet | `#BF7FFF` |
| 50% | Magenta | `#DE47CD` |
| 75% | Hot Pink | `#FF1E78` |
| 100% | Red | `#FF0000` |

Between stops, colors are interpolated linearly. On the Legend and Tooltip, the gradient is rendered as **18 discrete segments** for visual clarity.

### Alternative: Base Preset

| Position | Color | Hex |
|:---:|:---:|:---:|
| 0% | White | `#FFFFFF` |
| 25% | Green | `#00D000` |
| 50% | Yellow | `#FFD600` |
| 75% | Orange | `#FF6D00` |
| 100% | Red | `#D50000` |

Switch presets in **Editor Preferences → Plugins → Material Complexity Visualizer → Gradient → Gradient Preset**.

---

## Wires Toggle

The **Wires** section in the MCV dropdown provides a per-tab switch to disable wire coloring entirely.

### When to Use

- You want to inspect the graph layout without color distractions
- You only need the Legend and Hotspots, not individual wire costs
- Performance testing: verify that MCV has minimal overhead

### Behavior

- When **Wires OFF**: all wires revert to their default Unreal Engine colors; cost labels are hidden
- When **Wires ON**: full heatmap coloring and labels are applied
- The toggle is **per-tab** — disabling wires in one Material Editor does not affect others
- The **default state** for new tabs is controlled by **Editor Preferences → Default State → Wires Enabled by Default**

---

## Cost Labels

When **Show Labels** is enabled (in the Wires section of the dropdown), numeric cost values are printed directly on each wire.

### Behavior

- Labels show the raw cost value for the current mode
- Labels auto-hide when zoomed out far (text LOD)
- Labels use a double shadow (outer + inner) for readability on any wire color
- Font size is configurable (6–24pt, default: 9pt)

### Settings

| Setting | Default | Range | Path |
|---------|:---:|:---:|------|
| Show Labels | On | On/Off | MCV dropdown → Wires → Show Labels |
| Font Size | 9 | 6–24 | Material Complexity → Wire Labels → Wire Label Font Size |
| Shadow Opacity | 0.7 | 0.0–1.0 | Material Complexity → Wire Labels → Wire Label Shadow Opacity |

---

## Tooltip

Hover over any wire to see a detailed cost breakdown.

### Tooltip Sections

| Section | Content |
|---------|---------|
| **Header** | Wire label (From → To node names), mode badge |
| **Wire Score** | Mini heatbar (18 segments) showing position on gradient |
| **Tick Labels** | Numeric scale values matching the Legend |
| **Primary Value** | Cost value for current mode with formatted number |
| **Metrics Grid** | All 6 mode values for this wire (ALU, Samples, Dependent, Flow, Feature Penalty, Total) |
| **Context** | Source node, destination node, output pin name |
| **Hint** | Status tag (OK / Warm / Hot) with explanation |
| **Footer** | Current mode and normalization info |

### Hint Tags

| Tag | Color | Meaning |
|-----|:---:|---------|
| OK | 🟢 Green | Wire cost is within normal range |
| Warm | 🔵 Blue | Wire cost is elevated — worth noting |
| Hot | 🔴 Red | Wire cost is high — optimization candidate |

### Node Name Abbreviations

The tooltip uses short names for common node types:

| Full Name | Short | Full Name | Short |
|-----------|:---:|-----------|:---:|
| Multiply | Mult | TextureSample2D | Tex2D |
| Add | Add | TextureSample | TexSample |
| Subtract | Sub | Constant3Vector | Float3 |
| Divide | Div | Constant4Vector | Float4 |
| Power | Pow | ComponentMask | Mask |
| Lerp | Lerp | AppendVector | Append |
| Fresnel | Fresnel | DotProduct | Dot |
| Normalize | Normalize | CrossProduct | Cross |

Material Function calls display their function name. Function Output nodes display their `OutputName` (e.g., "Result", "Color", "Normal").

---

## Legend

The Legend is a floating panel in the top-right corner of the Material Editor graph. It contains the color scale and the Hotspots ranking.

### Toggle

- **Ctrl+L** keyboard shortcut
- **MCV dropdown → Legend → Show Legend** checkbox
- Default state: **Editor Preferences → Default State → Legend Visible by Default**

### Color Scale

| Element | Description |
|---------|-------------|
| **Header pill** | "MCV Legend" with colored dot indicator |
| **Mode subtitle** | Current mode name (e.g., "Mode: ALU") |
| **Metric section** | Current metric value and description |
| **Normalization section** | Current normalization mode and description |
| **Scale ramp** | 18 discrete color segments from gradient |
| **Tick labels** | 5 values at 0%, 25%, 50%, 75%, 100% of scale max |
| **Subnote** | "Discrete ramp - White → Magenta → Red" |

### Dynamic Tick Labels

Tick labels update automatically when the scale changes:

- For **P95/Max**: ticks show actual cost values based on the computed scale maximum
- For **Absolute**: ticks show fixed budget fractions
- The last tick always shows "X+" to indicate values can exceed the scale
- Small values (< 10) display one decimal place; larger values round to integers

**Example** (ALU mode, P95, ScaleMax = 80):
```
0    20    40    60    80+
```

**Example** (Samples mode, Absolute, budget = 16):
```
0    4    8    12    16+
```

---

## Hotspots

The Hotspots section appears below the color scale in the Legend. It shows a ranked list of the **Top N most expensive nodes**, sorted by incremental contribution.

### Scoring

Hotspots ranks nodes by **how much cost each node adds**, not by total cumulative cost. This ensures that simple aggregator nodes (Add, Lerp) don't dominate the list just because they collect many input branches.

**Incremental score formula:**

```
input_cost  = max(cost of each connected input pin)
delta       = max(0, output_cost - input_cost)
Cin         = max(input_cost, 1.0)
score       = delta × ln(output_cost / Cin)
```

- **delta** is displayed as the value (intuitive: "this node adds X cost")
- **score** is used for sorting (balances absolute and relative contribution)
- Both output and input costs use the current mode and weights

### Navigation

Click any row in the Hotspots list to **pan the graph** to that node and select it. This works across the full graph, including nodes that are currently off-screen.

### Mode Awareness

The Hotspots list updates when you switch modes. In ALU mode, it shows the top ALU contributors; in Samples mode, the top texture sampling nodes; and so on.

### Top N Setting

The number of nodes shown is configurable:
- **Dropdown** (per-tab): MCV button shows current count; cycle through 5/10/20/50 with the combobox
- **Default**: **Editor Preferences → Default State → Default Hotspots Top N**

---

## Material Function Support

MCV fully supports Material Functions with inline cost analysis.

### How It Works

When a `MaterialFunctionCall` node is encountered:

1. MCV traverses **into** the function body
2. All internal nodes are analyzed as if they were directly in the parent material
3. The function call's output cost equals the total internal cost
4. Nested functions (functions calling other functions) are resolved recursively

### Function Output Nodes

`FunctionOutput` nodes inside Material Functions display their `OutputName` property in the tooltip (e.g., "Result", "Color", "Normal"). If no name is set, it shows "Output".

### Considerations

- Material Functions are analyzed every time they're encountered (no cross-material caching)
- Very deeply nested function chains may accumulate significant cost
- The Legend's Hotspots section can point to nodes inside functions

---

## Settings Reference

All settings are located at: **Editor Preferences → Plugins → Material Complexity Visualizer**

### Default State

These values apply only when a **new** Material Editor tab is opened. Changing them does not affect already-open tabs.

| Setting | Type | Default | Description |
|---------|------|:---:|-------------|
| View Mode | Enum | Total | Analysis mode for new tabs |
| Normalization Mode | Enum | Percentile95 | Normalization for new tabs |
| Wires Enabled by Default | Bool | true | Whether wire coloring starts enabled |
| Legend Visible by Default | Bool | true | Whether Legend panel starts visible |
| Default Hotspots Top N | Enum | 10 | Hotspots count: 5, 10, 20, or 50 |

### Wire Labels

| Setting | Type | Default | Range | Description |
|---------|------|:---:|:---:|-------------|
| Show Labels | Bool | true | On/Off | Display cost numbers on wires |
| Wire Label Font Size | int | 9 | 6–24 | Font size in points |
| Wire Label Shadow Opacity | float | 0.7 | 0.0–1.0 | Shadow behind labels for readability |

### Advanced — Weights (Total Mode)

| Setting | Type | Default | Min | Description |
|---------|------|:---:|:---:|-------------|
| ALU Weight | int | 1 | 1 | How much ALU contributes to Total |
| Samples Weight | int | 8 | 1 | How much Samples contributes to Total |
| Dependent Weight | int | 12 | 1 | How much Dependent contributes to Total |
| Flow Weight | int | 10 | 1 | How much Flow contributes to Total |
| Feature Penalty Weight | int | 1 | 0 | How much Feature Penalty contributes to Total |

### Advanced — Absolute Scale Budgets

| Setting | Type | Default | Range | Description |
|---------|------|:---:|:---:|-------------|
| Total Scale | float | 300 | 1–5000 | Budget for Total mode |
| ALU Scale | float | 250 | 1–5000 | Budget for ALU mode |
| Samples Scale | float | 64 | 1–512 | Budget for Samples mode |
| Flow Scale | float | 80 | 1–2000 | Budget for Flow mode |
| Dependent Scale | float | 64 | 1–512 | Budget for Dependent mode |
| Feature Penalty Scale | float | 100 | 1–1000 | Budget for Feature Penalty mode |

### Gradient

| Setting | Type | Default | Description |
|---------|------|:---:|-------------|
| Gradient Preset | Enum | Neon | Preset selection (Neon / Base / Custom) |
| Color at 0% | LinearColor | White (#FFFFFF) | Lowest cost color |
| Color at 25% | LinearColor | Violet (#BF7FFF) | Low cost color |
| Color at 50% | LinearColor | Magenta (#DE47CD) | Moderate cost color |
| Color at 75% | LinearColor | Hot Pink (#FF1E78) | High cost color |
| Color at 100% | LinearColor | Red (#FF0000) | Maximum cost color |

> **Note:** Manually changing any gradient color automatically switches the preset to **Custom**.

---

## Gradient Customization

### Using Presets

1. Open **Editor Preferences → Plugins → Material Complexity Visualizer → Gradient**
2. Select **Gradient Preset**:
   - **Neon** — White → Violet → Magenta → Hot Pink → Red (default)
   - **Base** — White → Green → Yellow → Orange → Red (traditional heatmap)

### Custom Colors

1. Set **Gradient Preset** to any value
2. Modify individual color stops (Color at 0%, 25%, 50%, 75%, 100%)
3. The preset automatically switches to **Custom**
4. Changes apply immediately to all open Material Editors

### Tips for Custom Gradients

- Keep **Color at 0%** light (white or light gray) for readability on dark backgrounds
- Keep **Color at 100%** saturated and warm (red, orange) for visibility
- Ensure sufficient contrast between adjacent stops
- The gradient appears on wires, Legend, and Tooltip simultaneously

---

## Optimization Workflows

### Workflow 1: General Material Audit

1. Open the material in the Material Editor
2. Set mode to **Total**, normalization to **P95**
3. Press **Ctrl+L** to show the Legend
4. Scan the graph — red wires are the most expensive paths
5. Check the **Hotspots** section in the Legend to see ranked expensive nodes
6. Click a Hotspot row to jump to that node
7. Switch to individual modes (ALU, Samples, Dependent) to understand *why* it's expensive

### Workflow 2: Reduce Texture Samples

1. Set mode to **Samples**
2. Every wire shows how many texture fetches flow through it
3. Look for parallel chains of texture samples that could be packed into fewer textures
4. Check if any textures are sampled but their output is masked away

### Workflow 3: Find Dependent Reads

1. Set mode to **Dependent**
2. Red wires indicate UV computation chains feeding into texture samples
3. Hover a red wire — the tooltip shows the dependency level (0–3)
4. Level 2–3 wires are optimization priorities:
   - Can the UV computation be moved to vertex shader?
   - Can you replace computed UVs with a precomputed lookup texture?
   - Can you simplify the UV math chain?

### Workflow 4: Cross-Material Comparison

1. Set normalization to **Absolute**
2. Open Material A and note the general color range
3. Open Material B in another tab
4. Both materials use the same fixed scale — colors are directly comparable
5. A red wire in one material is exactly as expensive as a red wire in the other

### Workflow 5: Verify Optimization Results

1. Analyze the material **before** optimization — note the Legend's Hotspots rankings and scale
2. Make your changes
3. The visualization updates instantly — verify that:
   - The Legend's Hotspots no longer list the optimized nodes
   - The Legend's scale max has decreased
   - Previously red wires are now cooler colors

---

## Keyboard Shortcuts

| Shortcut | Action | Context |
|----------|--------|---------|
| **Ctrl+L** | Toggle Legend | Material Editor |

---

## Performance Notes

### Editor Impact

- MCV is **editor-only** — zero impact on packaged builds
- All three modules (Core, Editor, Batch) are excluded from shipping configurations
- Wire coloring and labels add minimal overhead to the Material Editor render loop

### Large Graphs

- Tested with materials containing 100–200+ nodes
- Cost values are **cached per (Expression, OutputIndex)** pair
- Cache invalidation occurs automatically on mode/normalization change or graph edit
- Wire labels auto-hide at far zoom levels (text LOD)
- Legend and Hotspots update at most 10 times per second (throttled)

### Memory

- No persistent memory allocation beyond cached analysis results
- Cache is cleared when closing a Material Editor tab
- Legend overlay is destroyed on tab close

### Wires Toggle for Performance

If the Material Editor feels sluggish with MCV active on a complex material:
1. Use the **Wires** toggle in the MCV dropdown to disable wire coloring
2. Keep the Legend and Hotspots for overview without per-wire overhead
