# Normalized Metal Artifact Reduction (NMAR)

back-and-forth provides supervised normalized metal artifact reduction following
the normalized MAR sequence of Meyer et al. NMAR is optional and should be used
only when a reviewed material prior is scientifically defensible.

## Published Method Boundary

The production NMAR path is:

1. Reconstruct an uncorrected FDK volume.
2. Segment metal or pyrite in that quantitative reconstruction.
3. Forward-project the metal mask and linearly interpolate the measured
   line-integral projections across the metal trace.
4. Reconstruct the linearly corrected projections to form the MAR1 material
   image.
5. Smooth MAR1 with a fixed three-dimensional Gaussian (sigma one preview
   voxel) and classify it into a material prior.
6. Forward-project the reviewed prior and metal mask.
7. Divide each measured line integral by the prior projection, interpolate the
   normalized values across each supported metal trace, and multiply by the
   prior projection.
8. Filter and backproject the completed projections with the selected FDK
   filter, then retain the reconstructed high-density values in the result.

The CUDA path changes batching and storage, not these equations. Every selected
projection participates in the initial FDK, MAR1, prior projection, normalized
interpolation, and final FDK.

## Material Models

The two-class model contains air and one specimen class. Use it when a specimen
does not have a meaningful separate soft-material population. The recorded
specimen value is assigned to the specimen prior and replaces metal/pyrite in
that prior.

The three-class model contains air, ordinary or replacement specimen, and dense
nonmetal specimen. Ordinary specimen and metal replacement receive the recorded
replacement value; dense nonmetal voxels retain their MAR1 values.

Metal/pyrite is always represented by a separate binary mask. The `Prior
Classes` image is a class display, not the quantitative prior projection.

## Picking Domains

- `Pick Metal/Pyrite` samples the uncorrected initial reconstruction because the
  metal mask is defined from the high-density material before correction.
- Air, specimen/replacement, and dense-specimen picks sample the quantitative
  MAR1 material image used to construct the prior.
- Display-window or LUT changes do not change the sampled numerical values.
- The metal threshold defines the volume mask. Detector-trace dilation expands
  its forward projection only; it does not enlarge the displayed volume mask.

`Auto Guess` supplies starting values. It smooths MAR1 and performs a
deterministic 256-bin Otsu search for the two-class model or multi-Otsu search
for the three-class model. The user must inspect multiple slices and confirm the
segmentation before reconstruction.

## Review Checklist

Before selecting `Confirm Segmentation`, verify that:

- the metal mask includes the complete metal or pyrite and excludes dense
  nonmetal anatomy;
- air remains air, especially near the volume and specimen boundary;
- the two-class specimen region or all intended three-class regions are visible;
- the replacement value lies above the air threshold and below the metal
  threshold;
- the dense threshold, when used, lies between air and metal;
- several Z slices have been reviewed, including the slice containing the most
  metal and slices near the ends of the inclusion.

Air/specimen, soft/dense, prior-value, and material-model edits reclassify the
cached MAR1 volume without another reconstruction. Metal-threshold and
detector-dilation edits change MAR1 and require `Refresh MAR1`. Confirmation is
disabled until the displayed prior matches the controls.

## Progress And Reproducibility

The GUI reports the initial FDK, preliminary mask, MAR1 mask forward projection,
MAR1 interpolation/filtering/backprojection, smoothing, classification, final
prior and mask forward projections, final filtering, and final backprojection.
Network I/O and output publication are separate phases.

Saved jobs and output provenance record the correction method, metal/air/dense
thresholds, replacement value, prior model, detector-trace dilation, prior
epsilon, parameter source, and whether the material segmentation was confirmed.
This is sufficient to repeat the numerical NMAR configuration when the same
input projections and geometry are available.

## Limitations

NMAR depends on the correctness of its metal mask and prior. It can erase real
structure or introduce new streaks when either is wrong. Auto Guess is not a
scientific decision and does not remove the need for review.

The normalized interpolation requires valid, nonzero prior support at both ends
of each corrected detector trace. Metal exposed beside air or extending outside
the specimen can violate that condition. back-and-forth reports unsupported
runs; it does not manufacture specimen support in real air. There is currently
no production MAR method in back-and-forth for that geometry.

NMAR is substantially slower than one FDK because it includes multiple forward
projections and reconstructions. It does not correct scatter, motion, severe
truncation, inaccurate geometry, or an incorrect center of rotation.

## References

Meyer et al., "Normalized metal artifact reduction (NMAR) in computed
tomography", Medical Physics, 2010, DOI
[10.1118/1.3484090](https://doi.org/10.1118/1.3484090).

Otsu, "A Threshold Selection Method from Gray-Level Histograms", IEEE
Transactions on Systems, Man, and Cybernetics, 1979, DOI
[10.1109/TSMC.1979.4310076](https://doi.org/10.1109/TSMC.1979.4310076).
