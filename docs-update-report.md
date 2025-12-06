# Documentation Update Report

This document tracks changes made to the RatOS documentation based on code changes between commits `44152d1a` (v2.1.x) and `2364a93a` (vnext-development).

## Overview

The changes span multiple areas:
1. New Beacon-related features (adaptive heat soak, enhanced mesh management, true zero improvements)
2. Fast configuration system (fastconfig.py replacing configfile)
3. Extensive macro changes (mesh calibration, filament handling, overrides, IDEX)
4. New hardware definitions (chamber filters, lighting, filament sensors, toolhead alignment)
5. Enhanced filament sensor behavior control

## Summary of Changes

This update focused on documenting user-facing features while preserving technical accuracy. The changes reflect significant enhancements to the Beacon probe system, improved thermal management, and more flexible filament sensor configuration.

## Changes Made

### 1. macros.md - Beacon Probe Section

**Updated the Beacon probe section** with comprehensive documentation of new features:

#### New Beacon Variables Documented:
- **Basic Settings**: Updated `variable_beacon_contact_expansion_compensation` description
- **Contact Mode Settings**: Added warnings about textured surfaces, reorganized into subsection
  - `variable_beacon_contact_z_homing` - with warning
  - `variable_beacon_contact_bed_mesh` - with warning  
  - `variable_beacon_contact_bed_mesh_samples`
  - `variable_beacon_contact_z_tilt_adjust` - with warning
  - `variable_beacon_contact_z_tilt_adjust_samples`

- **True Zero Settings**: New subsection
  - `variable_beacon_contact_start_print_true_zero`
  - `variable_beacon_contact_start_print_true_zero_fuzzy_position` - prevents wear marks
  - `variable_beacon_contact_wipe_before_true_zero`
  - `variable_beacon_contact_true_zero_temp`

- **Model Calibration**: New subsection
  - `variable_beacon_contact_calibrate_model_on_true_zero` - auto-calibrate model every print

- **Scan Compensation**: New comprehensive subsection
  - `variable_beacon_scan_compensation_enable`
  - `variable_beacon_scan_compensation_profile` - with "auto" option
  - `variable_beacon_scan_compensation_desired_spacing`
  - `variable_beacon_scan_compensation_bed_temp_mismatch_is_error`
  - `variable_beacon_scan_method_automatic`

- **Adaptive Heat Soak**: Completely new feature section
  - `variable_beacon_adaptive_heat_soak`
  - `variable_beacon_adaptive_heat_soak_max_wait`
  - `variable_beacon_adaptive_heat_soak_extra_wait_after_completion`
  - `variable_beacon_adaptive_heat_soak_layer_quality` - 5-level quality scale
  - `variable_beacon_adaptive_heat_soak_maximum_first_layer_duration`

**Removed deprecated variables**:
- `variable_beacon_contact_z_calibration`
- `variable_beacon_contact_calibration_location`
- `variable_beacon_contact_calibrate_margin_x`
- `variable_beacon_contact_calibration_temp`
- `variable_beacon_contact_expansion_multiplier`
- `variable_beacon_scan_compensation_probe_count`

### 2. macros.md - Toolhead Configuration Section

**Added new filament sensor variables**:
- `variable_toolhead_sensor_button_only_when_sensor_enabled` - control button behavior when sensor disabled
- `variable_toolhead_detect_clog_only_when_sensor_enabled` - control clog detection when sensor disabled

These provide more granular control over filament sensor behavior, particularly useful when temporarily disabling sensors.

### 3. macros.md - User Macro Hooks Section

**Added new idle timeout hooks**:
- `_USER_BEFORE_IDLE_TIMEOUT` - runs before entering idle timeout
- `_USER_AFTER_IDLE_TIMEOUT` - runs after entering idle timeout

### 4. macros_list.md - Command Reference Updates

**Updated macro listings** with new commands and parameters:

#### macros/mesh.cfg:
- Added `_BED_MESH_SANITY_CHECK`
- Added `_START_PRINT_PREFLIGHT_CHECK_AND_RESET_BED_MESH`
- Updated `CALIBRATE_ADAPTIVE_MESH` with `ZERO_REF_POS_X` and `ZERO_REF_POS_Y` parameters

#### macros/overrides.cfg:
- Added `BED_MESH_CALIBRATE` override (now includes auto ABL and probe deployment)

#### macros/load_filament.cfg:
- Added `AUTOMATED` parameter to:
  - `LOAD_FILAMENT`
  - `_DEFAULT_LOAD_FILAMENT`
  - `_IDEX_LOAD_FILAMENT`
  - `_LOAD_FILAMENT`
  - `_MOVE_TO_PARKING_POSITION`
  - `_MOVE_TO_LOADING_POSITION`
  - `_CLEANING_MOVE`

#### macros/unload_filament.cfg:
- Added `_LEGACY_UNLOAD_FILAMENT` (moved from inline)
- Added `AUTOMATED` parameter to all unload macros

#### macros/user-hooks.cfg:
- Added `_USER_BEFORE_IDLE_TIMEOUT`
- Added `_USER_AFTER_IDLE_TIMEOUT`

#### macros/util.cfg:
- Added `_STOP_AND_RAISE_ERROR` with `MSG` parameter

#### z-probe/beacon.cfg:
- Added extensive list of new Beacon commands:
  - `BEACON_RATOS_CALIBRATION` - fully automated calibration
  - `BEACON_RATOS_CALIBRATE` - alias for above
  - `BEACON_INITIAL_CALIBRATION`
  - `BEACON_FINAL_CALIBRATION` with heat soak integration
  - `BEACON_POKE_TEST`
  - `BEACON_MEASURE_BEACON_OFFSET`
  - `BEACON_CREATE_SCAN_COMPENSATION_MESH` - create compensation meshes
  - `BEACON_APPLY_SCAN_COMPENSATION` - manually apply compensation
  - `BEACON_WAIT_FOR_PRINTER_HEAT_SOAK` - **new adaptive heat soak command**
  - `SET_ZERO_REFERENCE_POSITION` - set mesh zero reference
  - `Z_OFFSET_APPLY_PROBE` - replaces SAVE_Z_OFFSET
  - Multiple internal macros for scan compensation and offset management

### 5. beacon_contact.md - Comprehensive Beacon Documentation Updates

#### Section 5: Beacon Scan Compensation
- Removed "BETA" designation
- Updated to reflect automatic profile naming based on bed temperature
- Changed from manual `PROFILE` parameter to automatic temperature-based naming
- Added `variable_beacon_scan_compensation_profile: "auto"` as recommended setting
- Clarified that compensation meshes are automatically selected based on bed temperature
- Simplified user workflow - less manual profile management required

#### New Section 5a: Adaptive Heat Soak
- Comprehensive new section documenting adaptive heat soak feature
- Explained how the system works using Beacon proximity measurements
- Documented layer quality settings (1-5 scale):
  - 1 = Rough (fastest)
  - 2 = Draft
  - 3 = Normal (default)
  - 4 = High
  - 5 = Maximum (slowest, best quality)
- Important warnings about `maximum_first_layer_duration` setting
- Configuration examples
- Manual command usage with all parameters
- Note that it's enabled by default for V-Core 4

#### Section 6: First Print and Fine Tuning
- Changed `SAVE_Z_OFFSET` to `Z_OFFSET_APPLY_PROBE`

#### Section 7: RatOS Configuration
- Completely restructured into logical subsections:
  - Basic Configuration
  - True Zero Settings
  - Contact Mode Settings (with warnings)
  - Model Calibration
  - Scan Compensation
  - Adaptive Heat Soak (new)
  - Advanced Settings
- Added reference link to full variable documentation
- Updated `variable_beacon_scan_compensation_profile` default to "auto"
- Removed deprecated variables
- Added all new adaptive heat soak variables

#### Section 9: FAQ
- Added new FAQ entry about `SAVE_Z_OFFSET` → `Z_OFFSET_APPLY_PROBE` transition

### 6. Internal Changes Not Documented (Out of Scope)

The following changes were identified but not documented as they are internal/advanced:

- New Python modules: `fastconfig.py`, `beacon_mesh.py`, `beacon_true_zero_correction.py`, `ratos_z_offset.py`
- Changes to `ratos.py` internal commands
- New hardware definition schemas (these are for configurator use)
- Mesh metadata parameters (used internally by the system)
- Various internal macros prefixed with underscore

These could be documented in a separate technical reference if needed.

## Key User-Facing Improvements

1. **Adaptive Heat Soak**: Automatically determines optimal heat soak time based on thermal stability
2. **Enhanced Scan Compensation**: Automatic temperature-based profile selection
3. **Fuzzy True Zero**: Prevents bed surface wear from repeated probing at same point
4. **Better Filament Sensor Control**: More granular control over sensor behavior
5. **Model Auto-Calibration**: Option to calibrate fresh model every print for consistent results
6. **Improved Z-Offset Management**: Clearer command naming (`Z_OFFSET_APPLY_PROBE`)

## Breaking Changes / Migrations

### Commands
- `SAVE_Z_OFFSET` → `Z_OFFSET_APPLY_PROBE` (old command deprecated but documented in FAQ)

### Variables  
- Several Beacon variables removed/renamed (documented in beacon_contact.md)
- `beacon_scan_compensation_profile` now defaults to "auto" instead of "Contact"

## Testing Recommendations

Users upgrading should:
1. Review new Beacon variables in their printer.cfg
2. Test adaptive heat soak if using V-Core 4
3. Recreate scan compensation meshes if using that feature (automatic naming)
4. Update any custom macros using `SAVE_Z_OFFSET` to use `Z_OFFSET_APPLY_PROBE`

---

_Report completed: 2025-12-04_
_Documentation updated for RatOS vnext (commit 2364a93a)_
