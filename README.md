# Cloud_cci spectral band adjustment factors (SBAFs)

This repository provides two sets of linear spectral band adjustment factors
(SBAFs) used in the Cloud_cci context to harmonize observations from satellite
imagers with different spectral response functions.

The coefficients use the convention

```text
reference_value = slope * source_value + offset
```

The **slope is dimensionless**. The **offset has the unit of the adjusted
quantity**: percentage points for reflectance and kelvin for brightness
temperature.

## Contents

| File | Description |
|---|---|
| `theoretical_sbafs_noaa18.txt` | Human-readable theoretical SBAF table |
| `theoretical_sbafs_noaa18.py` | Importable theoretical SBAF coefficients |
| `empirical_sbafs_m11.txt` | Human-readable empirical SBAF table |
| `empirical_sbafs_m11.py` | Importable empirical SBAF coefficients |

## Theoretical SBAFs

The theoretical SBAFs use **NOAA-18 AVHRR (WMO 209)** as the common spectral
reference.

The method follows the same general concept described for spectral band
adjustment in Appendix C of Stengel et al. (2020), but the reference sensor in
the coefficient set provided here is NOAA-18 AVHRR.

Hyperspectral measurements are used to generate synthetic observations for each
sensor:

1. SCIAMACHY spectra are used for the solar channels.
2. IASI spectra are used for the thermal infrared channels.
3. The hyperspectral spectra are convolved with the spectral response functions
   (SRFs) of the source sensor and NOAA-18 AVHRR.
4. A linear relationship is fitted between the synthetic source-sensor value
   and the corresponding NOAA-18 value.

Thus,

```text
NOAA18_equivalent = slope * source_sensor_value + offset
```

The six channels correspond to the AVHRR reference channels:

| AVHRR channel | Approx. wavelength | Quantity |
|---|---:|---|
| 1 | 0.6 µm | reflectance |
| 2 | 0.8 µm | reflectance |
| 3a | 1.6 µm | reflectance |
| 3b | 3.7 µm | brightness temperature |
| 4 | 10.8 µm | brightness temperature |
| 5 | 12.0 µm | brightness temperature |

Solar-channel values use the 0–100 reflectance scale of the SCIAMACHY input
spectra. Thermal SRF-integrated radiances are converted to brightness
temperatures before the linear fit.

For thermal channels, the theoretical day fit uses **SZA < 80°** and the night
fit uses **SZA > 90°**. The solar channels have a daytime fit only; their night
entries are neutral placeholders (`slope = 1`, `offset = 0`).

Methodological context:

> Stengel, M. et al. (2020): *Cloud_cci Advanced Very High Resolution
> Radiometer post meridiem (AVHRR-PM) dataset version 3: 35-year climatology of
> global cloud and radiation properties*, Earth System Science Data, 12,
> 41–60. https://doi.org/10.5194/essd-12-41-2020  
> See Appendix C.

## Empirical SBAFs

The empirical SBAFs use **Meteosat-11 SEVIRI (WMO 70)** as the common reference.
They are derived from collocated satellite observations and map the measured
source-sensor quantity to its Meteosat-11-equivalent value:

```text
M11_equivalent = slope * source_sensor_value + offset
```

The matchup strategy differs for geostationary and polar-orbiting sensors.

For the **geostationary (GEO) sensors**, the empirical relationships are derived
from the spatial overlap regions of pairs of GEO sensors. Only observations
between **60°S and 60°N** are considered. Within these overlap regions,
measurements are paired when both sensors observe the **same L1g grid pixel at
the same nominal observation time**. These collocated measurements are used to
derive the linear relationships between the individual GEO sensors and the
common Meteosat-11 reference.
Not all GEO sensors have a direct overlap with Meteosat-11. If a direct sensor-to-Meteosat-11 relationship cannot be derived, the adjustment is transferred along a path of overlapping GEO sensors. For example, a Himawari-8 observation can be transformed first to the Meteosat-8 reference and subsequently from Meteosat-8 to Meteosat-11.

For the **polar-orbiting sensors**, the original sensor swaths are collocated
with Meteosat-11 observations on the common L1g grid. The polar measurements
are spatially and temporally matched to Meteosat-11 before the empirical fits
are derived.


The empirical channels are:

| Channel | Quantity | Offset unit |
|---|---|---|
| 0.65 µm | reflectance [%] | percentage points |
| 0.86 µm | reflectance [%] | percentage points |
| 3.8 µm | brightness temperature [K] | K |
| 11.0 µm | brightness temperature [K] | K |
| 12.0 µm | brightness temperature [K] | K |

### Current application rules

These rules describe the current application of the empirical coefficient set
and are separate from the basic linear SBAF definition.

- 0.65/0.86 µm: day for SZA < 80°; larger SZA values are not used.
- Polar 3.8 µm: day for SZA < 75°; 75° ≤ SZA < 90° is not used; night for
  SZA ≥ 90°.
- GEO 3.8 µm: day for SZA < 90°; night for SZA > 90°.
- 11/12 µm: day for SZA < 80°; night for SZA ≥ 80°.

## Python usage

```python
from empirical_sbafs_m11 import apply_sbaf

# Example: transform a source-sensor value to the M11-equivalent value.
m11_value = apply_sbaf(
    value=250.0,
    wmo_id=270,
    mode="day",
    channel="11.0",
)
```

The coefficient modules have no external Python dependencies. `apply_sbaf`
also works with NumPy arrays when an array is passed as `value`.
