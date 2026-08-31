# Nikon Metadata Compatibility

For a field-by-field definition of the XTekCT, VGI/VOL, combined JSON, and
copied XML metadata, see the [Metadata reference](metadata-reference.md). That
reference labels each interpretation as a CIL reader interpretation, a
vendor-documented concept or field, an observation from paired Nikon outputs,
a preserved opaque value, or behavior defined by back-and-forth. A CIL
interpretation is not presented as Nikon's proprietary definition.

back-and-forth writes a Nikon-style post-reconstruction `.xtekct` beside each
VGI/VOL output. The file has two distinct responsibilities:

1. Nikon sections preserve acquisition and reconstructed-volume metadata in the
   form used by Nikon final XTekCT files.
2. `[BackAndForth]` records the exact reconstruction choices that do not have a
   documented Nikon field.

The second section is intentionally separate. It prevents software-specific
provenance from changing the meaning of Nikon keys.

## Fields read by CIL

CIL's `NikonDataReader` reads the following acquisition fields. The writer has
independent regression coverage for each one rather than relying on a
back-and-forth write/read round trip.

| CIL input | back-and-forth output rule |
| --- | --- |
| `Name` | Preserve the source scan name. |
| `InputFolderName` | Preserve the source value. This remains acquisition-location metadata; a final XTekCT in a `VOL` folder is not a relocated projection manifest. |
| `Projections` | Number of projection frames admitted to reconstruction, excluding a detected trailing return/check frame. |
| `WhiteLevel` | Preserve the detector white level. With `normalise=True`, CIL clamps loaded values below `1` to `1` and then divides by this value. |
| `DetectorPixelsX/Y` | Preserve detector dimensions. |
| `DetectorPixelSizeX/Y` | Preserve detector pitch. CIL uses it in the same physical coordinate system as source distances; back-and-forth interprets Nikon geometry values as millimetres. |
| `DetectorOffsetX/Y` | Preserve detector offsets. CIL describes them only as being "in units"; back-and-forth interprets Nikon geometry values as millimetres. |
| `SrcToObject`, `SrcToDetector` | Preserve source-to-axis and source-to-detector distances. CIL uses them directly in its geometry; back-and-forth interprets Nikon geometry values as millimetres. |
| `InitialAngle`, `AngularStep` | Preserve Nikon's nominal angular metadata. CIL converts these values into its own angular coordinate convention when loading. |
| `ObjectOffsetX` | Encode the selected rotation-axis offset evaluated at the detector row midpoint, converted to object-space millimetres by `SOD/SDD`. CIL places its axis at the negative of this value. |
| `ObjectRoll` | Encode the selected row-dependent COR slope as a physical in-plane rotation-axis angle, including detector-pixel aspect ratio. |
| `ObjectTilt` | Preserve a Nikon source value. back-and-forth currently warns and requires acknowledgement because ordinary reconstruction does not model a tilted rotation axis. |

CIL also supports the older `CentreOfRotationTop/Bottom` representation. Nikon
final reconstruction files use `ObjectOffsetX/ObjectRoll`, so back-and-forth
writes that final form and omits the obsolete pair.

The final XTekCT also preserves Nikon's nominal angle scalars even when a scan
contains a measured angle table. CIL intentionally ignores Nikon angle files.
For provenance, `[BackAndForth]` records whether measured or nominal angles were
used, their count and endpoints, measured coverage, and whether sampling was
irregular. The combined metadata JSON records the same sampling classification.

## Nikon reconstruction fields

These fields are not interpreted by CIL's projection reader. Their presence and
format are checked against supplied Nikon post-reconstruction XTekCT/VGI/VOL
examples. An example establishes interoperability behavior, not Nikon's
undocumented semantics.

- `VoxelsX/Y/Z` and `VoxelSizeX/Y/Z` describe the written float32 volume.
- `OffsetX/Y/Z` and VGI `position` are emitted using the relationship observed
  in paired Nikon outputs. A public Nikon semantic definition has not been
  located. The authoritative back-and-forth output center is recorded
  separately in `[BackAndForth]`.
- `Scaling` and `OutputUnits` are preserved Nikon fields whose public field
  definitions were not located. Back-and-forth recognizes inverse-metre unit
  spellings and applies the recorded positive scale to the output voxels; its
  exact interpretation is documented in the metadata reference.
- In a back-and-forth output, `CoefX0` through `CoefX4` and `Scale` record the
  exact beam-hardening polynomial it applied. This does not claim that the same
  fields' proprietary Nikon encoding has been publicly defined.
- `OutputName`, `OutputFolderName`, and incremented `_01`, `_02`, ... values use
  the naming pattern observed in Nikon final outputs; this does not define every
  vendor-side use of those fields.
- `VirtualDetector*` fields are written with observed-compatible values but are
  not semantically defined from examples alone. CTPro slice heights, X-ray
  settings, DICOM tags, filter material, shuttling, and flux-profile region are
  preserved where available.

`MaskRadius` is preserved from the source XTekCT. The CT Pro manual describes a
reconstruction-radius concept and explains that COR correction and shuttling
can reduce it, but it does not name or define the `MaskRadius` metadata key.
CIL does not read this field and back-and-forth does not use it to zero
reconstructed voxels or invent a replacement value.

Nikon's numeric `FilterType` code is retained from the source metadata because
its complete code mapping is not publicly documented. The actual analytical
filter and cutoff used by back-and-forth are authoritative in
`[BackAndForth] FdkFilter` and `FdkFilterCutoffFraction`.

## Exact reconstruction provenance

The `[BackAndForth]` section records the selected algorithm and backend,
iteration and stopping information, FDK filter and redundancy weighting,
beam-hardening and NMAR settings, the exact row-0 COR offset and slope, crop
geometry, projection step, angular-sampling facts, motion/tilt acknowledgements,
and software version. These values describe what was actually computed when a
legacy Nikon field cannot do so without ambiguity.

## References

- CIL 25.0.0 [`NikonDataReader` documentation](https://tomographicimaging.github.io/CIL/v25.0.0/io/#nikon)
  and pinned
  [`NikonDataReader.py`](https://github.com/TomographicImaging/CIL/blob/v25.0.0/Wrappers/Python/cil/io/NikonDataReader.py),
  including Nikon field parsing, white-level normalization, angle conversion,
  and `ObjectOffsetX/ObjectRoll` geometry construction.
- Nikon/Metris, [*CT Pro User Manual*](https://cns1.rc.fas.harvard.edu/facilities/docs/CT%20Pro%20User%20Manual.pdf),
  especially COR (manual pp. 11-18), setup/calibration (pp. 19-28), the worked
  VGI representation/file example (p. 31), volume and resolution (pp. 29-35),
  and file types (p. 45).
- Volume Graphics, [*Reference VGStudio MAX / myVGL 2.0*](https://research.amnh.org/~debel/tomo-aps/VGViewer/myvgl20.pdf),
  for general volume-import, voxel-resolution, data-range, and transfer-function
  concepts. It is not treated as a dictionary for undocumented legacy VGI
  tokens.
