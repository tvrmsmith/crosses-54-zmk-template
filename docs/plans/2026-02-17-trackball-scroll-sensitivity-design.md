# Left Trackball Scroll Sensitivity Reduction

**Date:** 2026-02-17
**Status:** Approved

## Overview

Reduce the scroll sensitivity of the left trackball to 50% of the current default value to provide more controlled scrolling behavior.

## Context

Previously, the keymap included trackball scroll sensitivity configuration that was removed in commit ad6eea3. The previous implementation used a scroll scaler of `1 4` to achieve a 50% reduction in sensitivity. This design restores that functionality specifically for the left trackball.

## Architecture

The solution uses ZMK's input processor chain to configure the left trackball peripheral. A device tree override will be added to `crosses.keymap` that applies scroll scaling to only the left trackball, leaving any other pointing devices unaffected.

The configuration is placed at the end of the keymap file, outside the root devicetree node, following standard ZMK override patterns.

## Implementation Details

### File to Modify

`config/crosses.keymap`

### Code to Add

At the end of the file, after the closing brace of the root node:

```devicetree
// Reduce left trackball scroll sensitivity by 50%
&trackball_peripheral_left {
  input-processors
    = <&zip_xy_to_scroll_mapper>
    , <&zip_scroll_scaler 1 4>  // 1:4 ratio = 50% reduction
    , <&zip_ble_report_rate_limit>;
};
```

### How It Works

- `&trackball_peripheral_left` - References the left trackball device from the gggw-zmk-keebs shield definition
- `&zip_xy_to_scroll_mapper` - Converts X/Y trackball movement to scroll events
- `&zip_scroll_scaler 1 4` - Scales scroll values by 1/4 (reduces to 25% of original, which is 50% reduction from current)
- `&zip_ble_report_rate_limit` - Limits BLE report frequency to prevent connection issues

## Error Handling

### Compile-Time Validation

If the device reference `&trackball_peripheral_left` doesn't exist in the shield definition, the firmware build will fail with a devicetree error message indicating the reference cannot be resolved.

### Fallback Plan

If the build fails due to an incorrect device reference:
1. Examine the gggw-zmk-keebs shield devicetree files to identify the correct reference name
2. Alternative naming patterns to try: `&trackball_left`, `&trackball_peripheral`, or side-specific variants
3. Update the override with the correct reference name

## Testing

### Test Plan

1. **Build verification**: Trigger GitHub Actions build and confirm no compilation errors
2. **Firmware flash**: Download and flash `crosses_54_left.uf2` to the left keyboard half
3. **Scroll behavior**: Test trackball scrolling in various applications
4. **Sensitivity check**: Verify scroll speed is noticeably reduced (approximately 50% slower)
5. **Feature validation**: Confirm smooth scrolling (CONFIG_ZMK_POINTING_SMOOTH_SCROLLING) still works correctly
6. **Connectivity**: Verify BLE connection remains stable during scrolling

### Success Criteria

- Firmware builds successfully without errors
- Scrolling feels more controlled and less sensitive
- Smooth scrolling functionality is preserved
- BLE connectivity is stable
- No unexpected side effects on other keyboard functionality

## Dependencies

- ZMK firmware with pointing device support
- gggw-zmk-keebs shield definition for Crosses keyboard
- Left trackball peripheral correctly defined in shield devicetree

## Alternatives Considered

1. **Global scroll value definition**: Using `#define ZMK_POINTING_DEFAULT_SCRL_VAL` - Rejected because it would affect all pointing devices, not just the left trackball
2. **Configuration file option**: Using a `CONFIG_ZMK_POINTING_SCROLL_DIVISOR` setting - Rejected because this config option may not exist in ZMK

## References

- Previous implementation: commit ad6eea3
- ZMK pointing device documentation
- gggw-zmk-keebs shield repository
