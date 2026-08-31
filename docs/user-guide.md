# User Guide

## Starting a workflow

The startup chooser separates reconstruction from archival export:

- **Reconstruct from projections** shows the Scan, COR, Beam, Crop, Specimen,
  and Reconstruct tabs.
- **Load/Export existing VGI/VOL** shows only the existing-volume import,
  Specimen, and Distribution controls.

The guidance bar identifies the next recommended action. File and Settings menus
provide scan/job commands and local scratch configuration.

## Reconstructing from projections

1. Open the scan folder. Back-and-forth searches the selected folder and common
   Nikon `PROJ`/`INFO` layouts for TIFF projections, XTekCT, angle, and XML
   metadata. When more than one plausible metadata file exists, select the file
   describing the source projections rather than a prior reconstructed volume.
2. On **COR**, place the Z-plane handles in informative specimen regions. Use
   metadata COR directly when available or run Basic COR, then refine and review
   candidate slices. Manually confirm ambiguous planes before continuing.
3. On **Beam**, choose **None**, metadata coefficients, or a preset. Compare
   previews and use the weakest correction that removes cupping without erasing
   real contrast.
4. On **Crop**, wait for the low-resolution whole-volume MIP preview and drag the
   bounds in XY, XZ, and YZ. Crop coordinates are reconstruction-space bounds,
   not projection-image bounds.
5. On **Specimen**, enter archival specimen fields when Distribution output is
   required. Blank fields are permitted for ordinary reconstruction but trigger
   a warning before Distribution export.
6. On **Reconstruct**, review the suggested method, select an alternative when
   scientifically appropriate, choose the output options, and start. The phase
   label and progress bar distinguish projection preparation, iterative work,
   backprojection, writing, and archival export. **Abort** requests a clean stop.

Select **Create PLY isosurface** to open threshold selection after the verified
VGI/VOL has been written. The interactive view is a downsampled GPU ray-cast
preview; the saved binary PLY is extracted from the original full-resolution
float32 volume. **Auto Otsu** is a starting threshold only. The final corrected
MC33 mesh retains all components crossing the chosen isovalue, records the
threshold and coordinate provenance in its header, and is not smoothed.
Optionally choose a vertex-color mapping to sample attenuation at one inward
depth, or the mean/maximum over an inward depth interval. The preview includes
a color legend. The PLY stores both RGB and the unclipped float `ct_value` for
each valid vertex; depths are physical millimetres along the inward surface
normal.

See [reconstruction-methods.md](reconstruction-methods.md) for method-specific
strengths, limitations, memory behavior, and selection guidance.

### Correcting metal and pyrite artifacts

Metal artifact correction is optional and requires a reviewed segmentation:

1. Select **Meyer NMAR** only when the metal/pyrite trace is enclosed by valid
   specimen support in the prior projection.
2. Choose the two-class model for air plus one specimen material, or the
   three-class model only when distinct ordinary and dense nonmetal materials
   are actually present.
3. Run **Auto Guess**, inspect multiple Z slices, and adjust the initial-FDK
   metal threshold, tissue boundaries, and replacement value as needed.
4. Confirm that the separate metal mask contains the complete metal/pyrite and
   that the prior classes do not relabel real air as specimen.
5. Click **Confirm Segmentation** before reconstruction. Saved jobs preserve the
   selected method, thresholds, prior model, parameter source, and review state.

NMAR is not recommended when metal is exposed beside air or extends outside the
specimen prior. In that geometry, normalized interpolation can be undefined or
unstable because the prior projection approaches zero at a trace boundary. See
[nmar.md](nmar.md) for the exact method boundaries, parameter domains, and
recorded provenance.

## Exporting an existing Nikon reconstruction

Open the VGI directly or open the scan folder. In the common Nikon hierarchy,
the VGI/VOL is under `VOL/<scan_name>_01` while metadata may be under sibling
`INFO` and `PROJ` folders. Back-and-forth selects the highest numbered final
XTekCT file for archival export and asks for a file only when discovery is
ambiguous or incomplete.

Verify the volume dimensions, voxel size, XTekCT, XML files, full scan name, and
optional `CTEES_CT` filename insertion. Enter specimen metadata, render and
orient the thumbnail, then export.

**Create PLY Isosurface** is also available for the loaded VGI/VOL. It uses the
VGI position and resolution for physical millimetre coordinates and writes next
to the volume by default. The same optional surface-attached CT depth mapping is
available. PLY extraction requires a compatible NVIDIA CUDA GPU.

## Output locations

New reconstructions use `VOL/<scan_name>_01`, then `_02`, `_03`, and so on.
The folder contains the VGI, float32 VOL, and post-reconstruction XTekCT. When
enabled, `Distribution` contains:

- bottom-to-top uint16 XY TIFF slices and `scale.txt`;
- final XTekCT and available `ctprofile.xml`/`ctinfo.xml` source metadata;
- combined metadata JSON, README, and license;
- the selected volume-rendered thumbnail;
- a compressed Distribution ZIP and matching MD5 checksum.

The ZIP contains the TIFF stack, `scale.txt`, archival README and license,
combined metadata JSON, final XTekCT, and available source XML files. The
thumbnail remains beside the ZIP in the Distribution folder and is not added to
the archive.

See the [Metadata reference](metadata-reference.md) for field-by-field evidence
status, publicly supported meanings, units, parser/writer behavior, value
origins, and source citations for XTekCT, VGI/VOL, combined JSON, and copied
Nikon XML metadata. Proprietary fields without a public definition are listed
without an inferred meaning.

Distribution TIFFs use one linear mapping over the complete finite float32 VOL
range. `scale.txt` records that reversible mapping. A scan containing very dense
metal or pyrite can therefore look nearly uniform in an image viewer that opens
16-bit TIFFs without automatic windowing: the dense inclusions consume much of
the display range while ordinary specimen occupies fewer gray levels. This is a
display-window effect, not removal of the nonmetal attenuation values. Apply a
narrower display LUT for inspection and use `scale.txt` when recovering the
quantitative float values; do not infer reconstruction quality from the TIFF
viewer's initial window.

## Network storage

Use **Settings > Scratch** for scans on slow network storage. The persistent
local cache copies each required source file once, validates it by source size
and modification time, and reuses it across previews and reconstruction.
Outputs retain the original Nikon naming and destination rules. Ensure the local
scratch disk has enough free space for the selected workflow.

## Saved jobs

**Save Job** records the active scan, COR, beam correction, crop, specimen
metadata, reconstruction settings, and output choices. **Open Job** restores the
workflow and regenerates previews when cached images are unavailable. Always
review restored crop bounds and output destination before starting a long run.
