# Heatsoaking

TODO:
- Interaction with chamber heating macros
- Improve introduction
- Should default basic bed heatsoak time be non-zero?
- More theory of operation for adaptive heatsoak

## Introduction

Most materials used in the construction of 3D printers are subject to thermal expansion - that is, the dimensions change depending on how hot or cold they are. The heated bed is a common example. It is typically necessary or desirable to allow the bed to get fully heated through before printing starts. More TODO.

1. [Bed heatsoaking](#bed-heatsoaking)
2. [Hotend heatsoaking](#hotend-heatsoaking)

## Bed Heatsoaking

RatOS provides two mechanisms to help manage heatsoaking:

- Basic heatsoaking simply waits for a configured time period after the bed has reached temperature.
- Beacon adaptive heatsoak dynamically determines when the printer has reached sufficient thermal stability.

The configured heatsoaking is used during the `START_PRINT` macro and by applicable calibration routines.

### Basic Bed Heatsoaking

Basic bed heatsoaking is used if adaptive heatsoaking is not enabled. It simply waits for a fixed time period after the bed has reached temperature before proceeding.

By default, no bed heatsoaking is performed.

`[gcode_macro RatOS]`

| Name                          | Possible Values     | Default    | Description                                                          |
| ----------------------------- | ------------------- | ---------- | -------------------------------------------------------------------- |
| variable_bed_heat_soak_time   | integer (seconds)   | 0          | Time in seconds to heat soak the bed before printing or calibration  |

### Beacon Adaptive Bed Heatsoaking

Beacon adaptive heatsoaking is only available when a beacon probe is configured. It uses beacon proximity measurements to dynamically assess the printer's dimensional stability and determine when it is appropriate to start printing. The soak duration is controlled by the duration of the first layer of the print, and the user's choice of "quality". Quality expresses the trade-off between soak time and first/early layer accuracy - much like slicer profiles trade speed against quality. The algorithm automatically adapts to different gantry constructions and sizes, bed temperatures, starting from cold or warm, and so on. The user does not need to be an experienced expert.

The algorithm has been tested successfully under various scenarios:

- aluminium extrusion, steel tube and titianium tube gantry beams
- various bed and enclosure temperatures
- open and enclosed printers
- from cold and warm start

Active air circulation and active enclosure heating have not been extensively tested at this time. Any setup which creates unstable thermal conditions - for example, coarse bang-bang thermostat controlled chamber heating with bimetallic gantry - will likely lead to a soak hitting the maximum wait time rather than completing sooner, as the thermal conditions will not stabilize adequately. Likewise, an open frame printer in a drafty environment may struggle to reach stability. In these scenarios, you may need to fall back to basic heatsoaking with fixed times.

Generally, the algorithm is designed to err on the safe side - that is, to soak longer than strictly necessary rather than shorter, but this cannot be guaranteed in all scenarios. If your printer has an unusual design or thermal characteristics, it is recommended to monitor the first few prints carefully to ensure that the adaptive heatsoaking is working as expected.

#### Basic Configuration

Adaptive heatsoaking is enabled by default for V-Core 4.x variants, and can be optionally enabled on other printers. Note that some aspects of the adpative algorithm were trained and tuned using the V-Core 4 design, so enabling it on other printers should be done with caution.

##### Layer Quality Settings

The layer quality setting controls the tradeoff between soak time and first layer quality:

- **1 (Rough)**: Fastest soak time. This setting seeks to allow the greatest amount of oversquish during the first few layers, while not being so oversquished as to risk print failure or bed damage. It is suitable for draft prints where speed is more important than first/early layer quality.
- **2 (Draft)**: Faster soak, first/early layer imperfections acceptable  
- **3 (Normal)**: Balanced soak time and quality (default)
- **4 (High)**: Longer soak, minimal first layer imperfections
- **5 (Maximum)**: Slowest soak, best first layer quality and Z dimensional accuracy

TODO: add examples of typical soak times for different printer sizes and layer quality settings.

##### Important: Maximum First Layer Duration

The `maximum_first_layer_duration` setting must be set correctly for your prints. This should be the longest first layer time you expect to print.

For example:
- If your typical first layers take up to 30 minutes, set this to `1800` (seconds)
- For very large prints with 60-minute first layers, set this to `3600`

:::warning
If you print a first layer significantly longer than this value, excessive thermal deflection may occur during the print, leading to poor first layer quality, print failure, or even damage to the bed.
:::

At present, `variable_beacon_adaptive_heat_soak_maximum_first_layer_duration` must be manually configured by the user. A future enhancement may be introduced to automatically determine this value from the gcode file for each print, so that it would no longer need to be manually configured.

`[gcode_macro RatOS]`

| Name | Possible Values | Default | Description |
| ---- | --------------- | ------- | ----------- |
| variable_beacon_adaptive_heat_soak                            | True / False    | *(see above)*   | Enable adaptive heat soaking based on beacon proximity measurements.              |
| variable_beacon_adaptive_heat_soak_layer_quality              | number (1-5)    | 3       | Controls the tradeoff between soak time and first layer quality. 1 = rough (fastest soak), 2 = draft, 3 = normal, 4 = high, 5 = maximum (slowest soak, best quality). Longer soak times leave less thermal deflection during the print.                                        |
| variable_beacon_adaptive_heat_soak_maximum_first_layer_duration | number        | 1800    | Maximum first layer print time in seconds for your gcode files. Must be set correctly. If you print a significantly longer first layer than this value, excessive thermal deflection may occur. Internally, this value is clamped to between 60 and 7200 seconds.                                      |

#### Advanced Configuraton

`[gcode_macro RatOS]`

| Name                                                          | Possible Values | Default | Description                                                                                                                                                                                                                                                                      |
| ------------------------------------------------------------- | --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| variable_beacon_adaptive_heat_soak_max_wait                   | number          | 5400    | The maximum time in seconds to wait for adaptive heat soaking to complete. This is a sanity limit to prevent waiting indefinitely.                                                                                                                                              |
| variable_beacon_adaptive_heat_soak_extra_wait_after_completion | number         | 0       | Extra time in seconds to wait after adaptive heat soaking is considered complete. Typically not needed, but could be useful for large printers with stable gantry designs (such as steel rail on steel tube) where the adaptive heat soak completes before the bed edge temperature stabilizes. |

#### Theory of Operation

Beacon adaptive heatsoaking places the toolhead at the centre of the bed and monitors beacon proximity measurements as the soak progresses. The rate of change of the proximity measurment, or *z-rate*, is used to determine when the printer has reached sufficient level of dimensional stability to start printing. The core principle here is to estimate how much residual z-deflection will occur during the first layer, and to wait until that residual deflection is below an acceptable threshold. 

TODO...

## Hotend Heatsoaking

Hotend heatsoaking is typically not needed, as typical modern hotends have relatively low thermal mass and experience no significant dimensional changes once at temperature. However, a hotend heatsoak duration can be configured if desired, and is used during printing and calibration routines.

`[gcode_macro RatOS]`

| Name                           | Possible Values     | Default    | Description                                                            |
| ------------------------------ | ------------------- | ---------- | ---------------------------------------------------------------------- |
| variable_hotend_heat_soak_time | integer (seconds)   | 0          | Time in seconds to heat soak the hotend before printing or calibration |
