# Metadata Reference

This is the data dictionary for metadata read, preserved, calculated, and
written by back-and-forth. It covers the post-reconstruction XTekCT, VGI/VOL,
Distribution combined metadata JSON, and copied Nikon XML files.

## Evidence labels

Each table identifies the authority for its interpretation:

- **CIL 25**: the pinned CIL 25.0.0 `NikonDataReader` documentation or source.
  This establishes how CIL interprets a key, not necessarily Nikon's
  proprietary intent.
- **Nikon manual**: the Metris/Nikon CT Pro User Manual. A workflow concept in
  the manual is not treated as the definition of a similarly named metadata
  key unless the manual identifies that key.
- **VG manual**: the Volume Graphics myVGL/VGStudio reference manual. This is
  used for documented format concepts only; it does not define every legacy
  VGI token written by Nikon.
- **Nikon example**: field presence, value pattern, or formatting observed in
  supplied Nikon source and post-reconstruction file pairs. This label alone
  does not define the field's semantics. A table entry with no independent
  authority therefore records only the observation and back-and-forth's literal
  output rule.
- **Preserved**: copied from source Nikon metadata without assigning a new
  interpretation.
- **back-and-forth**: a field defined by this program to record what it actually
  computed.

Nikon's public manual explains the reconstruction concepts and workflow, but it
does not define every XTekCT or XML key. CIL supplies a public, executable
interpretation of the Nikon acquisition fields it reads. Where neither source
defines a field, back-and-forth preserves the value or labels it as proprietary
instead of guessing.

In particular, names that sound self-explanatory are not treated as vendor
definitions. `Offset`, `VirtualDetector`, and legacy VGI scene fields remain
listed for interoperability even where Nikon's intended semantics are not
publicly documented.

## Coordinate and value conventions

| Convention | Meaning | Authority |
| --- | --- | --- |
| Detector X/Y | Detector column and row coordinates. CIL uses detector pitch, offsets, and source distances in one consistent physical coordinate system but does not read the XTekCT `Units` key. Back-and-forth interprets Nikon physical geometry values as millimetres; fields explicitly labeled as pixels remain pixels. | CIL 25; back-and-forth |
| Object X/Y/Z | Physical reconstruction coordinates in millimetres. The output grid center and size are recorded explicitly in `[BackAndForth]`. | back-and-forth |
| XTekCT `OffsetX/Y/Z` | Observed in Nikon final files, but a public Nikon semantic definition has not been located. For interoperability, back-and-forth writes its output-center X/Y values and the negative of its output-center Z value. The unambiguous internal center remains `OutputCenterXMM/YMM/ZMM`. | Nikon example; back-and-forth |
| VGI `position` | Observed in Nikon VGI files, but its Nikon-specific relationship to XTekCT is not publicly defined. Back-and-forth writes its XTekCT `Offset` values divided by source voxel spacing, including the written Z sign. Treat `OutputCenterXMM/YMM/ZMM`, not this compatibility field, as authoritative reconstruction provenance. | Nikon example; back-and-forth |
| COR detector offset | Horizontal detector offset in pixels. A slope gives change in that offset per detector row. | back-and-forth |
| Nikon final COR | CIL consumes `ObjectOffsetX` in the same physical coordinate system as the rest of the geometry and places the rotation axis at its negative; it consumes `ObjectRoll` in degrees. Back-and-forth writes `ObjectOffsetX` in object-space millimetres. | CIL 25; back-and-forth |
| Linear attenuation | Back-and-forth treats reconstructed float values as linear attenuation. Its writer applies the recorded `Scaling` when `OutputUnits` is an inverse-metre spelling such as `/m`; this is documented program behavior, not a claimed public Nikon definition of those keys. | Nikon example; back-and-forth |
| Angles | Nikon nominal angles are recorded in degrees. `[BackAndForth]` states whether reconstruction used those scalars or a measured angle table. | CIL 25; back-and-forth |

## Post-reconstruction XTekCT

### Identity and file naming

| Field | Meaning and output rule | Source |
| --- | --- | --- |
| `Name` | Original scan or dataset name. | CIL 25; Preserved |
| `OperatorID` | Preserved source field. A public Nikon semantic definition has not been located. | Preserved |
| `InputSeparator`, `OutputSeparator` | Preserved source naming fields. Back-and-forth does not assign additional vendor semantics. | Preserved |
| `InputFolderName` | Original projection-folder metadata. It is not rewritten to point at the final `VOL` folder. | CIL 25; Preserved |
| `OutputFolderName` | Observed Nikon final-output field. Back-and-forth writes its incremented output root, such as `<scan>_01`; no broader Nikon semantic claim is made. | Nikon example; back-and-forth |
| `InputName`, `OutputName` | Observed Nikon naming fields. Back-and-forth preserves/writes the corresponding input and output roots without assigning additional vendor semantics. | Nikon example; Preserved; back-and-forth |
| `InputDigits`, `OutputDigits` | Observed Nikon naming fields whose source values are preserved; their Nikon-side behavior is not defined here. | Nikon example; Preserved |
| `ModifierID` | Preserved source field. A public Nikon semantic definition has not been located. | Preserved |

### Reconstructed volume grid

| Field | Meaning and output rule | Source |
| --- | --- | --- |
| `VoxelsX`, `VoxelsY`, `VoxelsZ` | Number of float32 voxels written by back-and-forth in each output dimension. | back-and-forth |
| `VoxelSizeX`, `VoxelSizeY`, `VoxelSizeZ` | Back-and-forth output voxel spacing in millimetres, calculated as output physical size divided by voxel count. The exact Nikon key semantics are not inferred beyond that writer behavior. | Nikon example; back-and-forth |
| `OffsetX`, `OffsetY` | Nikon final-output fields with no located public semantic definition. Back-and-forth writes `OutputCenterXMM` and `OutputCenterYMM` for interoperability. | Nikon example; back-and-forth |
| `OffsetZ` | Nikon final-output field with no located public semantic definition. Back-and-forth writes `-OutputCenterZMM`, matching the supplied paired outputs. | Nikon example; back-and-forth |
| `Units` | Preserved source length-unit label; back-and-forth defaults it to `mm` when absent. CIL 25.0.0 does not read this key. | Preserved; back-and-forth |
| `Scaling` | Preserved source-compatible scalar. Back-and-forth uses its positive value as the output-voxel multiplier when `OutputUnits` is an inverse-metre spelling, defaulting to `1000` in that case if no positive value is available. Public Nikon documentation located for this audit does not define the key. | Nikon example; back-and-forth |
| `OutputUnits` | Preserved output-unit string. Back-and-forth recognizes `/m`, `1/m`, `m^-1`, and `m-1` as inverse-metre spellings for its output scaling rule. Public Nikon documentation located for this audit does not define the key. | Nikon example; back-and-forth |
| `OutputType`, `ImportConversion`, `AutoScalingType` | Nikon output/import scaling mode codes. Their complete proprietary enum mappings are not assigned by back-and-forth. | Preserved |
| `ScalingMinimum`, `ScalingMaximum` | Preserved Nikon fields. Their exact vendor-side behavior is not publicly defined; they are not used as the measured VGI/VOL data range by back-and-forth. | Preserved; back-and-forth |
| `LowPercentile`, `HighPercentile` | Preserved Nikon fields with no located public definition of their vendor-side behavior. | Preserved |

### Acquisition geometry and detector

| Field | Meaning and output rule | Source |
| --- | --- | --- |
| `SrcToObject` | X-ray source-to-rotation-axis distance. CIL uses the numeric value directly in its geometry; back-and-forth interprets Nikon scan geometry distances as millimetres. Often called SOD. | CIL 25; back-and-forth |
| `SrcToDetector` | X-ray source-to-detector distance. CIL uses the numeric value directly in its geometry; back-and-forth interprets Nikon scan geometry distances as millimetres. Often called SDD. | CIL 25; back-and-forth |
| `DetectorPixelsX`, `DetectorPixelsY` | Physical detector pixel counts. | CIL 25 |
| `DetectorPixelSizeX`, `DetectorPixelSizeY` | Detector pixel pitch. CIL uses these values in the same physical coordinate system as the source distances; back-and-forth interprets Nikon pitch values as millimetres. | CIL 25; back-and-forth |
| `DetectorOffsetX`, `DetectorOffsetY` | Physical detector offsets. CIL describes these only as being "in units" and uses them in the same coordinate system as detector pitch; back-and-forth interprets Nikon values as millimetres. | CIL 25; back-and-forth |
| `RegionStartX`, `RegionStartY`, `RegionPixelsX`, `RegionPixelsY` | Preserved Nikon detector-region fields. Their names and values are retained, but no undocumented vendor behavior is inferred. | Preserved |
| `MaskRadius` | Preserved Nikon field. The CT Pro manual documents a reconstruction-radius concept but does not identify this key, so no field-level meaning or unit is asserted. Back-and-forth does not use it to zero voxels or recalculate it. | Preserved; back-and-forth |
| `Projections` | CIL interprets this as the projection count. Back-and-forth writes the number of frames admitted to reconstruction, excluding a detected trailing return/motion-check frame. | CIL 25; back-and-forth |
| `InitialAngle`, `AngularStep` | Nominal Nikon angular start and step in degrees. These remain nominal even when reconstruction uses a measured angle table. | CIL 25; Preserved |
| `WhiteLevel` | CIL's detector white-level value. With `normalise=True`, CIL first clamps loaded values below `1` to `1`, then divides by `WhiteLevel`. | CIL 25 |
| `Increment` | Nikon projection increment parameter. The actual projection stride used by back-and-forth is `ProjectionStep` in its own section. | Preserved; back-and-forth |

### Corrections, filtering, and sampling

| Field | Meaning and output rule | Source |
| --- | --- | --- |
| `InterpolationType` | Nikon detector interpolation mode code. The public CT Pro manual describes nearest-pixel and interpolated backprojection, but the numeric enum is preserved rather than redefined. | Nikon manual; Preserved |
| `BeamHardeningLUTFile` | Preserved Nikon field. Back-and-forth does not interpret or load it as evidence that a correction ran. | Preserved; back-and-forth |
| `CoefX0`, `CoefX1`, `CoefX2`, `CoefX3`, `CoefX4` | Observed Nikon beam-hardening fields whose public vendor encoding was not located. For a back-and-forth reconstruction, these are the exact polynomial coefficients that back-and-forth applied to projection line integrals. | Nikon example; back-and-forth |
| `Scale` | Observed Nikon beam-hardening field whose public vendor encoding was not located. For a back-and-forth reconstruction, this is the exact output scale used with the recorded polynomial. | Nikon example; back-and-forth |
| `FilterType` | Nikon numeric analytical-filter code. The complete mapping is not publicly documented, so the source code is preserved. The actual back-and-forth filter is `FdkFilter`. | Preserved; back-and-forth |
| `CutOffFrequency`, `Exponent` | Nikon filter-shape parameters. Preserved for compatibility; `FdkFilterCutoffFraction` records the actual back-and-forth cutoff. | Preserved; back-and-forth |
| `Normalisation`, `Scattering` | Nikon correction parameters. Their source values are preserved and are not evidence that an equivalent back-and-forth correction ran. | Preserved |
| `MedianFilterKernelSize`, `ConvolutionKernelSize` | Preserved Nikon fields; exact vendor-side behavior is not assigned from their names. | Preserved |
| `Blanking`, `OrderFFT` | Preserved Nikon fields; exact vendor-side behavior is not publicly defined here. | Preserved |

### Virtual detector and final COR

| Field | Meaning and output rule | Source |
| --- | --- | --- |
| `VirtualDetectorPixelsX`, `VirtualDetectorPixelsY` | Observed Nikon final-output fields. Their vendor semantics are not publicly defined; back-and-forth writes the physical detector pixel counts for compatibility. | Nikon example; back-and-forth |
| `VirtualDetectorPixelSizeX`, `VirtualDetectorPixelSizeY` | Observed Nikon final-output fields. Their vendor semantics are not publicly defined; back-and-forth writes the physical detector pitches. | Nikon example; back-and-forth |
| `VirtualDetectorOffsetX`, `VirtualDetectorOffsetY` | Observed Nikon final-output fields. Their vendor semantics are not publicly defined; back-and-forth writes the physical detector offsets. | Nikon example; back-and-forth |
| `SrcToVirtualDetector` | Observed Nikon final-output field. Its vendor semantics are not publicly defined; back-and-forth writes `SrcToDetector`. | Nikon example; back-and-forth |
| `AutomaticCentreOfRotation` | Observed Nikon final-output field with no located public enum definition. Back-and-forth writes `0`; actual COR method and review state are recorded separately. | Nikon example; back-and-forth |
| `ObjectOffsetX` | CIL interprets this in the geometry's physical units as the rotation-axis horizontal position and places the axis at its negative. Back-and-forth writes the selected detector-center offset at the detector row midpoint, converted by SOD/SDD magnification to object-space millimetres. | CIL 25; Nikon example; back-and-forth |
| `ObjectOffsetY` | Observed Nikon final-output field whose vendor semantics are not publicly defined by the sources used here. Back-and-forth writes `0` for its supported geometry. | Nikon example; back-and-forth |
| `ObjectRoll` | CIL interprets this as rotation-axis roll in degrees. Back-and-forth writes its selected row-dependent COR slope converted with detector pixel aspect ratio. | CIL 25; Nikon example; back-and-forth |
| `ObjectTilt` | CIL interprets this as a rotation-axis tilt in degrees about X. Back-and-forth preserves the source value; ordinary reconstruction does not model this geometry and requires user acknowledgement when nonzero. | CIL 25; Preserved; back-and-forth |
| `CentreOfRotationTop/Bottom` | Older Nikon COR representation that CIL explicitly interprets as millimetres at the detector. Back-and-forth reads it as a seed when present but final files use `ObjectOffsetX/ObjectRoll`. | CIL 25; back-and-forth |

### Other Nikon sections

| Section and field | Meaning and output rule | Source |
| --- | --- | --- |
| `[Xrays]`: `XraykV`, `XrayuA` | Preserved Nikon fields. Back-and-forth parses them as tube potential in kilovolts and tube current in microamperes; no separate public Nikon key definition was located. | Preserved; back-and-forth |
| `[Xrays]`: `FluxNormalisation.Profile.Region` | Observed Nikon field. When source metadata explicitly enables flux normalization, back-and-forth interprets the value as the detector rectangle for its per-projection flux reference and records that use separately. | Nikon example; back-and-forth |
| `[DICOM]`: `DICOMTags` | Opaque Nikon DICOM-tag XML payload. It is preserved without semantic rewriting. | Preserved |
| `[AutoBeamHardening]`: `Stack`, `Slice`, `SinogramOffsetX`, `SinogramBandSampling`, `SliceThreshold`, `SliceAreaStartX`, `SliceAreaStartY`, `SliceAreaEndX`, `SliceAreaEndY` | Observed Nikon fields whose exact vendor semantics are not publicly defined. Source values are preserved when present; compatibility defaults may otherwise be written. They do not override the actual polynomial recorded above and in `[BackAndForth]`. | Preserved; Nikon example |
| `[CTPro]`: `Version`, `Product` | Source Nikon CT Pro software identification. | Preserved |
| `[CTPro]`: `Filter_ThicknessMM`, `Filter_Material` | Preserved Nikon fields. Back-and-forth parses them as prefilter thickness in millimetres and material text; no separate public Nikon key definition was located. | Preserved; back-and-forth |
| `[CTPro]`: `Shuttling` | Preserved Nikon field associated with the CT Pro shuttling workflow described by the vendor manual. | Nikon manual; Preserved |
| `[CTPro]`: `AutoCOR_NumBands`, `CorAutoAccuracy` | Preserved Nikon fields associated by name and location with automatic COR. Their exact enum/range semantics are not defined here and they do not describe the back-and-forth COR method. | Preserved |
| `[CTPro]`: `SliceHeightMMSingle`, `SliceHeightMMDualTop`, `SliceHeightMMDualBottom` | Observed Nikon fields not named or defined in the public CT Pro manual. Back-and-forth writes the source physical-height center, upper-quarter, and lower-quarter values as compatibility metadata; this rule does not claim Nikon's proprietary interpretation. | Nikon example; back-and-forth |
| `[CTPro]`: `AngleFile_Use`, `AngleFile_IgnoreErrors` | Preserved Nikon fields whose proprietary behavior is not assigned from their names. Actual angle provenance used by back-and-forth is recorded in `[BackAndForth]`. | Preserved; back-and-forth |

## `[BackAndForth]` reconstruction provenance

These fields are defined by back-and-forth. Optional fields are omitted when a
value was not applicable or was not captured. They are the authoritative record
of choices made by back-and-forth when a Nikon field is ambiguous or only
preserved for compatibility.

### Software and route

| Field(s) | Meaning |
| --- | --- |
| `Product`, `SoftwareVersion` | Program name and released software version. |
| `ProvenanceStatus` | `recorded` when a reconstruction provenance record was supplied; otherwise `not-recorded`. |
| `ReconstructionRoute` | High-level reconstruction path selected after scan classification. |
| `Algorithm` | Complete user-facing method label for the executed reconstruction. |
| `Backend` | `CPU` or `CUDA`. |
| `Streaming` | Whether the reconstruction used the bounded-memory streaming path. |
| `IterativeAlgorithm` | Exact iterative solver selection, when applicable. |
| `ProjectorDiscretization` | Forward/backprojector discretization used by the iterative solver. |
| `ConvergenceOrStoppingStatus` | Recorded completion, convergence, stopping, or abort status. |

### Iterative controls

| Field(s) | Meaning |
| --- | --- |
| `Iterations`, `CompletedIterations` | Requested and actually completed outer iterations. |
| `Relaxation` | Iterative update relaxation factor. |
| `OrderedSubsetSize` | Number of projection views in each ordered subset. |
| `NonNegativity` | Whether negative voxel values were projected to zero. |
| `NormalizedSirt` | Whether SIRT row/column normalization was enabled. |
| `DataToleranceRmse` | Data-residual RMSE stopping tolerance. |
| `TvWeight` | Total-variation regularization weight. |
| `TvPrimalStep`, `TvDualStep` | Requested Chambolle-Pock primal and dual steps, before automatic norm-based reduction. Older outputs with only these fields do not establish the effective steps. |
| `TvExtrapolation` | Chambolle-Pock extrapolation parameter, theta. |
| `TvEffectivePrimalStep`, `TvEffectiveDualStep` | Steps actually used by the solver, captured during execution. |
| `TvOperatorNormSquaredEstimate` | Power-iteration estimate of the squared norm of the combined data and gradient operator, including `TvNormSafetyFactor`. An estimate, not a certified upper bound. |
| `TvNormEstimator` | `power-iteration-AtA-plus-gradient-adjoint-gradient`: iteration on the data normal operator plus the finite-difference gradient normal operator. |
| `TvNormIterations`, `TvNormMaxIterations` | Actual and maximum power iterations (maximum currently 24). |
| `TvNormRelativeTolerance` | Rayleigh-value convergence tolerance. After at least five iterations, stop when the change is at most this value times `max(1, abs(current Rayleigh value))`. |
| `TvNormSafetyFactor` | Multiplier on the final Rayleigh estimate (currently 1.25). This numerical precaution does not prove that the true operator norm is bounded. |
| `TvPreparationBackend` | Backend used for FDK backprojection and norm estimation: `cpu` or `cuda`. TIFF loading and host staging still use the CPU. |
| `TvInitialization` | `fdk`: initial volume is FDK, with negatives clamped only when `NonNegativity` is enabled. |
| `TvInitializationFilter`, `TvInitializationFilterCutoffFraction`, `TvInitializationRedundancyWeight` | Filter, cutoff, and requested redundancy policy for the FDK initializer, distinct from the iterative objective. `auto` denotes automatic redundancy selection from scan geometry. |
| `TvGradientSteps` | Number of inner TV-gradient updates where used by the selected method. |
| `PocsBeta`, `PocsBetaReduction` | ASD-POCS projection update and reduction factor. |
| `AsdAlpha`, `AsdAlphaReduction`, `AsdRmax` | ASD-POCS steepest-descent step, reduction factor, and update-ratio control. |
| `TpvP` | TpV exponent parameter. |
| `TpvPriorIterations`, `CompletedTpvPriorIterations` | Requested and completed prior-forming iterations. |
| `TpvGradientBeta`, `TpvDataBeta`, `TpvMultiplierStep` | TpV-GIF optimization parameters. |
| `TpvErrorTolerance` | TpV-GIF stopping tolerance. |
| `GuidedFilterRadius`, `GuidedFilterEpsilon` | Guided-filter window radius and regularization epsilon. |

The exact equations and publication boundaries for these methods are documented
in [Reconstruction methods](reconstruction-methods.md) and
[Algorithm citations](citations.md).

### FDK, beam hardening, and metal correction

| Field(s) | Meaning |
| --- | --- |
| `FdkFilter` | Actual analytical filter, such as Ram-Lak or Hann. |
| `FdkFilterCutoffFraction` | Filter cutoff as a fraction of detector Nyquist frequency. |
| `FdkRedundancyWeight` | Angular or displaced-detector redundancy weighting selected for FDK. |
| `BeamHardeningApplied` | Whether the applied polynomial differs from the identity transform. |
| `BeamHardeningCoefX0`, `BeamHardeningCoefX1`, `BeamHardeningCoefX2`, `BeamHardeningCoefX3`, `BeamHardeningCoefX4`, `BeamHardeningScale` | Exact polynomial and scale used on projection line integrals. |
| `BeamHardeningUserSelected` | Whether the user explicitly selected the correction rather than inheriting metadata/default behavior. |
| `DenseInclusionCorrection`, `DenseInclusionThreshold`, `DenseInclusionDilationPixels` | Legacy dense-inclusion correction mode and segmentation controls, when that mode is present. |
| `NmarApplied` | Whether Meyer NMAR was applied. |
| `NmarMetalThreshold`, `NmarAirThreshold`, `NmarBoneThreshold`, `NmarSoftTissueValue`, `NmarPriorEpsilon` | Recorded NMAR material thresholds, replacement value, and numerical prior floor in the native reconstructed attenuation domain. |
| `NmarPriorModel` | Selected two- or three-class NMAR prior model. |
| `NmarMaterialParameterSource` | Whether values came from automatic estimation, user picks, or another recorded source. |
| `NmarMaterialSegmentationReviewed` | Whether the user confirmed the NMAR segmentation. |
| `MarMaterialMetalThreshold`, `MarMaterialAirThreshold`, `MarMaterialDenseThreshold`, `MarMaterialReplacementValue` | Current material-segmentation thresholds and metal replacement value used to construct the prior. |
| `MarPriorModel`, `MarMaterialParameterSource`, `MarMaterialSegmentationReviewed` | Current prior model, parameter source, and user-review state. |

See [Metal artifact correction](nmar.md) for the threshold domain and method
limitations. A recorded threshold is not evidence that segmentation was valid;
the review flag must also be considered.

### COR and geometry safeguards

| Field(s) | Meaning |
| --- | --- |
| `CorMethod` | Seed/refinement/review method that produced the accepted rotation axis. |
| `CorDetectorCenterOffsetPixels` | Accepted horizontal detector-center offset at detector row zero, in detector pixels. |
| `CorDetectorCenterOffsetSlopePerDetectorRow` | Change in accepted detector-center offset per detector row, in pixels per row. |
| `CorManualReviewOverride` | Whether reconstruction proceeded through an explicit user override of automatic COR confidence. |
| `CorUnresolvedPlanesAtOverride` | Plane identifiers still unresolved when that override was accepted. |
| `MotionCorrectionApplied` | Whether the optional projection-motion correction was actually applied. |
| `SourceObjectTiltDegrees` | Nonzero source Nikon `ObjectTilt`, in degrees. |
| `SourceObjectTiltGeometryApplied` | Whether reconstruction geometry modeled that tilt. Current ordinary paths record `false`. |
| `SourceObjectTiltAcknowledged` | Whether the user acknowledged the unsupported tilted-axis condition. |
| `CroppedProjectionModelUsed` | Whether a cropped/truncated projection model was used. |
| `CroppedProjectionModelAcknowledged` | Whether the user acknowledged the associated model limitation. |

### Angles, output grid, and flux normalization

| Field(s) | Meaning |
| --- | --- |
| `ProjectionAnglesSource` | `measured-angle-table` or `nominal-xtekct`. |
| `ProjectionAnglesCount` | Number of angles admitted to reconstruction. |
| `ProjectionAnglesFirstDegrees`, `ProjectionAnglesLastDegrees` | First and last angles actually associated with admitted projections. |
| `MeasuredAngularCoverageDegrees` | Measured angular span used for scan classification. |
| `AngularSamplingIrregular` | Whether the admitted angle intervals depart materially from uniform sampling. |
| `ProjectionStep` | Projection subsampling stride used for this reconstruction. A production full-data run normally records `1`. |
| `OutputVoxelsX`, `OutputVoxelsY`, `OutputVoxelsZ` | Output voxel dimensions, repeated here as explicit provenance. |
| `OutputCenterXMM`, `OutputCenterYMM`, `OutputCenterZMM` | Internal physical center of the requested output grid in millimetres. |
| `OutputSizeXMM`, `OutputSizeYMM`, `OutputSizeZMM` | Physical output extent in millimetres. |
| `SourceFluxNormalisationUsed` | Present and `true` only when source metadata enabled Nikon profile-region flux normalization. |
| `SourceFluxNormalisationProfileRegion` | Exact source detector region used to estimate per-projection incident flux. |

## VGI and VOL

Back-and-forth reconstruction writes a headerless, contiguous float32 `.vol`
voxel array. Existing-volume import also supports little-endian unsigned and
explicitly signed 16-bit integer arrays. The `.vgi`
describes its shape, values, physical placement, and display defaults.

On import, `[file1]` takes precedence for stored dimensions, scalar type, bit
depth, and data range. `[representation]` is a fallback for omitted dimensions
and type. A display range is not substituted for a missing stored range when
`[file1]` declares its own type;
a partially specified file type (only datatype or only bit depth) is rejected.
This precedence follows the CT Pro manual's distinction between actual file
data and display representation, not a conversion between the two [1]. A
float32 VOL may legitimately have an unsigned-16 display representation.
Byte count must match the stored type and dimensions exactly. Unsupported
types, headers, and non-raw storage are rejected. Integer samples are converted
exactly to float for processing; they are not rescaled or assigned attenuation
units merely because the source XTekCT contains output-unit fields.

| VGI field | Meaning | Source |
| --- | --- | --- |
| `[representation]` `size` | Volume dimensions X Y Z. This exact legacy VGI field appears in the CT Pro manual's worked VGI example. | Nikon manual; back-and-forth |
| `datatype`, `bitsperelement` | Representation scalar type and bit depth. The CT Pro manual distinguishes representation values from the actual file values; back-and-forth writes 32-bit float for both. | Nikon manual; back-and-forth |
| `datarange` | Representation minimum and maximum. Back-and-forth writes the finite minimum and maximum observed in the VOL. | Nikon manual; back-and-forth |
| `resamplemode`, `mirror` | Legacy VGI fields observed in Nikon output. Back-and-forth writes the same inactive/zero literals; no Nikon-specific semantic interpretation is asserted. | Nikon example; back-and-forth |
| `[file1] Name` | VOL filename referenced by the VGI. This exact field appears in the CT Pro manual's worked VGI example. | Nikon manual; back-and-forth |
| `FileFormat` | Back-and-forth writes `raw`; the CT Pro manual shows this literal in its worked VGI example. The VOL layout itself is verified by the back-and-forth writer as headerless float32. | Nikon manual; back-and-forth |
| `Size`, `Datatype`, `BitsPerElement`, `datarange` | Actual file dimensions, scalar type, bit depth, and range. The CT Pro manual explicitly distinguishes these file values from `[representation]`. | Nikon manual; back-and-forth |
| `RegionOfInterestStart`, `RegionOfInterestEnd` | Exact legacy VGI fields shown in the CT Pro manual. Back-and-forth writes inclusive `0` through `size-1` voxel bounds; no broader interpretation is asserted. | Nikon manual; back-and-forth |
| `[section1]`: `min`, `max` | Observed Nikon VGI fields. Back-and-forth writes the finite VOL data minimum and maximum. | Nikon example; back-and-forth |
| `[section1]`: `offset`, `name` | Observed legacy VGI fields. Back-and-forth writes compatibility literals and does not assign Nikon semantics. | Nikon example; back-and-forth |
| `[geometry]`: `status`, `relativeposition` | Observed legacy VGI fields. Back-and-forth writes visible status and zero relative displacement; no field-level public definition was located. | Nikon example; back-and-forth |
| `[geometry] position` | Observed Nikon VGI field with no located public Nikon mapping to XTekCT. Back-and-forth writes `OffsetX/vx OffsetY/vy OffsetZ/vz` for compatibility. The authoritative output center is in `[BackAndForth]`. | Nikon example; back-and-forth |
| `resolution` | Back-and-forth writes X/Y/Z voxel spacing in millimetres. The VG manual independently defines imported X/Y/Z resolution as voxel size, but does not document this exact legacy token. | VG manual; Nikon example; back-and-forth |
| `scale` | Observed legacy VGI field. Back-and-forth writes `1 1 1`; no field-level public definition was located. | Nikon example; back-and-forth |
| `center` | Observed legacy VGI field. Back-and-forth writes half the dimensions; no field-level public definition was located. | Nikon example; back-and-forth |
| `rotate` | Observed legacy VGI field. Back-and-forth writes the identity matrix; no field-level public definition was located. | Nikon example; back-and-forth |
| `unit` | Observed legacy VGI field. Back-and-forth writes `mm`; no field-level public definition was located. | Nikon example; back-and-forth |
| `[volume]`: `volume` | Observed legacy VGI link field written by back-and-forth; no field-level public definition was located. | Nikon example; back-and-forth |
| `[description]`: `text` | Back-and-forth writes the human-readable volume name in this observed VGI field. | Nikon example; back-and-forth |
| `[segment1]` transfer fields | Back-and-forth writes these as initial grayscale opacity/color display defaults. The VG manual documents transfer-function concepts, but not these exact legacy tokens. They do not alter VOL values. | VG manual; Nikon example; back-and-forth |
| `{default}`, `{scene}`, `{camera1}` | Observed legacy VGStudio scene, camera, light, and rendering fields written for convenient opening. They are not scientific acquisition or reconstruction metadata, and no exact public token dictionary was located. | Nikon example; back-and-forth |

The display-only VGI terms are included for completeness:

| VGI display field(s) | Meaning |
| --- | --- |
| `[segment1] status`, `description` | Initial transfer-function activation and legacy segment identifier. |
| `opacityx`, `opacityvalue` | Scalar control points and corresponding initial opacity values. |
| `redx`, `greenx`, `bluex`; `redvalue`, `greenvalue`, `bluevalue` | Scalar control points and channel values for the initial grayscale transfer function. |
| `[version]`: `release` | Legacy VGStudio version string used by Nikon-compatible examples. It is not the back-and-forth version. |
| `[viewer]`: `perspective`, `viewing angle` or `viewingangle` | Initial perspective state and camera field of view. |
| `[viewer] position`, `up`, `lookat` | Initial camera position, up vector, and look-at point. These do not change volume coordinates. |
| `[viewer]`: `oversampling`, `resultsize` | Initial display sampling and image dimensions. |
| `[perspective] status` | Initial perspective-projection activation. |
| `[light1]` and `[light2]`: `diffuse`, `ambient`, `status` | Initial scene-light colors and activation states. |
| `[angle] position`, `status`, `right`, `left` | Legacy scene angle-marker settings. |
| `[resolution] resolution`, `unit`, `displayunit` | Initial scene resolution display value and units. The authoritative per-axis voxel spacing remains `[geometry] resolution`. |
| `[rendering]`: `background`, `intensity`, `colormode`, `quickmode`, `algorithm` | Initial viewer rendering preferences. They do not alter stored attenuation values. |

## Distribution combined metadata JSON

The combined JSON is an archival schema used by the supplied UMMP Distribution
examples. It is not a Nikon or CIL standard. Back-and-forth preserves the
historical field names and orientation string for compatibility.

| JSON field | Meaning and source |
| --- | --- |
| `Institution` | User-entered institution. |
| `SpecimenCatalogNumber` | User-entered specimen catalog identifier. |
| `SpecimenUUID` | User-entered persistent specimen UUID. |
| `SpecimenTaxon` | User-entered taxonomic identification. |
| `Element(s)` | User-entered anatomical or object description. |
| `DistributionFileRootName` | Root used to name archival TIFF, JSON, ZIP, and checksum outputs. |
| `DistributionFilenamesIncludeCTEESCT` | Whether `_CTEES_CT` was inserted into applicable archival filenames. |
| `ScannerXML` | Structural JSON conversion of `ctprofile.xml`. XML attributes become `@name` keys and repeated children become arrays. Empty/absent XML values remain empty or null. |
| `CTPro3D` | Structural JSON conversion of the final post-reconstruction XTekCT, including its `[BackAndForth]` section. Values remain the actual XTekCT values rather than being reinterpreted. |
| `ConversionTo16bit` | Object containing the global stored-VOL-value-to-uint16 TIFF conversion record. |
| `ConversionTo16bit.SourceStorageType`, `InputValueDomain` | Present for imported integer VOLs: `uint16` or `int16`, and an explicit statement that input means stored VOL values without inferred attenuation calibration. Float32 exports retain the existing archival wording and fields. |
| `ConversionFunction` | The recorded TIFF mapping: `16output = SCALE * (32input + OFFSET)` for float32 sources; `16output = SCALE * (input + OFFSET)` for imported integer sources. |
| `SCALE`, `OFFSET` | Unsigned 16-bit VOL values are preserved with `SCALE=1`, `OFFSET=0`. Signed 16-bit VOLs use `SCALE=1`, `OFFSET=32768`: a fixed shift without range stretching. Float32 VOLs use one global linear mapping from their complete finite stored range to uint16. Recover source values with `input = output / SCALE - OFFSET` (exact for integer inputs, subject to rounding for float32); pixels replaced by the required origin annotation in TIFF 0000 are excluded. This does not reconstruct any unknown earlier attenuation calibration. |
| `TIFOrientation` | Object containing the historical archive orientation record. |
| `XYZOrigin` | Historical archive convention value `upper_left_firstImage`. Its wording and value are retained for compatibility with existing deposits. |
| `MethodString` | Human-readable reconstruction and export method summary. |
| `DescriptionString` | Human-readable specimen/scan description. |
| `License` | Distribution data-license text or identifier. This is separate from the software license. |

## Nikon XML files

`ctprofile.xml` and `ctinfo.xml` are copied into the Distribution output when
available. `ctprofile.xml` is also converted structurally into `ScannerXML`.
Typical records include tube settings, detector/imaging settings, shading and
flux-normalization profiles, manipulator positions, projection counts, scan
limits, and reconstruction presets.

The following `ctprofile.xml` terms are consumed directly by back-and-forth.
Their descriptions state the parser's use of each value, not an unpublished
Nikon schema definition:

| XML element or attribute | Meaning and use | Source |
| --- | --- | --- |
| `FramesPerProjection` | Parsed by back-and-forth as the detector-frame count associated with each stored projection. | Preserved Nikon XML; back-and-forth |
| `ImagingSettings@exposure` | Parsed by back-and-forth as exposure time in milliseconds. | Preserved Nikon XML; back-and-forth |
| `HelicalScan` | Parsed as a helical-acquisition flag. Helical geometry is not supported by the ordinary circular cone-beam paths. | Preserved Nikon XML; back-and-forth |
| `LimitedAngleScan` | Parsed as a limited-angle flag and used with measured angular coverage for route selection. | Preserved Nikon XML; back-and-forth |
| `FluxNormalisation` | Parsed as the source flag controlling back-and-forth's per-projection profile-region flux normalization. | Preserved Nikon XML; back-and-forth |
| `FluxNormalisationRect/X`, `Y`, `Width`, `Height` | Parsed as the detector rectangle used by back-and-forth to estimate incident flux for each projection when normalization is enabled. | Preserved Nikon XML; back-and-forth |
| `XStart`, `XEnd`, `YStart`, `YEnd`, `ZStart`, `ZEnd` | Parsed as inclusive source volume-of-interest bounds used to initialize the ROI. | Preserved Nikon XML; back-and-forth |
| `kV`, `uA` | Parsed as tube potential in kilovolts and current in microamperes. | Preserved Nikon XML; back-and-forth |

Other common elements such as `ManipulatorPosition`, `AxisPosition`,
`CTAxisOffset`, `FocusMode`, `XrayHead`, `ShadingCorrectionProfile`,
`GreyLevelTargets`, `WhiteTargetLevel`, `ReconstructionOutput`,
`BeamHardeningPreset`, and `NoiseReductionPreset` are retained in the original
XML and structural JSON. Their exact coordinate systems, enum values, and
vendor-side behavior are not inferred unless a public Nikon source defines
them.

These XML schemas and numeric enum codes are not fully documented publicly.
Back-and-forth therefore preserves the original files and XML structure. An
unknown XML element or numeric code must be treated as opaque Nikon metadata,
not as a back-and-forth scientific interpretation. `ctinfo.xml` is copied as a
source record but is not duplicated inside the combined JSON.

## Existing-volume compatibility checks

These are back-and-forth import checks, not additional Nikon field definitions
or a published provenance-identification algorithm. The importer compares
`VoxelsX/Y/Z` to VGI `file1.Size` (or `representation.size` when omitted) exactly, without axis permutations,
and compares `VoxelSizeX/Y/Z` to VGI `geometry.resolution * geometry.scale`
per axis. It follows the application's millimetre interpretation above; an
explicit XTekCT `Units` other than `mm` cannot be verified by this check.
Missing, nonfinite, nonpositive or malformed geometry is not a match.

Spacing comparisons allow the larger of `1e-9 mm` and `5e-6` times the larger
spacing on that axis, to accommodate rounded decimal VGI values. This is an
application tolerance, not a claimed Nikon specification. Known mismatches
and unverifiable metadata block archival export until resolved.

The chooser uses filename relationships to suggest compatible candidates,
but never infers COR, beam-hardening settings, attenuation scaling, specimen
identity or reconstruction provenance from a grid match. In particular, VGI
`position` is not compared to XTekCT `OffsetX/Y/Z` to select a file: those
fields lack a sufficiently established common semantic definition. Selection
does not rewrite the selected XTekCT or XML files.

## References

1. Core Imaging Library 25.0.0,
   [`NikonDataReader` documentation](https://tomographicimaging.github.io/CIL/v25.0.0/io/#nikon)
   and pinned
   [`NikonDataReader.py` source](https://github.com/TomographicImaging/CIL/blob/v25.0.0/Wrappers/Python/cil/io/NikonDataReader.py).
   These are the authority for CIL's interpretation of the Nikon acquisition
   geometry, white-level normalization, angle conversion, and COR fields.
2. Metris/Nikon, [*CT Pro User Manual*](https://cns1.rc.fas.harvard.edu/facilities/docs/CT%20Pro%20User%20Manual.pdf),
   especially Centre of Rotation (manual pp. 11-18), Setup and Calibration
   (pp. 19-28), the worked VGI representation/file example (p. 31), Volume and
   reconstruction resolution (pp. 29-35), and File Types (p. 45).
3. Volume Graphics, [*Reference VGStudio MAX / myVGL 2.0*](https://research.amnh.org/~debel/tomo-aps/VGViewer/myvgl20.pdf),
   especially volume import, voxel resolution, scalar data-range mapping, and
   transfer-function concepts. The manual does not provide a field dictionary
   for the legacy VGI scene tokens written by Nikon.
4. Supplied paired Nikon source and post-reconstruction XTekCT/VGI/VOL examples,
   used as interoperability fixtures in the back-and-forth metadata tests.
