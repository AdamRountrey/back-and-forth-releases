# PLY Isosurface Export

back-and-forth can extract a measurement-oriented triangle isosurface from a
reconstructed float32 VGI/VOL volume. The exported surface is not smoothed or
regularized.

## User Workflow

1. Open a reconstructed volume or select `Create PLY isosurface` before a new
   reconstruction.
2. Inspect the GPU ray-cast preview. Two-class Otsu thresholding supplies a
   reproducible first guess; adjust the attenuation isovalue when needed.
3. Select `Create PLY`. The final mesh is calculated from the original
   full-resolution float32 VOL, not from the downsampled preview.

The **Vertex colors** menu optionally attaches CT measurements to that surface:

- **Attenuation at inward depth** samples one physical depth below each vertex.
- **Mean attenuation over inward interval** averages uniformly sampled values
  between two physical depths.
- **Maximum attenuation over inward interval** records the largest sampled
  value in that interval.

Depths are in millimetres and follow the negative exported vertex normal. The
preview displays the same normal-direction definition on the sampled volume;
the saved values are recalculated from the original full-resolution float32
VOL. `Auto Range` only selects display colors. It does not clip or rescale the
saved attenuation measurement.

Otsu's threshold is an initial estimate, not a material calibration or an
automatically validated specimen boundary. Every connected component crossing
the selected isovalue is retained, including detached pieces and enclosed
cavities.

## Extraction Method

The final extractor uses the corrected Marching Cubes 33 method of Vega,
Abache, and Coll (2019), including their corrected interior test. The decision
logic and 2,310-entry lookup table are pinned to MC33++ 5.4 commit
`252dabd7ea6dea3d0529e66b047adbe878caf2fa`. This avoids the ambiguous-face and
interior-connectivity errors possible with the original 256-case Marching
Cubes table.

CUDA performs case classification, prefix scans, vertex-key sorting, welding,
linear edge interpolation, and spacing-aware normal calculation. The volume is
streamed in bounded Z slabs with overlap. Every voxel cell is evaluated once,
and vertices shared by adjacent slabs retain one global index. Changing slab
depth therefore does not change the extracted surface.

Surface crossings are localized at subvoxel coordinates: for each intersected
grid edge, the vertex is placed at
`t = (threshold - value0) / (value1 - value0)` between the two voxel centres.
This removes voxel-face quantization and preserves fractional physical
coordinates. It is not image super-resolution; localization accuracy remains
limited by the reconstructed sampling, point-spread function, noise, partial
volume effects, and the selected attenuation threshold.

PLY coordinates are millimetres. VGI `position` is treated as the corner of
the volume; the first scalar sample is half a voxel inward on every axis. VGI
resolution supplies the X, Y, and Z spacing. The binary PLY header records the
threshold, dimensions, spacing, origin, software version, algorithm, and pinned
reference commit. If an exact-isovalue intersection collapses an MC33 face to
zero area, that unsupported face is omitted and the number removed is recorded
in the header. Vertex normals point from higher attenuation toward lower
attenuation.

## Surface-Attached CT Mapping

Surface-attached measurements use spacing-aware normals and trilinear
interpolation in the original physical VGI coordinate system. Point mapping
stores one sample. Interval mapping aims for a spacing no larger than half the
smallest voxel dimension and uses at most 64 evenly spaced samples; the actual
sample count is recorded in the PLY header. A mapped vertex is valid only when
every requested trilinear sample and its interpolation support are finite and
inside the volume. Invalid vertices retain `NaN` as `ct_value`, receive a
neutral gray color, and set `mapping_valid` to zero.

Mapped binary PLY vertices contain:

```text
x y z nx ny nz ct_value red green blue mapping_valid
```

`ct_value` remains in the reconstruction's native float attenuation units.
RGB is a visualization using a documented nine-knot linear approximation to
the viridis color map and the user-selected color range. Values outside that
range are color-clipped, but `ct_value` is not changed. PLY header comments
record the mapping statistic, depth interval, sample count, color range, and
valid/invalid vertex counts.

These measurements are straight profiles along local surface normals. They do
not classify materials, estimate geodesic depth, or guarantee that a profile
remains inside a thin or highly concave specimen. Interpretation remains
limited by the reconstruction point-spread function, artifacts, partial-volume
effects, surface threshold, and local normal accuracy.

## Validation

The automated validation compares the back-and-forth decision path against the
official MC33++ CPU implementation over every nontrivial sign case with
randomized magnitudes. It checks triangle count, topology, and winding. A
closed anisotropic analytic sphere additionally verifies official-reference
mesh counts and area, slab-depth invariance, two-manifold edge incidence,
physical scale and origin, nonzero triangles, valid indices, outward normals,
cancellation, and transactional publication. An exact-isovalue regression case
also verifies that collapsed zero-area faces are not emitted. The analytic
sphere carries a known radial scalar field that checks the sign and physical
scale of point, interval-mean, and interval-maximum depth mapping, PLY
scalar/color layout, validity flags, and invariant surface geometry.

## References

- Otsu, N. (1979). A Threshold Selection Method from Gray-Level Histograms.
  IEEE Transactions on Systems, Man, and Cybernetics, 9(1), 62-66.
- Custodio, L., Etiene, T., Pesco, S., and Silva, C. (2013). Practical
  considerations on Marching Cubes 33 topological correctness. Computers &
  Graphics, 37(7), 840-850. https://doi.org/10.1016/j.cag.2013.04.004
- Vega, D., Abache, J., and Coll, D. (2019). A Fast and Memory Saving Marching
  Cubes 33 implementation with the correct interior test. Journal of Computer
  Graphics Techniques, 8(3), 1-17.
  https://jcgt.org/published/0008/03/01/
- Liu, B., Clapworthy, G. J., Dong, F., and Wu, E. (2016). Parallel Marching
  Blocks: A Practical Isosurfacing Algorithm for Large Data on Many-Core
  Architectures. Computer Graphics Forum, 35(3), 501-510.
  https://doi.org/10.1111/cgf.12897. This is engineering background for the
  bounded GPU execution strategy; back-and-forth retains the separate MC33++
  case-selection and indexed-welding path described above.
