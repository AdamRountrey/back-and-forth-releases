# Center of Rotation

Accurate center-of-rotation (COR) geometry is essential for microCT. Incorrect
COR commonly produces doubled edges, circular edge traces, blur, and loss of
fine structure.

## Recommended workflow

1. Place the two Z-plane handles in specimen regions with clear, detailed edges.
   The initial locations are near one-third and two-thirds of the useful height;
   either plane may be above the other.
2. Use the Nikon metadata COR when it is present and plausible. **Basic COR** is
   available when a new seed is needed.
3. Run **Refine COR**. Candidate slices are reconstructed at full output-plane
   resolution with all selected projections and the Ram-Lak filter.
4. Review every plane marked ambiguous or failed. Zoom and pan are retained while
   stepping through candidates. The plot marker identifies the displayed
   candidate.
5. Use **Extend Left** or **Extend Right** when the best visible alignment lies
   outside the sampled range. The extension is followed by local fine sampling
   and quadratic minimum fitting.
6. Apply the visually defensible candidate for each unresolved plane. The status
   changes to accepted only when all required planes have an accepted value.

The final result is a detector-row-dependent axis line:

```text
center(row) = center_at_row0 + slope * row
```

This line, rather than a display-only image shift, is passed to all later beam,
crop, FDK, iterative, and NMAR reconstruction stages.

In a reconstructed Nikon-style `.xtekct` sidecar, the same line is also encoded
in Nikon's final-form fields. `ObjectOffsetX` is the projected detector-center
COR offset converted back to object-space millimetres using magnification, and
`ObjectRoll` is the physical in-plane axis angle (including detector pixel
aspect ratio). The `[BackAndForth]` section retains the exact row-0 offset and
pixels-per-row slope used by reconstruction, together with method provenance.

## Automatic seeds

- Nikon `CentreOfRotationTop/Bottom` or final-form
  `ObjectOffsetX/ObjectRoll` metadata is the preferred baseline.
- Full and valid short scans without metadata use a Vacek/Jacobsen-style
  near-180-degree projection-pair phase-symmetry seed.
- True limited-angle and offset scans use metadata when available and otherwise
  use a projection-moment coarse seed that does not assume a 180-degree pair.

Seeds are starting estimates, not final acceptance evidence.

## Candidate scores

**Specimen-edge focus** is the default. It emphasizes gradient focus at the
segmented specimen boundary and penalizes background artifacts. Optional
specimen-buffer entropy, gradient-sharpness, and total-variation scores are
available as diagnostic alternatives. These scores are not substitutes for
visual inspection when minima are shallow or multiple candidates are similar.

An automatic result is accepted only when each sampled plane has a bracketed,
stable local optimum and the plane-wise centers agree with a single axis line.
Boundary optima, shallow valleys, multiple comparable minima, or inconsistent
planes require manual review. A failed refinement does not replace the active
COR.

## Scan-specific cautions

- Limited-angle candidate slices use the selected iterative reconstruction
  method rather than pretending FDK is complete.
- Offset/displaced-detector scans must retain the correct detector geometry;
  the rotation center need not be near the detector midpoint.
- Specimen motion can make different planes or projection intervals disagree.
  Motion QA is reported separately from COR because one cannot generally absorb
  arbitrary motion into a single COR value.
- Nonzero Nikon `ObjectTilt` describes a tilted rotation axis and is not modeled
  by the ordinary row-dependent COR line.

## References

- Vacek and Jacobsen, "Fast and noise-tolerant determination of the center of
  rotation in tomography," *Journal of Synchrotron Radiation* (2022), DOI
  [10.1107/S1600577521012777](https://doi.org/10.1107/S1600577521012777).
- Donath, Beckmann, and Schreyer, "Automated determination of the center of
  rotation in tomography data," *JOSA A* (2006), DOI
  [10.1364/JOSAA.23.001048](https://doi.org/10.1364/JOSAA.23.001048).
- Vo et al., "Reliable method for calculating the center of rotation in
  parallel-beam tomography," *Optics Express* (2014), DOI
  [10.1364/OE.22.019078](https://doi.org/10.1364/OE.22.019078). The Vo method is
  retained as a reference diagnostic, not the default final Nikon cone-beam COR
  decision.
- CIL's [NikonDataReader](https://github.com/TomographicImaging/CIL/blob/master/Wrappers/Python/cil/io/NikonDataReader.py)
  documents the Nikon COR metadata interpretation used by back-and-forth.
