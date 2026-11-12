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
   candidate slices. The two user-selected planes determine the axis; the
   intermediate (middle) plane is check-only. If axis refinement needs manual
   review, choose offsets on both selected planes. An inconclusive middle check
   needs **Confirm Intermediate Check**, which does not change the axis. See the
   [COR guide](center-of-rotation.md#intermediate-check-and-manual-review) for
   visual-review limitations and the explicit reconstruction override.
3. On **Beam**, choose **None**, metadata coefficients, or a preset. Compare
   previews and use the weakest correction that removes cupping without erasing
   real contrast.
4. On **Crop**, wait for the low-resolution whole-volume MIP preview and drag the
   bounds in XY, XZ, and YZ. Crop coordinates are reconstruction-space bounds,
   not projection-image bounds. Moving these bounds does not require another
   render: they are overlays on the full-volume MIP. Once the preview is ready,
   the suggested step advances to reconstruction (or NMAR review if enabled).
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
`INFO` and `PROJ` folders. Metadata can also be in the scan folder or the
volume folder's parent. Back-and-forth first matches the volume filename to
the XTekCT; when no exact match exists, numbered reconstruction files are
considered.

The existing-volume workflow accepts headerless, little-endian float32,
unsigned 16-bit, and explicitly signed 16-bit VOL data. The stored type and
dimensions come from VGI `[file1]`, not a possibly different display
`[representation]`. See the distinction in the
[CT Pro manual, p. 28](https://cns1.rc.fas.harvard.edu/facilities/docs/CT%20Pro%20User%20Manual.pdf#page=32).
Preview, TIFF export, and PLY extraction use the stored voxel values. No display
LUT or inferred attenuation calibration is applied. Integer data is read in
bounded chunks without creating a full float32 volume copy. The original VOL
and Nikon metadata files are not modified.

Before copying the volume to scratch, the importer compares all three voxel
counts and voxel spacings against the VGI. If the proposed XTekCT conflicts or
cannot be verified, a chooser shows the nearby candidates and field-by-field
comparisons. A unique compatible numbered candidate from the same scan-name
family is suggested first; otherwise a unique compatible candidate is
suggested. You must confirm the choice. **Browse** can select a file outside
the searched folders, and that file is checked too.

Renaming a VGI/VOL to remove `_01` does not make the unnumbered acquisition
XTekCT authoritative. Even if its grid matches, the importer asks for
confirmation when numbered counterparts exist. Several reconstructions can
have identical grids but different COR or processing settings: geometry
checks cannot prove which one produced the volume. Choose the original
post-reconstruction file. Cancel leaves export unavailable until the metadata
is resolved. Source filenames and file contents are not changed.

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

Distribution TIFFs preserve imported unsigned 16-bit VOL values without
rescaling (`SCALE=1`, `OFFSET=0`). Signed 16-bit inputs use a fixed offset of
32768 (`SCALE=1`, `OFFSET=32768`) without range stretching; this exactly maps
-32768...32767 to 0...65535. Float32 inputs use one linear mapping over the
complete finite stored VOL range to unsigned 16-bit TIFFs, with rounding.
`scale.txt` records the mapping from source values. The required
origin annotation in TIFF 0000 still replaces pixels under its text; it is not
part of that intensity mapping. The mapping does not recover any calibration
lost in an earlier conversion to integer. A scan containing very dense
metal or pyrite can therefore look nearly uniform in an image viewer that opens
16-bit TIFFs without automatic windowing: the dense inclusions consume much of
the display range while ordinary specimen occupies fewer gray levels. This is a
display-window effect, not removal of the nonmetal attenuation values. Apply a
narrower display LUT for inspection and use `scale.txt` when recovering the
source values (within rounding precision); do not infer reconstruction quality from the TIFF
viewer's initial window.

## Network storage

Use **Settings > Local Scratch Cache...** for scans on slow network storage.
**Use local scratch for datasets opened from network storage** is enabled by
default. The dialog also provides **Scratch location**, a usage display, and
**Clear Source Cache**. Source files are staged per dataset and reused across
previews and reconstruction while source and cached-file size/modification-time
checks pass against the manifest. Interrupted copies are not reusable.
Outputs retain the original Nikon naming and destination rules.

**Final output** is the published VGI destination, not a scratch filename.
Default names come from the original acquisition XTekCT, even when its working
copy uses a short cache name: `VOL/<scan>_01/<scan>_01.vgi`, advancing the suffix
when that output already exists.

For a network reconstruction destination, the VOL, VGI, QA, and sidecars are
written and verified in local scratch first. The completed set is then copied
to destination-side staging with byte-level progress; the VGI is published
last. A failed transfer retains the completed local set at the recovery path
shown in the error message. Successful completion removes this temporary set.
Recovery files also survive application restart; copy them to a safe location
and remove them manually when no longer needed.
The local VOL is also reused for a requested Distribution export instead of
rereading it across the network. Local output destinations still use nearby
transaction staging rather than making an extra scratch copy.

Network reconstruction requires scratch space for one float32 volume plus
512 MiB of sidecar/preview headroom, in addition to solver work and any requested
Distribution workspace. Scratch capacity and destination staging paths are
checked before reconstruction; network or filesystem failures can still occur
later. The final network copy can still be slow.

Normal source staging does not hash every TIFF or reread copied files. A manifest
marks each copy complete only after successful writing, size checks, and a check
that the source size/modification time did not change during copying. Reuse checks
sizes and timestamps, not file contents; it cannot detect corruption that leaves
both unchanged. Older completed cache manifests remain usable. Distribution ZIP
checksums are separate and are still produced for download-integrity verification.

**Keep local source copies after the workflow completes** is off by default.
With it off, the application releases its source-cache leases and attempts
cleanup:

- after a verified reconstruction completes, provided Distribution was not
  requested or its export also completed;
- after a successful Distribution export from an existing VGI/VOL;
- when a different dataset replaces the active one in the same workflow;
- on normal application close, for caches used during that session.

A failed or cancelled operation does not trigger successful-completion cleanup;
in particular, an incomplete requested Distribution export keeps the source
cache available for retry until replacement or close. Enabling the keep option
retains copies for later reuse instead of these automatic cleanup attempts.
Cleanup is best-effort, not a guarantee that the scratch location becomes empty.

Both automatic cleanup and **Clear Source Cache** remove only files owned by a
valid per-dataset application manifest, their recognized `.partial` copies, and
the manifest files. They do not recursively delete the selected scratch root.
Unknown files, unrecognized folders, original scans, reconstruction outputs,
and Distribution recovery packages are retained. Invalid manifests and unsafe
traversal, symbolic-link, or reparse paths are refused.

An OS-backed lease protects each dataset cache while it is in use, including
against deletion by another application process. Clear is disabled while the
current session holds source copies; another process's lease can also make a
clear attempt fail. Leases must be released before deletion. The zero-byte
`.bf-source-cache.lock` marker deliberately remains in each dataset directory
after cleanup to prevent lock-file replacement races; its presence alone does
not mean a lease is active.

The dialog's **Current source cache** number currently counts all regular files
under the selected scratch location, including retained recovery or unrelated
files. It is not a measurement of only the manifest-owned files eligible for
deletion, so it can remain nonzero after clearing.

When the Distribution destination is on network storage, back-and-forth writes
and verifies the TIFF stack, metadata, and compressed ZIP in local scratch
first. It then publishes the completed files to a destination-side transaction
with byte-level progress. Existing destination files are replaced only after
the complete transfer succeeds. If publication fails, the prior destination
package remains intact and the completed local package is retained at the
recovery path shown in the error message. The scratch preflight reserves space
for the uncompressed TIFF stack and the ZIP, so ensure the selected local
scratch disk has enough free space before export.

## Saved jobs

**Save Job** records the active scan, COR, beam correction, crop, specimen
metadata, reconstruction settings, and output choices. **Open Job** restores the
workflow and regenerates previews when cached images are unavailable. Always
review restored crop bounds and output destination before starting a long run.
