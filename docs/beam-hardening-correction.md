# Beam Hardening Correction

## Method

back-and-forth starts with projection-domain polynomial linearization:

```text
p_corrected = scale * (x4*p^4 + x3*p^3 + x2*p^2 + x1*p + x0)
```

where `p = -log(I / I0)` after flat-field normalization. This is the practical industrial CT workflow used by Nikon/CT Pro-style reconstruction: choose a correction strength, preview slices, and pick the lowest setting that removes cupping without flattening true material contrast.

Metadata beam-hardening coefficients are used by default. Once the user chooses
a Beam-tab preset or custom polynomial, that correction applies to subsequent
previews and compatible reconstruction methods and is recorded with the output.

## Presets

The GUI presets mirror a CT Pro 3D-style polynomial preset ladder using:

```text
Y = a * (b + cX + dX^2 + eX^3 + fX^4)
```

with `a` mapped to back-and-forth as `scale`, and `x4=f`, `x3=e`, `x2=d`, `x1=c`, and `x0=b`.

| Preset | scale | x4 | x3 | x2 | x1 | x0 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| None | 1.0 | 0.0 | 0.0 | 0.0 | 1.0 | 0.0 |
| Very Light | 1.32 | 0.0 | 0.0 | 0.25 | 0.75 | 0.0 |
| Light | 1.0 | 0.0 | 0.0 | 0.50 | 0.50 | 0.0 |
| Medium | 4.44 | 0.0 | 0.0 | 0.80 | 0.20 | 0.0 |
| Strong | 1.0 | 0.0 | 0.0 | 0.90 | 0.10 | 0.0 |
| Very Strong | 1.0 | 0.0 | 0.80 | 0.20 | 0.0 | 0.0 |

These are not universal material constants. They are a useful scan-preview ladder. The correct setting depends on kV, filtration, exposure, material, sample thickness, and acceptable contrast loss.

## Preview Guidance

- Use the weakest preset that visibly reduces cupping in a homogeneous region.
- Use the preset preview grid before committing to a correction strength.
- Watch edges and small pores/features: over-correction can reduce true edge contrast or create inverse cupping.
- Multi-material objects are necessarily a compromise with single-polynomial correction.
- Use calibration or a validated material-specific correction when quantitative
  attenuation accuracy is required.

## Sources

- CT Pro User Manual: describes preset and custom beam-hardening correction, with custom correction based on flux through different material thicknesses. The preset names in back-and-forth are vendor-workflow labels, not material constants.
- Dewulf et al., "Sense and non-sense of beam hardening correction in CT metrology", CIRP Annals, 2012, DOI [10.1016/j.cirp.2012.03.013](https://doi.org/10.1016/j.cirp.2012.03.013): describes fourth-order polynomial linearization used in CT metrology.
- "Comparison of X-ray computed tomography and coordinate-measuring system dimensional measurement for additive manufacturing parts using physical and simulation methods", Measurement, 2024, DOI [10.1016/j.measurement.2024.114414](https://doi.org/10.1016/j.measurement.2024.114414): reproduces CT Pro 3D polynomial shape coefficients used as the preview ladder. The `Very Light` `scale = 1.32` entry is a Nikon final-form XTekCT value observed in project sample metadata, not a value supported by that paper alone.
- Hammersberg and Mangard, "Correction for beam hardening artefacts in computerised tomography", Journal of X-Ray Science and Technology, 1998, DOI [10.3233/XST-1998-00024](https://doi.org/10.3233/XST-1998-00024): warns that low-order polynomials may be insufficient for some materials and motivates calibration/spline/LUT workflows.

