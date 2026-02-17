# Left Trackball Scroll Sensitivity Reduction Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Reduce left trackball scroll sensitivity to 50% of default by adding input processor configuration to the keymap.

**Architecture:** Add a device tree override at the end of `crosses.keymap` that configures the left trackball peripheral with a scroll scaler (1:4 ratio) in its input processing chain.

**Tech Stack:** ZMK firmware, Device Tree, GitHub Actions

---

## Task 1: Add Left Trackball Scroll Configuration

**Files:**
- Modify: `config/crosses.keymap:97` (end of file)

**Step 1: Add trackball scroll sensitivity override**

Add the following code at the end of `config/crosses.keymap` after the closing brace on line 97:

```devicetree

// Reduce left trackball scroll sensitivity by 50%
&trackball_peripheral_left {
  input-processors
    = <&zip_xy_to_scroll_mapper>
    , <&zip_scroll_scaler 1 4>  // 1:4 ratio = 50% reduction
    , <&zip_ble_report_rate_limit>;
};
```

Expected result: File now has trackball configuration at the end

**Step 2: Verify file syntax**

Run: `cat config/crosses.keymap | tail -15`

Expected output should show:
```
        };
    };
};

// Reduce left trackball scroll sensitivity by 50%
&trackball_peripheral_left {
  input-processors
    = <&zip_xy_to_scroll_mapper>
    , <&zip_scroll_scaler 1 4>  // 1:4 ratio = 50% reduction
    , <&zip_ble_report_rate_limit>;
};
```

**Step 3: Commit the change**

```bash
git add config/crosses.keymap
git commit -m "Reduce left trackball scroll sensitivity by 50%

Add input processor configuration to scale scroll values by 1:4 ratio,
reducing sensitivity to make scrolling more controllable.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

Expected: Clean commit with changes to crosses.keymap

---

## Task 2: Verify Build

**Files:**
- Monitor: `.github/workflows/build.yml` (GitHub Actions)

**Step 1: Push to trigger build**

```bash
git push
```

Expected: Push succeeds and triggers GitHub Actions workflow

**Step 2: Monitor GitHub Actions build**

Run: `gh run watch`

Expected: Build completes successfully with no devicetree errors

Alternative (if gh CLI not available):
- Visit GitHub Actions tab in repository
- Wait for build to complete
- Verify "build" workflow shows green checkmark

**Step 3: Download and inspect artifacts (optional verification)**

If build succeeds, artifacts should include:
- `crosses_54_left.uf2` - Contains the new scroll configuration
- `crosses_54_right.uf2` - Unchanged
- `settings_reset.uf2` - Unchanged

---

## Task 3: Update Keymap Visualization

**Files:**
- Modify: `keymap-drawer/crosses.yaml` (regenerated)
- Modify: `keymap-drawer/crosses.svg` (regenerated)

**Step 1: Parse keymap to YAML**

Run: `~/.local/bin/keymap parse -z config/crosses.keymap > keymap-drawer/crosses.yaml`

Expected: YAML file updated with current keymap state

**Step 2: Generate SVG visualization**

Run: `~/.local/bin/keymap draw -j config/info.json -l gggw_crosses_54_layout keymap_drawer.config.yaml keymap-drawer/crosses.yaml -o keymap-drawer/crosses.svg`

Expected: SVG file regenerated (note: visualization won't show scroll sensitivity, but keeps docs in sync)

**Step 3: Commit visualization updates**

```bash
git add keymap-drawer/crosses.yaml keymap-drawer/crosses.svg
git commit -m "Update keymap visualization

Regenerate after scroll sensitivity configuration change.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

Expected: Clean commit with visualization files

**Step 4: Push final changes**

```bash
git push
```

Expected: Push succeeds

---

## Troubleshooting

### If Build Fails with "unknown node referenced: trackball_peripheral_left"

**Diagnosis:** The device reference name doesn't match what's defined in the gggw-zmk-keebs shield.

**Resolution Steps:**

1. Check the shield definition repository for correct reference name
2. Common alternatives to try:
   - `&trackball_left`
   - `&trackball_peripheral`
   - `&pimoroni_trackball_left`
3. Update `config/crosses.keymap` with correct reference name
4. Commit and push again

### If Scroll Behavior Doesn't Change After Flashing

**Diagnosis:** May need to reset settings or verify correct firmware was flashed.

**Resolution Steps:**

1. Flash `settings_reset.uf2` to clear any cached settings
2. Re-flash `crosses_54_left.uf2`
3. Test scrolling behavior again
4. If still no change, verify the build artifacts actually include the new configuration

---

## Testing Checklist

After firmware is flashed to the keyboard:

- [ ] Left trackball scrolls at approximately 50% of previous speed
- [ ] Scrolling feels more controlled and less sensitive
- [ ] Smooth scrolling (CONFIG_ZMK_POINTING_SMOOTH_SCROLLING) still works
- [ ] BLE connection remains stable during scrolling
- [ ] No unexpected behavior on right half or other keyboard functions

---

## Completion Criteria

- ✅ trackball configuration added to `crosses.keymap`
- ✅ Firmware builds successfully on GitHub Actions
- ✅ Keymap visualization updated
- ✅ All changes committed and pushed
- ✅ Firmware ready to flash and test on hardware
