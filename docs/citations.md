# Algorithm Citations and Method Boundaries

This page links algorithm names shown by back-and-forth to their primary
publications and states where an available option differs from a literal
published formulation.

## Metadata Formats and Interpretation

- Nikon XTekCT acquisition geometry and normalization are cross-checked against
  CIL 25.0.0's
  [`NikonDataReader` documentation](https://tomographicimaging.github.io/CIL/v25.0.0/io/#nikon)
  and pinned
  [`NikonDataReader.py` source](https://github.com/TomographicImaging/CIL/blob/v25.0.0/Wrappers/Python/cil/io/NikonDataReader.py).
- Nikon reconstruction-workflow terms are referenced to the Metris/Nikon
  [*CT Pro User Manual*](https://cns1.rc.fas.harvard.edu/facilities/docs/CT%20Pro%20User%20Manual.pdf),
  particularly its COR, setup/calibration, volume, resolution, and file-type
  sections.
- General VGI volume-import, voxel-resolution, scalar-range, and transfer-function
  concepts are referenced to Volume Graphics,
  [*Reference VGStudio MAX / myVGL 2.0*](https://research.amnh.org/~debel/tomo-aps/VGViewer/myvgl20.pdf).
  Undocumented legacy VGI tokens are identified as observed compatibility fields,
  not assigned meanings from their names.
- The complete source-traced field dictionary is in
  [Metadata reference](metadata-reference.md). Fields not publicly defined by
  these sources are explicitly marked as preserved proprietary metadata or as
  back-and-forth-defined provenance.

## Production Reconstruction

- Feldkamp-Davis-Kress cone-beam reconstruction:
  Feldkamp, Davis, and Kress, "Practical cone-beam algorithm", JOSA A, 1984,
  DOI [10.1364/JOSAA.1.000612](https://doi.org/10.1364/JOSAA.1.000612).
  back-and-forth uses this as the standard circular cone-beam FDK path, with the
  usual circular-trajectory limitation for off-midplane exactness.
- Ramp/Ram-Lak filtering:
  Ramachandran and Lakshminarayanan, "Three-dimensional reconstruction from
  radiographs and electron micrographs: application of convolutions instead of
  Fourier transforms", PNAS, 1971, DOI
  [10.1073/pnas.68.9.2236](https://doi.org/10.1073/pnas.68.9.2236).
- Windowed FDK filters:
  Shepp and Logan, "The Fourier reconstruction of a head section", IEEE
  Transactions on Nuclear Science, 1974, DOI
  [10.1109/TNS.1974.6499235](https://doi.org/10.1109/TNS.1974.6499235).
  Hann, Hamming, and cosine filters are conventional windowed-ramp variants
  used to trade spatial resolution for noise suppression; see Kak and Slaney for
  filtered-backprojection background and Harris, "On the use of windows for
  harmonic analysis with the discrete Fourier transform", Proceedings of the
  IEEE, 1978, DOI
  [10.1109/PROC.1978.10837](https://doi.org/10.1109/PROC.1978.10837).
- Short-scan weighting:
  Parker, "Optimal short scan convolution reconstruction for fanbeam CT",
  Medical Physics, 1982, DOI
  [10.1118/1.595078](https://doi.org/10.1118/1.595078). The cone-beam
  implementation follows the standard Parker-style angular redundancy weighting
  used by FDK toolkits for short circular scans.
- Displaced-detector/offset reconstruction:
  Wang, "X-ray micro-CT with a displaced detector array", Medical Physics, 2002,
  DOI [10.1118/1.1489043](https://doi.org/10.1118/1.1489043). Offset-scan FDK
  uses displaced-detector redundancy weighting when the scan is classified as
  recoverable from complementary views.

## Iterative Reconstruction

- SIRT normalization:
  Gilbert, "Iterative methods for the three-dimensional reconstruction of an
  object from projections", Journal of Theoretical Biology, 1972, DOI
  [10.1016/0022-5193(72)90180-4](https://doi.org/10.1016/0022-5193(72)90180-4).
  back-and-forth uses row/column-normalized SIRT updates in the measured
  cone-beam geometry.
- Few-view/limited-angle divergent-beam CT:
  Sidky, Kao, and Pan, "Accurate image reconstruction from few-views and
  limited-angle data in divergent-beam CT", Journal of X-Ray Science and
  Technology, 2006, DOI
  [10.3233/XST-2006-00155](https://doi.org/10.3233/XST-2006-00155).
- SART:
  Andersen and Kak, "Simultaneous algebraic reconstruction technique (SART): a
  superior implementation of the ART algorithm", Ultrasonic Imaging, 1984, DOI
  [10.1177/016173468400600107](https://doi.org/10.1177/016173468400600107).
- OS-SART:
  Wang and Jiang, "Ordered-subset simultaneous algebraic reconstruction
  techniques (OS-SART)", Journal of X-Ray Science and Technology, 2004, DOI
  [10.3233/XST-2004-00110](https://doi.org/10.3233/XST-2004-00110).
- CGLS/CGNR least-squares reconstruction:
  Hestenes and Stiefel, "Methods of conjugate gradients for solving linear
  systems", Journal of Research of the National Bureau of Standards, 1952, DOI
  [10.6028/jres.049.044](https://doi.org/10.6028/jres.049.044), and Bjorck,
  *Numerical Methods for Least Squares Problems*, SIAM, 1996, DOI
  [10.1137/1.9781611971484](https://doi.org/10.1137/1.9781611971484).
  The option labeled CGLS is CGLS/CGNR on the explicit cone-beam CT operator.
- TV Chambolle-Pock:
  Chambolle and Pock, "A first-order primal-dual algorithm for convex problems
  with applications to imaging", Journal of Mathematical Imaging and Vision,
  2011, DOI [10.1007/s10851-010-0251-1](https://doi.org/10.1007/s10851-010-0251-1),
  with isotropic TV from Rudin, Osher, and Fatemi, "Nonlinear total variation
  based noise removal algorithms", Physica D, 1992, DOI
  [10.1016/0167-2789(92)90242-F](https://doi.org/10.1016/0167-2789(92)90242-F).
- ASD-POCS ART-TV:
  Sidky and Pan, "Image reconstruction in circular cone-beam computed tomography
  by constrained, total-variation minimization", Physics in Medicine and Biology,
  2008, DOI [10.1088/0031-9155/53/17/021](https://doi.org/10.1088/0031-9155/53/17/021).
  The GUI labels this CPU-only option experimental.

## Projectors and Geometry

- Joseph ray-driven interpolation:
  Joseph, "An improved algorithm for reprojecting rays through pixel images",
  IEEE Transactions on Medical Imaging, 1982, DOI
  [10.1109/TMI.1982.4307572](https://doi.org/10.1109/TMI.1982.4307572).
- General CT reconstruction background and notation:
  Kak and Slaney, *Principles of Computerized Tomographic Imaging*, IEEE Press,
  1988; electronic copy at [slaney.org/pct](https://slaney.org/pct/).
- Cone-beam data completeness:
  Tuy, "An inversion formula for cone-beam reconstruction", SIAM Journal on
  Applied Mathematics, 1983, DOI
  [10.1137/0143035](https://doi.org/10.1137/0143035). This is the reason the
  documentation marks circular FDK as an accepted approximate method rather than
  an exact general 3D cone-beam inverse.

## Center of Rotation

- Projection-pair phase-symmetry seed:
  Vacek and Jacobsen, "Fast and noise-tolerant determination of the center of
  rotation in tomography", Journal of Synchrotron Radiation, 2022, DOI
  [10.1107/S1600577521012777](https://doi.org/10.1107/S1600577521012777).
- Sinogram Fourier double-wedge method:
  Vo et al., "Reliable method for calculating the center of rotation in
  parallel-beam tomography", Optics Express, 2014, DOI
  [10.1364/OE.22.019078](https://doi.org/10.1364/OE.22.019078). This is kept as
  a fallback/reference path and is not the default final decision path for Nikon
  cone-beam micro-CT.
- Reconstruction-space COR review/diagnostics:
  Candidate FDK slices are scored and shown for user review. The specimen-edge
  focus and specimen-buffer variants are image-derived focus metrics computed
  from reconstruction-space candidate images. Their names describe what they
  measure; they are not presented as named published algorithms.
- Coarse fallback COR seeds:
  Projection-pair mirrored-error and projection-moment sinusoid seeds are used
  only when stronger metadata or phase-symmetry seeds are unavailable. They
  provide starting points for reconstruction-space review in coarse,
  limited-angle, and offset workflows; they are not final acceptance evidence
  or named published COR algorithms.
- Entropy-style COR diagnostics:
  Donath, Beckmann, and Schreyer, "Automated determination of the center of
  rotation in tomography data", JOSA A, 2006, DOI
  [10.1364/JOSAA.23.001048](https://doi.org/10.1364/JOSAA.23.001048). TomoPy and
  related tomography packages use reconstruction-artifact and entropy-style
  center searches. Back-and-forth keeps entropy as an optional diagnostic
  metric; specimen-edge focus is the default.

## Truncation, Projection Completion, and Interior ROI

- Width-truncated cone-beam CT:
  Cho, Rudd, and Johnson, "Cone-beam CT from width-truncated projections",
  Computerized Medical Imaging and Graphics, 1996, DOI
  [10.1016/0895-6111(96)00031-6](https://doi.org/10.1016/0895-6111(96)00031-6).
- Object-outside-FOV correction:
  Ohnesorge et al., "Efficient correction for CT image artifacts caused by
  objects extending outside the scan field of view", Medical Physics, 2000, DOI
  [10.1118/1.598855](https://doi.org/10.1118/1.598855).
- Sinogram completion for truncated CT:
  Chityala et al., "Artifact reduction in truncated CT using sinogram
  completion", SPIE Medical Imaging, 2005, DOI
  [10.1117/12.595450](https://doi.org/10.1117/12.595450).
- Interior micro-CT with offset detector:
  "Interior micro-CT with an offset detector" compares displaced-detector
  weighting, completion, and iterative strategies for interior/truncated
  micro-CT: [PMC4039736](https://pmc.ncbi.nlm.nih.gov/articles/PMC4039736/).

## Beam Hardening and Dense-Inclusion Correction

- Polynomial beam-hardening correction:
  Van de Casteele et al., "A model-based correction method for beam hardening
  artefacts in X-ray microtomography", Journal of X-Ray Science and Technology,
  2004, DOI [10.3233/XST-2004-00100](https://doi.org/10.3233/XST-2004-00100).
  Dewulf et al., "Sense and non-sense of beam hardening correction in CT
  metrology", CIRP Annals, 2012, DOI
  [10.1016/j.cirp.2012.03.013](https://doi.org/10.1016/j.cirp.2012.03.013).
  Davis, "Development of two-dimensional beam hardening correction for X-ray
  micro-CT", Journal of X-Ray Science and Technology, 2022, DOI
  [10.3233/XST-221178](https://doi.org/10.3233/XST-221178).
  Hammersberg and Mangard, "Correction for beam hardening artefacts in
  computerised tomography", Journal of X-Ray Science and Technology, 1998, DOI
  [10.3233/XST-1998-00024](https://doi.org/10.3233/XST-1998-00024). The CT Pro
  preset ladder in the GUI is vendor-workflow/preset-derived rather than a set
  of material constants; a Measurement 2024 paper reproduces CT Pro 3D-style
  polynomial shape coefficients used as the preview ladder, DOI
  [10.1016/j.measurement.2024.114414](https://doi.org/10.1016/j.measurement.2024.114414).
- NMAR:
  Meyer et al., "Normalized metal artifact reduction (NMAR) in computed
  tomography", Medical Physics, 2010, DOI
  [10.1118/1.3484090](https://doi.org/10.1118/1.3484090). back-and-forth follows
  the NMAR sequence: initial reconstruction, dense mask, trace interpolation,
  MAR1 material image, segmented prior, prior normalization, interpolation in
  the normalized sinogram, denormalization, final reconstruction, and dense-value
  reinsertion.
- Gaussian-splat polychromatic reconstruction (research implementation, not a
  production GUI option):
  Choi et al., "Splat-based Metal Artifact Reduction in Cone-Beam CT via
  Polychromatic Modeling", Computer Graphics Forum, 2026, DOI
  [10.1111/cgf.70339](https://doi.org/10.1111/cgf.70339). The authors'
  [reference repository](https://github.com/KAIST-VCLAB/ct-metal-reduction) is
  used for equation-level and released-configuration cross-checks. Its
  separately licensed rasterizer is not copied or vendored by back-and-forth.
- Gaussian-splat CUDA acceleration reference:
  Pieta et al., "FaCT-GS: Fast and Scalable CT Reconstruction with Gaussian
  Splatting", arXiv:2604.01844, 2026,
  [paper](https://arxiv.org/abs/2604.01844) and
  [reference repository](https://github.com/PaPieta/fact-gs). Back-and-forth
  independently implements compatible density-aware rectangular bounds,
  order-independent X-ray accumulation, per-Gaussian backward, fused-loss, and
  slabbed voxelization strategies. It also offers a separately named
  initialization variant using FaCT-GS's 3D-Sobel gradient-magnitude-weighted
  sampling without replacement. That variant changes only initial positions;
  it retains Choi et al.'s dual-density polychromatic forward model and is not
  described as literal FaCT-GS. Packed float memory layouts remain a future
  optimization; the current validation path uses double precision. FaCT-GS's
  monochromatic forward model does not replace the Choi et al. equations.
- Gaussian-splat acquisition-stress validation (separate from the Choi
  algorithm):
  Wang et al., "An experimental study on the noise properties of X-ray CT
  sinogram data in Radon space", Physics in Medicine and Biology, 2008, DOI
  [10.1088/0031-9155/53/12/018](https://doi.org/10.1088/0031-9155/53/12/018),
  supports sampling transmitted quantum counts with mean proportional to
  `I0*exp(-p)` before calibration and logarithmic conversion. The current test
  does not claim to model the paper's measured electronic-noise contribution.
  Maier et al., "3D Non-Rigid Alignment of Low-Dose Scans Allows to Correct for
  Saturation in Lower Extremity Cone-Beam CT", IEEE Access, 2021, DOI
  [10.1109/ACCESS.2021.3079368](https://doi.org/10.1109/ACCESS.2021.3079368),
  motivates angle-localized clipping as an information-loss stress condition;
  back-and-forth does not implement or claim their two-scan correction in this
  validator. Ritschl et al., "Robust primary modulation-based scatter
  estimation for cone-beam CT", Medical Physics, 2015, DOI
  [10.1118/1.4903261](https://doi.org/10.1118/1.4903261), supports treating CBCT
  scatter as a smooth additive detector-intensity field. The validator's broad
  Gaussian field is explicitly a controlled surrogate, not the published
  scatter-estimation algorithm and not a Nikon scanner calibration.
- Independent polychromatic validation spectrum and attenuation data:
  Poludniowski et al., "Technical Note: SpekPy v2.0-a software toolkit for
  modeling x-ray tube spectra", Medical Physics, 2021, DOI
  [10.1002/mp.14945](https://doi.org/10.1002/mp.14945). The committed
  validation fixture is generated with pinned SpekPy 2.5.4 parameters; SpekPy
  is not a runtime dependency and the fixture is not claimed to reproduce a
  measured Nikon spectrum. Energy-dependent compound attenuation is generated
  with xraylib 4.2.1: Schoonjans et al., "The xraylib library for X-ray-matter
  interactions: Recent developments", Spectrochimica Acta Part B, 2011, DOI
  [10.1016/j.sab.2011.09.011](https://doi.org/10.1016/j.sab.2011.09.011).
  The fixture generator, package versions, material formulas/densities,
  detector assumption, and fixture SHA-256 are recorded so the test is
  reproducible without treating simulated coefficients as scanner
  calibration.

## Large-Volume CUDA Scheduling

- Out-of-core iterative CT operator splitting:
  Biguri et al., "Arbitrarily large tomography with iterative algorithms on
  multiple GPUs using the TIGRE toolbox", Journal of Parallel and Distributed
  Computing, 2020, 146:52-63, DOI
  [10.1016/j.jpdc.2020.07.004](https://doi.org/10.1016/j.jpdc.2020.07.004),
  arXiv [1905.03748](https://arxiv.org/abs/1905.03748). back-and-forth uses
  the same core rule for streaming iterative paths: spatial slabs are
  storage/scheduling units, not independent CT problems; forward projection
  accumulates global projections and backprojection applies the same global
  residual to each slab.
- Out-of-core CUDA FDK:
  Lu, Ino, and Hagihara, "Cache-aware GPU optimization for out-of-core cone beam
  CT reconstruction of high-resolution volumes", IEICE Transactions on
  Information and Systems, 2016, DOI
  [10.1587/transinf.2016EDP7174](https://doi.org/10.1587/transinf.2016EDP7174).

## PLY Isosurface Extraction

- Automatic threshold first guess:
  Otsu, "A Threshold Selection Method from Gray-Level Histograms", IEEE
  Transactions on Systems, Man, and Cybernetics, 1979, DOI
  [10.1109/TSMC.1979.4310076](https://doi.org/10.1109/TSMC.1979.4310076).
- Topologically corrected extraction:
  Custodio et al., "Practical considerations on Marching Cubes 33 topological
  correctness", Computers & Graphics, 2013, DOI
  [10.1016/j.cag.2013.04.004](https://doi.org/10.1016/j.cag.2013.04.004), and
  Vega, Abache, and Coll, "A Fast and Memory Saving Marching Cubes 33
  implementation with the correct interior test", Journal of Computer Graphics
  Techniques, 2019, [paper](https://jcgt.org/published/0008/03/01/).
  back-and-forth pins the official MC33++ 5.4 lookup table and case logic to
  commit `252dabd7ea6dea3d0529e66b047adbe878caf2fa`.
- Bounded GPU execution background:
  Liu et al., "Parallel Marching Blocks: A Practical Isosurfacing Algorithm for
  Large Data on Many-Core Architectures", Computer Graphics Forum, 2016, DOI
  [10.1111/cgf.12897](https://doi.org/10.1111/cgf.12897). back-and-forth uses an
  independent streamed CUDA scan/sort/weld scheduler around the exact MC33++
  decision path; it does not claim to reproduce PMB in full.

## Experimental Method Boundaries

- Guided-image-filter sparse-angle reconstruction:
  He, Sun, and Tang, "Guided image filtering", IEEE Transactions on Pattern
  Analysis and Machine Intelligence, 2013, DOI
  [10.1109/TPAMI.2012.213](https://doi.org/10.1109/TPAMI.2012.213).
  Xu et al., "Sparse angle CBCT reconstruction based on guided image filtering",
  Frontiers in Oncology, 2022, DOI
  [10.3389/fonc.2022.832037](https://doi.org/10.3389/fonc.2022.832037).
  back-and-forth's TpV-GIF option is explicitly labeled experimental/inspired because
  the published Fourier-domain update is not a literal fit to the Nikon cone-beam
  operator used here.
