# Limited-Angle And Offset/Truncated Reconstruction

## Scan Classes

Back-and-forth treats these as distinct acquisition conditions:

1. **Short scan**: angular coverage is roughly `180 degrees + full fan angle` or more, equivalently `180 degrees + 2 * fan_half_angle`. This is incomplete compared with 360 degrees, but complete enough for analytic short-scan reconstruction using Parker-style redundancy weighting.
2. **True limited-angle scan**: angular coverage is below the complete-data requirement, or has a missing angular wedge. FDK can produce previews, but serious reconstruction should use iterative regularized solvers.
3. **Offset detector / offset object**: the detector or rotation center is intentionally or accidentally off center, but complementary views over 360 degrees may still cover the full object. This needs geometry-aware reconstruction plus displaced-detector redundancy weighting.
4. **Truncated or interior scan**: some object regions are never fully measured. Projection completion and iterative regularization can reduce artifacts, but the problem is not uniquely solvable without assumptions.

Each Nikon scan is represented by a per-view geometry table:

```text
angle
source_xyz_mm
detector_center_xyz_mm
detector_u_xyz_mm
detector_v_xyz_mm
projection_weight
```

This preserves measured angles, detector offsets, row-dependent COR, and
asymmetric reconstruction volumes throughout reconstruction.

## Limited-Angle Strategy

### Short Scan

If coverage is at least about:

```text
180 degrees + 2 * fan_half_angle
```

then back-and-forth offers short-scan FDK:

- use the measured angle list;
- apply Parker or Parker-like redundancy weights;
- warns when angular range is below the short-scan requirement;
- uses ordinary full-scan FDK for complete 360-degree scans.

References:

- Parker, "Optimal short scan convolution reconstruction for fanbeam CT", Medical Physics, 1982, DOI [10.1118/1.595078](https://doi.org/10.1118/1.595078).
- ASTRA `FDK_CUDA` supports a `ShortScan` option that applies Parker weighting for non-360-degree data: [ASTRA FDK_CUDA docs](https://astra-toolbox.readthedocs.io/en/latest/docs/algs/FDK_CUDA.html).
- ODL documents Parker weighting as useful for fan/cone beam data to give each line unit total weight: [ODL Parker weighting](https://odlgroup.github.io/odl/generated/odl.tomo.analytic.filtered_back_projection.parker_weighting.html).

### True Limited Angle

If coverage is below the complete-data requirement:

- FDK remains available for a fast artifact-prone preview;
- output is labeled limited-angle with reduced quantitative confidence;
- CGLS, SIRT, SART/OS-SART, or TV-CP are suggested according to the desired
  data fidelity, runtime, and regularization assumptions.

References:

- Sidky and Pan, "Image reconstruction in circular cone-beam computed tomography by constrained, total-variation minimization", Physics in Medicine and Biology, 2008, DOI [10.1088/0031-9155/53/17/021](https://doi.org/10.1088/0031-9155/53/17/021).
- Sidky, Kao, Pan, "Accurate image reconstruction from few-views and limited-angle data in divergent-beam CT", X-Ray Science and Technology, 2006, DOI [10.3233/XST-2006-00155](https://doi.org/10.3233/XST-2006-00155).
- ASTRA supports GPU SIRT/SART/CGLS-style primitives for 2D/3D CT: [ASTRA docs](https://astra-toolbox.com/).
- TIGRE provides cone-beam FDK plus iterative and TV-family methods: [TIGRE docs](https://tigre.readthedocs.io/en/latest/d03_simple_image_reconstruction.html).

## Offset And Truncated Strategy

### Offset Detector / Recoverable Half-Acquisition

For 360-degree offset-detector scans, use:

- explicit detector offset in the geometry table;
- displaced-detector redundancy weighting;
- optional padding on the truncated side for FDK filtering;
- no padding for iterative methods unless explicitly requested.

References:

- Wang, "X-ray micro-CT with a displaced detector array", Medical Physics, 2002, DOI [10.1118/1.1489043](https://doi.org/10.1118/1.1489043).
- Cho, Rudd, Johnson, "Cone-beam CT from width-truncated projections", Computerized Medical Imaging and Graphics, 1996, DOI [10.1016/0895-6111(96)00031-6](https://doi.org/10.1016/0895-6111(96)00031-6).
- RTK `DisplacedDetectorImageFilter` implements weighting for off-centered panels and zero padding when needed: [RTK docs](https://www.openrtk.org/Doxygen/classrtk_1_1DisplacedDetectorImageFilter.html).

### Truncated Object / Interior ROI

For objects that exceed the detector in a way not fully recovered by complementary views:

- detect truncation from attenuation at left/right detector edges;
- report which sides and angular ranges are clipped;
- report that the data are truncated and assumption dependent;
- prefer iterative reconstruction and specimen-specific review for final output.

References:

- "Interior micro-CT with an offset detector" combines offset detector and interior truncation and compares weighting/completion/iterative approaches: [PMC4039736](https://pmc.ncbi.nlm.nih.gov/articles/PMC4039736/).
- Ohnesorge et al., "Efficient correction for CT image artifacts caused by objects extending outside the scan field of view", Medical Physics, 2000, DOI [10.1118/1.598855](https://doi.org/10.1118/1.598855).
- Chityala et al., "Artifact reduction in truncated CT using Sinogram completion", SPIE, 2005, DOI [10.1117/12.595450](https://doi.org/10.1117/12.595450).

## User Interpretation

The suggested route is advisory. Edge diagnostics measure attenuation near the
detector boundaries and are sensitive to noise, beam normalization, and the
selected threshold. Inspect projection previews and reconstruction-space crop
previews before accepting an offset/truncation classification.

Missing-angle streaks, elongated structures, and apparent smoothness from
regularization must not be interpreted as measured anatomy. Record the angular
coverage, selected algorithm, iteration/regularization settings, and warnings
with quantitative results.

See [limited-angle-sirt.md](limited-angle-sirt.md) for iterative equations and
[reconstruction-methods.md](reconstruction-methods.md) for method selection.

