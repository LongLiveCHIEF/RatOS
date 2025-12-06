# Beacon True Zero Correction

RatOS extends the beacon contact `BEACON_AUTO_CALIBRATE` command with a true zero correction feature via the `[beacon_true_zero_correction]` module. This feature is also known as _multipoint probing_ or _multipoint true zero_.

## Background

Following an extensive investigation, we found that while contact measurements at the same location are very repeatable, measuring neigbouring points (on a 1mm grid) can yield a result that in some cases differs by up to 100 microns (0.1mm). When used for true zero, this close proximity positional variablilty of contact readings is enough to have a significant impact on first layer consistency: a small difference in build sheet position or toolhead homing can lead to an effectively arbitrary true zero difference that is approaching 50% of a 0.2mm first layer.

Based on analysis of thousands of test data points, we determined that by applying a particular statistical method to six additional single-dive probes of randomly-positioned points in close proximity to the nominal true zero position, we could determine a correction value that significantly reduces the range of variability, with no negative impact for inherently low-variability build sheets. Also of note was the observation that the variabilty has some correlation to surface roughness, with textured surfaces generally showing more variability than smooth surfaces. Further, we saw the greatest variation on brand new textured PEI build sheets, and that variability appears to reduce as the sheet gets more use.

## Configuration

The `[beacon_true_zero_correction]` module is enabled by default in RatOS whenever a beacon probe is configured. It can be configured by adding overrides in `printer.cfg` as follows:

```properties
[beacon_true_zero_correction]
# If True, the true zero correction feature is disabled. Default: False
disabled: False                             
# z values greater than z_rejection_threshold are rejected. These typically correspond to early triggering
# of beacon contact before the nozzle has touched the bed. From test data, these are rare. Only 0.028% of samples
# exceeded 75um (from over 32,000 samples across multiple machines and print surfaces).
# Rejected samples are discarded and another sample is taken in its place at a different random location.
# Default: 0.075, minimum: 0.03
z_rejection_threshold: 0.075
# Controls the sampling strategy, notably affecting the number of points probed.
# - Level 1: 6 points probed, 1 zero sample, use mean of 3 minimal samples. This is the default and recommended level.
# - Level 2: 10 points probed, 1 zero sample, use mean of 3 minimal samples. This is a more robust probing strategy.
# - Level 3: 12 points probed, 1 zero sample, use mean of 3 minimal samples. This is the most robust probing strategy.
# From extensive testing, level 1 is very effective and efficient, with levels 2 and 3 offering only very modest gains
# and diminishing returns. Levels 2 and 3 are included for diagnostic purposes, but level 1 is recommended for most users.
# The zero sample is the implied zero sample from the initial BEACON_AUTO_CALIBRATE invocation.
sampling_strategy: 1
# If True, each of the multiple probe locations will itself be probed several times using
# the standard beacon error detection logic. From extensive testing, this mode offers no benefit
# and should not be used. It is included only as an option for diagnostic purposes. Deafult: False
use_error_corrected_probing: False
```
