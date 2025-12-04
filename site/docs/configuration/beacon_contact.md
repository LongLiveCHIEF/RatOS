# Beacon Contact

- [Prerequisites](#prerequisites)
- [Fully automated RatOS Beacon calibration](#fully-automated-ratos-beacon-calibration)
- [Initial calibration](#1-initial-calibration)
- [Beacon latency check](#2-beacon-latency-check)
- [Temperature expansion calibration](#3-temperature-expansion-calibration)
- [Final calibration](#4-final-calibration)
- [Beacon Scan Compensation](#5-beta-automated-beacon-scan-compensation)
- [First print and fine tuning](#6-first-print-and-fine-tuning)
- [RatOS configuration](#7-ratos-configuration)
- [Beacon Tools](#8-tools)
- [FAQ](#9-faq)

## Prerequisites

Please read the official [beacon contact documentation](https://docs.beacon3d.com/contact/), but do not follow any installation instructions. Beacon is already installed and configured in RatOS, you just need to connect it to your Raspberry Pi.

## NEW! Bed and hotend heat soaking time

It is recommended to use the new RatOS heat soaking variables. The Beacon calibration and the `START_PRINT` macro are using these values.

```
[gcode_macro RatOS]
variable_bed_heat_soak_time: 1200
variable_hotend_heat_soak_time: 300
variable_start_print_park_in: 'primeblob'
```

## Fully automated RatOS Beacon calibration

RatOS comes with a fully automated Beacon model and temperature offset calibration.

By default the Beacon contact feature is enabled. If you want to disable it, set `variable_beacon_contact_start_print_true_zero` to False.

1. Run `BEACON_RATOS_CALIBRATE BED_TEMP=85 CHAMBER_TEMP=45`. Use your target temperature for the `BED_TEMP` and `CHAMBER_TEMP` parameter. `CHAMBER_TEMP` is optional, and can be omitted.

The automated beacon calibration will run the following calibrations and tests, which can also be used individually. Please make sure to read every section before starting the calibration.

- [Initial calibration](#1-initial-calibration)
- [Beacon latency check](#2-beacon-latency-check)
- [Temperature expansion calibration](#3-temperature-expansion-calibration) (for non IDEX printer)
- [Final calibration](#4-final-calibration)
- [Beacon scan compensation](#5-beacon-scan-compensation) - _if scan compensation is enabled, a compensation mesh will be created if needed_

All calibration results will be saved automatically, and no user action is required. Klipper will restart on its own after the calibration is complete.

## 1. Initial calibration

We need to create an initial Beacon model to be able to home the printer.

1. Run `BEACON_INITIAL_CALIBRATION`

It will home your printer and run the calibration fully automated. This command can throw a tolerance error - in this case, simply repeat it until the command completes successfully.

For safety and peace of mind, the LED will turn on as soon as the contact system determines it has a strong enough signal for detection. It should normally turn on up to 10mm in advance of the metal target, allowing enough time to manually e-stop the machine if necessary.

2. Run `SAVE_CONFIG` to save the model to your printer.cfg file.

## 2. Beacon latency check

This test will show you the quality of your Beacon probing.

- Run `BEACON_POKE_TEST`

It will home your printer and poke the bed multiple times. After the test completes, check the console output - it should look similar to this:

```
Overshoot: 35.625 um
Triggered at: z=0.07369 with latency=2
Armed at: z=4.76021
Poke test from 5.000 to -0.300, at 3.000 mm/s
```

Compare your latency values with the following list.

| Score | Notes                                                                   |
| ----- | ----------------------------------------------------------------------- |
| 0-1   | Extremely low noise, rarely achieved                                    |
| 2-4   | Excellent performance for a typical printer                             |
| 5-8   | Acceptable performance, machine may have considerable cyclic axis noise |
| 9-11  | Not ideal, may want to verify proper mounting or use thinner stackups   |
| 12-14 | Reason for concern, present setup may be risky to continue with         |

## 3. Temperature expansion calibration

RatOS comes with built-in temperature expansion calibration and compensation.

**Preparation**

- Unload filament from the nozzle
- Make sure the nozzle is clean and that no filament is leaking out of it. Make a manual cold pull or use the RatOS `COLD_PULL` macro
- Let the machine cool down to ambient temperature
- Do **NOT** perform this calibration on a smooth PEI sheet - in this case, turn the sheet around and calibrate on its bare metal surface

**Cold Pull Macro**

The cold pull macro lets you perform an automated cold pull to clean your nozzle. Before the cold pull, 30mm of filament will be extruded at the specified `EXTRUSION_TEMP`.

The default values work well for PLA cold pulls. For PETG and ABS, you should use higher temperatures like `EXTRUSION_TEMP=250 COLD_PULL_TEMP=95`. If you hear skipping during the cold pull, slightly increase the `COLD_PULL_TEMP`.

```
COLD_PULL EXTRUSION_TEMP=220 COLD_PULL_TEMP=80 TOOLHEAD=0
```

**Single toolhead printer**

- Run `BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET`

This command will home your printer and run the calibration automatically. The process will take some time to complete.

**IDEX printer**

- Start VAOC
- Center both nozzles over the camera
- Click the `Calibrate Thermal Expansion` button in VAOC

This will automatically calibrate both nozzles. The process will take some time to complete.

It is recommended to repeat this calibration whenever you change a nozzle or before loading new filament.

After the test finishes, check the console output. A typical result looks like this:

```
RatOS | Beacon: T0 expansion coefficient: 0.075000
```

This value is in millimeters and represents the thermal expansion for a temperature difference of 100°C. RatOS uses this value to automatically calculate and apply the needed offset.

The result is automatically saved to the configuration file - no user action is required.

## 4. Final Calibration

For scan method Z-homing, we should create a Beacon model under real operating conditions. While optional, this step is recommended.

- Run `BEACON_FINAL_CALIBRATION BED_TEMP=85 CHAMBER_TEMP=45`

Use your target temperatures for the `BED_TEMP` and `CHAMBER_TEMP` parameters. This command will home your printer and run the calibration automatically.

- Run `SAVE_CONFIG` to save the model to `printer.cfg`.

## 5. Beacon Scan Compensation

With RatOS, you can automatically compensate for gantry twist across the entire build plate and inaccuracies in the build sheet's material thickness that cause ripple effects on scanned bed meshes.

### How do I know if I need this?

#### Measuring Gantry Twist

- `BEACON_MEASURE_GANTRY_TWIST` automatically measures the gantry twist at multiple locations on the bed. It will home your printer and level the bed if needed. The results will be displayed after the test has finished. The command may throw a tolerance error - in this case, simply repeat it until successful.

The result will look like the following example:

```
Gantry twist relative to the center

Low gantry twist: 50.324232μm.
You may experience first layer inconsistensies, consider beacon scan compensation.

Front left: 2.362576μm
Front center: 16.475233μm
Front right: -47.959155μm
Left center: 24.925954μm
Right center: -22.468182μm
Back left: 50.324232μm
Back center: 37.143389μm
Back right: -4.167238μm
```

Check your build plate:

- Create a scan bed mesh and save it as a custom profile name
- Rotate the build plate 90 degrees (rotate only the sheet, not the bed itself)
- Create a second scan bed mesh and save it with a different profile name
- If you observe that the mesh pattern follows the build plate when rotated, you need this compensation

Scan 1

![Scan 1: 0 degrees rotation](_media/0degree.png)

Scan 2 with the build plate rotated by 90°

![Scan 2: 90 degrees rotation](_media/90degree.png)

### Enabling Scan Compensation

To enable scan compensation, add the following to your `printer.cfg`:

```properties
[gcode_macro RatOS]
variable_beacon_scan_compensation_enable: True          # Enables beacon scan compensation
variable_beacon_scan_compensation_profile: "auto"       # Use "auto" for automatic selection based on bed temperature
```

### Creating Compensation Meshes

1. Run `BEACON_CREATE_SCAN_COMPENSATION_MESH BED_TEMP=85 CHAMBER_TEMP=45` to create a compensation mesh.

Use your target temperatures for the `BED_TEMP` and `CHAMBER_TEMP` parameters. The command will home your printer, heat it to the target temperatures, wait for thermal stabilization, and create the compensation mesh automatically.

2. You'll need a compensation mesh for each build plate and for different bed temperature ranges.

The compensation mesh creation process automatically determines the appropriate mesh name based on the bed temperature. When using `variable_beacon_scan_compensation_profile: "auto"`, RatOS will automatically select the most appropriate compensation mesh based on your current bed temperature.

Alternatively, you can specify a custom profile name: `BEACON_CREATE_SCAN_COMPENSATION_MESH BED_TEMP=85 PROFILE="PEI_PC_85"`

3. If the feature is enabled, it will automatically compensate during printing - no additional user action is required.

Click the image to open the video and see the results in action

[<img src="https://img.youtube.com/vi/qjRhAHsX0Hc/maxresdefault.jpg" width="50%" />](https://youtu.be/qjRhAHsX0Hc)

## 5a. Adaptive Heat Soak

Adaptive heat soak uses Beacon proximity measurements to monitor the thermal stability of your printer. Instead of using a fixed heat soak time, it waits until the printer reaches thermal stability before starting the print, reducing thermal Z deflection during the first layer.

:::info
Adaptive heat soak is enabled by default for V-Core 4 printers. Other printers can opt-in, but proceed with caution as the algorithm is currently tuned for the V-Core 4 design.
:::

### How It Works

The system continuously measures the rate of Z-axis change using Beacon proximity data. When the rate of change falls below a calculated threshold and remains stable, the heat soak is considered complete. The threshold is automatically calculated based on your layer quality preference and maximum first layer duration.

### Configuration

To enable adaptive heat soak, add this to your `printer.cfg`:

```properties
[gcode_macro RatOS]
variable_beacon_adaptive_heat_soak: True
variable_beacon_adaptive_heat_soak_layer_quality: 3              # 1=rough (fast), 5=maximum (slow, best)
variable_beacon_adaptive_heat_soak_maximum_first_layer_duration: 1800  # Maximum first layer time in seconds
variable_beacon_adaptive_heat_soak_max_wait: 5400                # Maximum wait time (safety limit)
```

### Layer Quality Settings

The layer quality setting controls the tradeoff between soak time and first layer quality:

- **1 (Rough)**: Fastest soak time, some oversquish may develop during first layer
- **2 (Draft)**: Faster soak, minor first layer imperfections acceptable  
- **3 (Normal)**: Balanced soak time and quality (default)
- **4 (High)**: Longer soak, minimal first layer imperfections
- **5 (Maximum)**: Slowest soak, best first layer quality and Z dimensional accuracy

### Important: Maximum First Layer Duration

The `maximum_first_layer_duration` setting must be set correctly for your prints. This should be the longest first layer time you expect to print.

For example:
- If your typical first layers take up to 30 minutes, set this to `1800` (seconds)
- For very large prints with 60-minute first layers, set this to `3600`

:::warning
If you print a first layer significantly longer than this value, excessive thermal deflection may occur during the print, leading to poor first layer quality, print failure, or even damage to the bed.
:::

### Manual Use

The adaptive heat soak runs automatically during `START_PRINT` when enabled. You can also run it manually:

```gcode
BEACON_WAIT_FOR_PRINTER_HEAT_SOAK LAYER_QUALITY=3 MAXIMUM_FIRST_LAYER_DURATION=1800
```

Parameters:
- `LAYER_QUALITY`: 1-5 (default from configuration)
- `MAXIMUM_FIRST_LAYER_DURATION`: Time in seconds, 60-7200 (default from configuration)
- `MINIMUM_WAIT`: Minimum wait time in seconds (default: 0)
- `MAXIMUM_WAIT`: Maximum wait time in seconds (default from configuration)

## 6. First print and fine tuning

1. Print a 150x30mm single layer rectangle in the middle of the build plate.
2. While printing, fine-tune using baby stepping.
3. Run `Z_OFFSET_APPLY_PROBE` to save the changes. Don't click the button - type `Z_OFFSET_APPLY_PROBE` into the console.

## 7. RatOS configuration

The Beacon contact feature is activated by default, so no configuration is required. However, you can override the settings to enable additional Beacon contact features if desired. Simply copy and paste the relevant configuration sections below into your printer.cfg file and modify the settings as needed.

For a complete reference of all Beacon-related variables, see the [Beacon probe section in the macros documentation](/docs/configuration/macros#beacon-probe).

### Basic Configuration

```properties
[gcode_macro RatOS]
variable_beacon_bed_mesh_scv: 25                        # Square corner velocity for bed meshing with proximity method
variable_beacon_contact_prime_probing: True             # Probes for priming with contact method
variable_beacon_contact_expansion_compensation: True    # Enables hotend thermal expansion compensation
```

### True Zero Settings

```properties
[gcode_macro RatOS]
variable_beacon_contact_start_print_true_zero: True     # Uses contact to determine true Z=0 during START_PRINT
variable_beacon_contact_start_print_true_zero_fuzzy_position: True  # Randomizes true zero position to avoid wear marks
variable_beacon_contact_wipe_before_true_zero: True     # Enables nozzle wipe before true zeroing
variable_beacon_contact_true_zero_temp: 150             # Nozzle temperature for true zeroing
                                                        # WARNING: If using a smooth PEI sheet, be careful with temperature
```

### Contact Mode Settings (Not Recommended)

:::warning
Using contact mode for homing, bed mesh, or z-tilt is not recommended on textured surfaces due to potential significant variation in contact measurements.
:::

```properties
[gcode_macro RatOS]
variable_beacon_contact_z_homing: False                 # Makes all G28 calls use contact instead of proximity scan
variable_beacon_contact_bed_mesh: False                 # Performs bed mesh with contact method
variable_beacon_contact_bed_mesh_samples: 2             # Number of probe samples for contact bed mesh
variable_beacon_contact_z_tilt_adjust: False            # Performs z-tilt adjust with contact method
variable_beacon_contact_z_tilt_adjust_samples: 2        # Number of probe samples for contact z-tilt adjust
```

### Model Calibration

```properties
[gcode_macro RatOS]
variable_beacon_contact_calibrate_model_on_print: True  # Calibrate a new beacon model every print
                                                        # Recommended, especially if you swap build plates
```

### Scan Compensation

```properties
[gcode_macro RatOS]
variable_beacon_scan_compensation_enable: False         # Enables beacon scan compensation
variable_beacon_scan_compensation_profile: "auto"       # Use "auto" for automatic selection or specify profile name
variable_beacon_scan_compensation_desired_spacing: 10   # Desired spacing between probe points (mm)
variable_beacon_scan_compensation_bed_temp_mismatch_is_error: False  # Raise error on temp mismatch
variable_beacon_scan_method_automatic: False            # Enable METHOD=automatic scan option (not recommended)
```

### Adaptive Heat Soak

Adaptive heat soak monitors thermal stability using Beacon proximity measurements, reducing thermal Z deflection during the first layer.

```properties
[gcode_macro RatOS]
variable_beacon_adaptive_heat_soak: False               # Enable adaptive heat soaking (enabled by default on V-Core 4)
variable_beacon_adaptive_heat_soak_max_wait: 5400       # Maximum wait time in seconds
variable_beacon_adaptive_heat_soak_extra_wait_after_completion: 0  # Extra wait time after soak completes
variable_beacon_adaptive_heat_soak_layer_quality: 3     # Quality level: 1=rough (fast), 5=maximum (slow, best quality)
variable_beacon_adaptive_heat_soak_maximum_first_layer_duration: 1800  # Maximum first layer time (60-7200 seconds)
```

### Advanced Settings

```properties
[gcode_macro RatOS]
variable_beacon_contact_poke_bottom_limit: -1           # The bottom limit for the contact poke test
```

## 8. Tools

### Measuring Z-Axis Backlash

The Beacon macro `BEACON_ESTIMATE_BACKLASH` allows you to measure the backlash in your printer's setup. Before using this macro, ensure your printer is homed and the bed is properly leveled.

```
Median distance moving up 1.99990, down 2.00286, delta 0.00296 over 20 samples
```

The delta value represents your backlash in millimeters.

## 9. FAQ

### Q: How can I set different Z-offsets for different filaments?

A: If you want to have different Z-offsets for different filament profiles, you can use `SET_GCODE_OFFSET Z_ADJUST=+0.01` for positive adjustments or `SET_GCODE_OFFSET Z_ADJUST=-0.01` for negative adjustments in your filament profile's custom G-code section. Note: Using `Z` instead of `Z_ADJUST` will cause Klipper to replace all previously set Z-offset adjustments, including hotend expansion compensation, with your provided value (which is not recommended).

### Q: What happened to SAVE_Z_OFFSET?

A: The `SAVE_Z_OFFSET` command has been replaced with `Z_OFFSET_APPLY_PROBE`. Use `Z_OFFSET_APPLY_PROBE` to save your Z-offset adjustments after baby stepping.
