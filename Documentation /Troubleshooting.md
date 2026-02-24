# Troubleshooting

Solutions for common issues with Material Complexity Visualizer.

---

## Table of Contents

- [Installation Issues](#installation-issues)
- [Wires Not Colored](#wires-not-colored)
  - [No wire colors on materials opened at startup](#no-wire-colors-on-materials-opened-at-startup)
- [Colors Look Wrong](#colors-look-wrong)
- [Tooltip Issues](#tooltip-issues)
- [Legend Issues](#legend-issues)
- [Hotspots Issues](#hotspots-issues)
- [Performance Issues](#performance-issues)
- [Material Function Issues](#material-function-issues)
  - [LandscapeLayerBlend cost drops after Break → Make passthrough](#landscapelayerblend-cost-drops-after-break--make-passthrough)
- [Settings Issues](#settings-issues)
- [Build / Compilation Issues](#build--compilation-issues)
- [UE Version Compatibility](#ue-version-compatibility)
- [Plugin Compatibility](#plugin-compatibility)
- [Debug Tools](#debug-tools)

---

## Installation Issues

### Plugin not visible in Edit → Plugins

**Symptoms:** The plugin doesn't appear in the Plugins window.

**Solutions:**
1. Verify the folder structure is correct:
   ```
   YourProject/Plugins/MaterialComplexityVisualizer/MaterialComplexityVisualizer.uplugin
   ```
2. Make sure the `.uplugin` file exists at the root of the plugin folder
3. Restart the editor completely

### MCV button not appearing in Material Editor toolbar

**Symptoms:** Plugin is enabled but no MCV button in the Material Editor.

**Solutions:**
1. Close and reopen the Material Editor
2. Check if the plugin is enabled: **Edit → Plugins → Material Complexity Visualizer**
3. Restart the editor after enabling
4. If using source build: verify all three modules compiled without errors (MaterialComplexityCore, MaterialComplexityEditor, MaterialComplexityBatch)

---

## Wires Not Colored

### All wires are white

**Symptoms:** The visualizer is active but all wires appear white.

**Possible causes and solutions:**

1. **Wires toggle is off.** Check the MCV dropdown → Wires section. If Wires are disabled, coloring won't apply. Enable the toggle.

2. **Low cost material.** If every wire has very low cost, they all appear white. Switch to **Absolute** normalization to use a fixed scale.

3. **Wrong mode for the material.** A material with no texture samples will show all-white in Samples mode. Switch to a mode that matches the material's content.

4. **Normalization mismatch.** In **P95** or **Max** normalization, if one wire is vastly more expensive than all others, the rest appear white by comparison. Try **Absolute** normalization.

### Wires are colored but don't change when switching modes

**Symptoms:** Switching from Total to ALU shows the same colors.

**Solutions:**
1. Make sure you are clicking the mode option in the dropdown (the radio button should change)
2. Close and reopen the Material Editor tab
3. Check the Legend — it should update to show the new mode name, scale, and Hotspots

### No wire colors on materials opened at startup

**Symptoms:** After restarting the editor, materials that were previously open (auto-restored by UE) show white wires, no Legend, and no Hotspots. Switching to the tab or interacting with the graph makes colors appear.

**Root cause:** Two separate bugs combined to cause this:

1. The DrawingPolicy skipped its initial cost calculation for tabs that hadn't been focused yet — triggering calculation only when a tab is active or the user changes a setting.
2. On UE 5.6+, auto-restored background tabs don't get the MCV node factory injected (only the foreground tab does during startup).

**This is fixed in the current plugin version** (`362aa47` + `b47ff2b`). If you still see it:

1. Wait 1–2 seconds after the editor finishes loading — the background factory injection ticker runs for 10 seconds after startup
2. Verify you have the latest plugin version installed
3. If it persists, close and reopen the affected Material Editor tab as a workaround

---

### Wire with highest cost is not red

**Symptoms:** A wire has the maximum cost in the material but appears white or light colored.

**Solutions:**

1. **Check normalization mode.** In Absolute mode, if the budget is much higher than the actual costs, nothing will reach red. Lower the Absolute Scale budget in settings.

2. **Mode not matching.** Ensure the mode matches what you're looking at. A wire expensive in ALU mode may be cheap in Samples mode.

3. **P95 outlier.** In P95 mode, values above the 95th percentile are clamped to red. If your expensive wire *is* the 95th percentile, it should be red. Verify by hovering to see the actual cost in the tooltip.

---

## Colors Look Wrong

### Colors don't match the Legend gradient

**Symptoms:** A wire with cost at 50% of the scale doesn't appear to be the Legend's midpoint color.

**Explanation:** Wire colors use linear interpolation between the 5 gradient stops. The Legend renders the gradient as 18 discrete segments. Small visual differences between continuous (wires) and discrete (Legend) are expected.

### Gradient appears as a single solid color

**Symptoms:** The Legend or tooltip gradient bar shows one flat color.

**Solutions:**
1. Check that gradient colors aren't all set to the same value in settings
2. Reset gradient preset: set **Gradient Preset** to **Neon** or **Base**
3. Verify in **Editor Preferences → Plugins → Material Complexity Visualizer → Gradient** that all 5 colors are different

### Colors are too dark or invisible

**Symptoms:** Wire colors are barely visible against the graph background.

**Solutions:**
1. The default Neon gradient is designed for dark Material Editor backgrounds
2. If using a custom gradient, ensure Color at 0% is bright (white recommended)
3. Increase gradient saturation for intermediate stops

---

## Tooltip Issues

### Tooltip not appearing on hover

**Symptoms:** Hovering over wires doesn't show the cost tooltip.

**Solutions:**
1. Make sure MCV is active (MCV button in toolbar should be highlighted)
2. Make sure **Wires** are enabled in the MCV dropdown — if Wires are off, tooltips are also disabled
3. Hover directly over the wire, not near it — the hit detection requires the cursor to be on the wire spline
4. Hold the hover position steady for a moment — the tooltip has a brief appear delay (standard Slate tooltip behavior)
5. Try hovering over a different wire to verify

### Tooltip shows "0" for all metrics

**Symptoms:** Tooltip appears but all metric values are zero.

**Solutions:**
1. The wire might genuinely have zero cost (e.g., a direct connection from a constant to an output with no operations)
2. If all wires show zero, the cost calculator may not have processed the material yet — close and reopen the Material Editor tab

### Tooltip tick values don't match Legend

**Symptoms:** The tick labels under the tooltip gradient show different numbers than the Legend.

**Explanation:** Both the tooltip and Legend use the same ScaleMax value and the same formatting logic. If you see a discrepancy:
1. The tooltip may have been created before a mode/normalization change — move the cursor off and back onto the wire
2. Switch modes and switch back to force a refresh

---

## Legend Issues

### Legend not visible after opening a material

**Symptoms:** Legend doesn't appear when opening a material even though it was visible before.

**Explanation:** This is the expected behavior if **Legend Visible by Default** is set to `false` in Editor Preferences. When you open a new Material Editor tab, the Legend starts hidden.

**Solutions:**
1. Press **Ctrl+L** to show the Legend manually
2. Enable via MCV dropdown → Legend → Show Legend
3. To make the Legend show automatically: **Editor Preferences → Plugins → Material Complexity Visualizer → Default State → Legend Visible by Default** → enable

### Legend not visible

**Symptoms:** Pressed Ctrl+L but Legend doesn't appear.

**Solutions:**
1. The Legend appears in the **top-right corner** of the graph panel — it may be off-screen if the graph is scrolled
2. Make sure the Material Editor window is focused (not the Content Browser or Details panel)
3. Try toggling via the MCV dropdown → Legend → Show Legend
4. Check that MCV is active

### Legend shows wrong scale values

**Symptoms:** Legend tick labels don't match expected costs.

**Explanation:** This is usually correct behavior:
- In **P95** mode: the scale maximum is the 95th percentile of all wire costs, which changes per material
- In **Max** mode: the scale maximum is the single highest wire cost
- In **Absolute** mode: the scale is the fixed budget from settings

If values seem wrong, hover over wires to see individual costs and verify they match the gradient position.

### Legend position overlaps other UI

**Symptoms:** Legend panel covers important parts of the graph.

**Current behavior:** The Legend is fixed to the top-right corner with a 16px edge margin. It cannot be dragged or repositioned. If it obscures your work, toggle it off with Ctrl+L and use the tooltip for individual wire analysis.

---

## Hotspots Issues

### Hotspots list is empty

**Symptoms:** Legend's Hotspots section shows but with no entries.

**Possible causes:**
1. The material has very few nodes — Hotspots shows only nodes with positive incremental cost
2. All nodes have zero incremental cost
3. The current mode may not have cost data (e.g., Samples mode on a material with no textures)

### Clicking a Hotspot doesn't navigate to the node

**Symptoms:** Clicking a row in the Hotspots list doesn't pan the graph.

**Solutions:**
1. The node may already be visible in the current view
2. The node may be inside a Material Function (navigation focuses the outer function call node)
3. Try zooming out first, then clicking the Hotspot row

### Hotspot values seem wrong

**Symptoms:** A node you expect to be expensive doesn't appear in Hotspots.

**Explanation:** Hotspots uses **incremental scoring**, not cumulative cost. A node with high cumulative cost but low *added* cost (because its inputs are already expensive) will rank lower than a node that dramatically increases cost from its inputs.

For example:
- An `Add` node with two expensive inputs (cost 50 each) that outputs cost 51 has a delta of only 1
- A `Power` node with a cheap input (cost 5) that outputs cost 25 has a delta of 20
- The Power node ranks higher despite lower total cost

### Hotspots shows different Top N than expected

**Symptoms:** Hotspots shows 10 nodes but you expect 5 (or vice versa).

**Explanation:** The default Top N for new tabs is set in **Editor Preferences → Default State → Default Hotspots Top N**. Already-open tabs retain their current setting.

---

## Performance Issues

### Material Editor feels sluggish with MCV active

**Solutions:**
1. **Disable wire coloring**: Use the **Wires** toggle in the MCV dropdown — this is the fastest way to reduce MCV overhead while keeping the Legend and Hotspots available
2. **Hide labels**: Disable **Show Labels** in the MCV dropdown → Wires section
3. **Hide Legend**: Toggle off the Legend overlay (Ctrl+L) when not needed
4. **Close other Material Editors**: Each open Material Editor tab runs its own analysis

### Wire labels cause frame drops when zoomed in

**Solutions:**
1. Reduce **Wire Label Font Size** in settings (smaller fonts render faster)
2. Wire labels auto-hide when zoomed out — zooming out should improve performance
3. Disable labels entirely: MCV dropdown → Wires → uncheck Show Labels

---

## Material Function Issues

### Material Function wires are not colored

**Symptoms:** When editing a Material Function directly, wires don't show cost colors.

**Explanation:** MCV works within the context of a Material. When editing a standalone Material Function, there is no parent material to provide context for normalization. Open a Material that *uses* this function to see the costs in context.

### Function call node shows unexpected high cost

**Symptoms:** A MaterialFunctionCall node shows very high cost relative to surrounding nodes.

**Explanation:** This is expected — the function call's cost equals the **total internal cost** of all nodes inside the function. A function with 20 internal nodes will naturally have higher cost than a single-node expression.

To investigate, double-click the function call to open the function body and analyze internal costs there.

### LandscapeLayerBlend cost drops after Break → Make passthrough

**Symptoms:** A `LandscapeLayerBlend` output wire shows cost X (e.g., 1340). After splitting through `BreakMaterialAttributes` and recombining with `MakeMaterialAttributes`, the output wire shows noticeably lower cost (e.g., 900).

**Explanation:** This is a known limitation of MCV's per-attribute cost model, not a bug.

`LandscapeLayerBlend` blends N material layers using Height and Alpha scalar inputs as blending weights for each layer. These blending weight computations have real GPU cost — but they operate on scalars, not on material attribute channels. MCV tracks cost per material attribute (BaseColor, Normal, Roughness, etc.).

The blending weight overhead cannot be attributed to any single attribute channel — it is shared infrastructure that runs once regardless of how many attributes are reconnected. When `BreakMaterialAttributes` splits the output into per-attribute wires, it passes along only the per-attribute costs. The blending infrastructure overhead is not carried through.

**Is this wrong?** No. The per-attribute cost after `BreakMaterialAttributes` is correct for the attribute pipeline — it accurately represents what each channel costs to produce. The gap represents blending infrastructure shared across all outputs.

**Workaround:** To see the full cost including blending overhead, inspect the wire **before** `BreakMaterialAttributes` in the tooltip. The `LandscapeLayerBlend` output wire cost includes everything.

---

## Settings Issues

### Changing View Mode or Normalization in settings has no effect on open tabs

**Symptoms:** Changed **View Mode** or **Normalization Mode** in Editor Preferences but open Material Editor tabs don't update.

**Explanation:** This is the intended behavior. View Mode and Normalization Mode in Editor Preferences are **default state** settings — they apply only when a **new** tab is opened. Already-open tabs keep their own per-tab state.

**Solution:** Change the mode directly in the MCV dropdown for the relevant tab.

### Settings don't persist between sessions

**Symptoms:** Changed settings revert when restarting the editor.

**Solutions:**
1. Settings are saved to `EditorPerProjectUserSettings` config
2. Make sure you're modifying settings through **Editor Preferences** (not DefaultEditor.ini directly)
3. Verify the project's `Saved/Config/` directory is writable

### Can't find settings in Editor Preferences

**Path:** Edit → Editor Preferences → Plugins → Material Complexity Visualizer

If the section doesn't appear:
1. Confirm the plugin is enabled
2. Search for "Material Complexity" in the Editor Preferences search bar
3. Restart the editor

### Gradient doesn't reset to preset colors

**Symptoms:** Selecting "Neon" or "Base" preset doesn't change the colors.

**Solutions:**
1. Change the preset dropdown to a different value, then back to the desired preset
2. Close and reopen Editor Preferences
3. If colors are locked to Custom, manually set each color stop to match the preset values

---

## Build / Compilation Issues

### Module not found: MaterialComplexityCore

**Symptoms:** Build error referencing MaterialComplexityCore.

**Solutions:**
1. Verify `MaterialComplexityVisualizer.uplugin` lists all three modules
2. Check that `Source/MaterialComplexityCore/` directory and its `Build.cs` file exist
3. Regenerate project files

### Linking errors with MaterialComplexityCore

**Symptoms:** Unresolved external symbols from MaterialComplexityCore.

**Solutions:**
1. Add `MaterialComplexityCore` to `PublicDependencyModuleNames` in `MaterialComplexityEditor.Build.cs`
2. Make sure exported classes use the `MATERIALCOMPLEXITYCORE_API` macro
3. Clean and rebuild (delete `Intermediate/` and `Binaries/` folders)

### Hot Reload fails

**Symptoms:** Changes don't apply after live coding / hot reload.

**Solutions:**
1. Close all Material Editor windows before recompiling
2. Use full editor restart instead of hot reload for structural changes
3. If Live Coding fails repeatedly, disable it in **Editor Preferences → Live Coding** and use manual build

---

## UE Version Compatibility

### UE 5.0 / 5.1 specific issues

| Error | Solution |
|-------|----------|
| `MaterialDomain.h` not found | Expected — the plugin uses version-conditional includes |
| `FMaterialAttributeDefinitionMap` not found | Expected — plugin has built-in fallback for UE 5.0/5.1 |
| `ToPaintGeometry(FVector2f)` compile error | Plugin handles this via `MCV_VECTOR2_PAINT` macro |
| `RefractionMethod` / `RM_None` not found | Plugin wraps this in version check |

These are all handled automatically by the version compatibility layer. If you see these errors, ensure you're building the correct plugin version for your engine.

### UE 5.3+ specific issues

| Error | Solution |
|-------|----------|
| `.generated.h` errors | Ensure engine shaders and headers are fully installed |
| API deprecation warnings | Non-blocking warnings — safe to ignore |

### Substrate Materials (not supported)

**Symptoms:** Wires show no cost, all wires white, Hotspots list empty — on a material that looks complex.

**Explanation:** MCV runs with Substrate enabled but results are not reliable — Substrate uses a different node architecture that MCV was not designed for.

**Solution:** Hide the visualization instead of disabling the plugin. Open the MCV dropdown and set **Wires = off** and **Show Legend = off**. MCV stays loaded and you can re-enable it instantly when switching back to standard materials.

---

## Plugin Compatibility

### MCV conflicts with other plugins that modify Material Editor wire rendering

**Symptoms:**
- Wires show no MCV colors even though MCV is active and was working before
- Wire colors appear for a moment then revert to engine defaults
- Wires look broken (wrong colors, rendering artifacts) with both plugins active
- Editor crashes or becomes unstable when multiple visualization plugins are loaded

**Explanation:** MCV hooks into the Unreal Engine wire rendering system at the Material Editor graph panel level. On UE 5.6+, this uses a per-panel factory (`FGraphNodeFactory`) that can only hold one active factory at a time — whichever plugin injects last wins, replacing the previous one. On all UE versions, the shared `VisualPinConnectionFactories` static array is the injection point, and only the first matching factory is used.

Plugins that conflict with MCV include any plugin that:
- Colors or replaces wire rendering in the Material Editor graph
- Injects a custom drawing policy for Material Graph connections
- Modifies how the Material Graph visualizes node connections
- Adds its own heatmap, complexity overlay, or wire annotation to the graph

**First step when debugging visual issues:**

1. **Disable ALL other plugins that affect the Material Editor visual**
2. Restart the editor completely
3. Verify MCV works correctly with only MCV active
4. Re-enable other plugins one by one to identify the conflicting plugin

If a conflict is confirmed, MCV and the other plugin cannot be active simultaneously. Use whichever plugin is needed for the current task and disable the other.

---

## Debug Tools

### UE Console Commands

| Command | Purpose |
|---------|---------|
| `Slate.ShowDebugView 1` | Visualize widget boundaries (useful for Legend debugging) |
| `Slate.ShowInvalidationCaching 1` | Show which regions are being repainted |
| `mcv.Dependent.Debug 1` | Enable UV dependency chain debugging |

### Log Output

MCV logs appear with the `[MCV]` prefix:

```
Output Log → filter by "MCV"
```

Log file location:
```
YourProject/Saved/Logs/YourProject.log
```

### Widget Reflector

Use the **Widget Reflector** (Window → Developer Tools → Widget Reflector) to inspect Legend and Tooltip Slate widgets. Look for widgets named:
- `SMaterialComplexityLegend`
- `SMaterialComplexityHoverToolTip`
- `SMaterialComplexityHotspotsWidget`

---

## Still Having Issues?

If your problem isn't listed here:

1. Check the [GitHub Issues](../../issues) for known bugs and workarounds
2. Open a new issue with:
   - Unreal Engine version
   - Plugin version
   - Steps to reproduce the problem
   - Material Editor screenshot (if applicable)
   - Relevant log output from `Saved/Logs/`
