# Gaussian-splat polychromatic reconstruction

## Status

This method is a research implementation. It is not exposed in the GUI and is
not an accepted production reconstruction path. The executable research CLI is
provided for reproducible numerical evaluation, not for scientific volume
production. Current validation results and failed acceptance gates are reported
in [scientific-validation.md](scientific-validation.md); method and software
references are collected in [citations.md](citations.md).

The target method is Choi et al., *Splat-based Metal Artifact Reduction in
Cone-Beam CT via Polychromatic Modeling*, Computer Graphics Forum 45(2), 2026,
DOI 10.1111/cgf.70339. The authors' implementation is available at
<https://github.com/KAIST-VCLAB/ct-metal-reduction>.

## Published model

The volume is represented by anisotropic Gaussian primitives. Each primitive
has a position, covariance, energy-independent attenuation density `rho_a`, and
photoelectric density `rho_b`. The attenuation field is

```text
mu(x, Ebar) = sum_i c_i(x) * (rho_a_i + rho_b_i / Ebar^3).
```

The cone-beam splat projector analytically integrates each Gaussian and uses the
paper's 15-component discretization of the polychromatic Beer-Lambert model:

```text
P = a - log(sum_k eta_k * exp(-b / Ebar_k^3)).
```

Here `a` and `b` are the projected line integrals of the two attenuation
fields. The normalized energy interval is `[1-gamma, 1+gamma]`, with
`gamma=(Emax-Emin)/(Emax+Emin)`. Following the paper, `Emin=10 keV`, `Emax` is
the tube voltage in kVp, and the response is a normalized linear-decay plus
constant-plateau model controlled by learned ratio and threshold parameters.

Optimization uses one randomly selected projection per iteration, L1 plus
`0.25 * (1-SSIM)`, and adaptive Gaussian clone/split/prune control. The paper
uses 20,000 iterations and initializes 50,000 Gaussians from an FDK volume.

The released training schedule is being treated as part of the reproducibility
contract where the paper delegates details to the implementation:

- Adam with `epsilon=1e-15` and exponential learning-rate interpolation over
  20,000 iterations.
- Base initial/final learning rates are `2e-4/2e-5` for position, `1e-2/1e-3` for
  `rho_a`, `1e-1/1e-2` for `rho_b`, `5e-3/5e-4` for scale,
  `1e-3/1e-4` for rotation, and `1e-3/1e-4` for the response ratio and
  threshold. The released Gaussian model further scales its spatial and density
  rates using the scene-size convention in the authors' code.
- The response ratio and threshold are held fixed for the first 2,000
  iterations, then follow their released log-linear schedules. Gamma is fixed.
- Density parameters use Softplus, response ratio uses sigmoid, response
  threshold uses tanh, and rotation uses a normalized quaternion.
- Clone/split/prune checks run every 100 iterations from iteration 500 through
  15,000, with a released maximum of 500,000 Gaussians and a density pruning
  threshold of `1e-5`.

These values are recorded here before optimizer integration so that subsequent
performance work cannot quietly change the scientific method.

## Input and output rules

- Input projections remain in calibrated negative-log line-integral space.
- Nikon per-projection flux normalization remains part of projection
  calibration. Polynomial beam-hardening correction is not applied before this
  method because the published forward model estimates the polychromatic
  response itself.
- The Nikon source, detector, angle, detector-offset, COR, and crop geometry
  must be mapped without changing physical units or handedness.
- The quantitative output is evaluated from Equation 9 at a declared physical
  energy. It must not silently use an implementation-specific weighting.
- The optimized response parameters, energy bounds, iteration schedule,
  initialization seed, Gaussian counts, losses, and convergence history must be
  stored as reconstruction provenance.

The current Nikon adapter makes these rules executable rather than leaving
them as GUI conventions:

- If Nikon flux normalization is enabled, `I0` is the per-projection mean of
  the recorded air-profile region.
- Every reconstruction projection is required. The initial validated scope is
  a complete circular 360-degree scan with no helical or source/object tilt.
- The selected back-and-forth COR is applied as an affine detector-coordinate
  transform to both the projected Gaussian mean and covariance.
- The method's densities are in inverse millimetres. The output attenuation is
  evaluated at the center of the assumed 10-keV-to-kVp interval and converted
  with Nikon's output-unit scale (normally 1000 for `/m`).
- Detector-space MAR/NMAR and polynomial beam-hardening preprocessing are
  rejected or bypassed rather than being silently composed with this model.

## Validation gates

1. **Spectral reference:** Equation 6 and Equation 12 match independent CPU
   calculations; all analytic derivatives pass central finite differences.
2. **Splat reference:** projected means, covariance, integration compensation,
   culling, and gradients match the paper and R2-Gaussian equations on single-
   Gaussian tests.
3. **Geometry:** Nikon rays and detector coordinates match back-and-forth's
   validated cone-beam geometry for centered, offset, cropped, and limited-angle
   synthetic scans. Unsupported geometries are rejected explicitly.
4. **Optimizer:** a tiny deterministic training case decreases the published
   objective and reproduces a reference run from the authors' implementation.
5. **Synthetic MAR:** polychromatic phantoms include metal beside air, metal
   inside specimen, dense non-metal material, and photon-starved paths. Results
   are compared to known attenuation at a declared energy and to FDK/NMAR.
6. **Memory:** the CUDA implementation respects a measured memory budget and
   checkpoints recoverably. Large-volume voxelization is slabbed; training does
   not depend on CUDA Unified Memory oversubscription.
7. **Production:** only after the preceding gates pass is the method added as an
   explicitly experimental GUI option with complete provenance and warnings for
   the paper's high-Z/K-edge and photon-starvation limitations.

## Repository cross-check policy

The authors' repository is used as an executable reference, but its rasterizer
submodule carries a non-commercial Gaussian-Splatting license. back-and-forth
does not vendor or copy that code. The native implementation is written from
the published equations and checked numerically against reference outputs.

Paper/repository differences are treated as validation questions, not copied
silently. Current examples include different published and released
initialization factors, a released final-volume weighting not stated in
Equation 9, and a suspected component swap in one covariance-gradient term.
These must be resolved with controlled comparisons before production use.

Specifically, the paper states FDK-derived initial densities of `0.075` and
`0.0075`, whereas the released configuration uses factors `0.5` and `0.25`.
The repository also multiplies its configured densification gradient threshold
by 100 before use; the resulting effective threshold is `5e-3`, consistent with
the paper, rather than the literal `5e-5` configuration value. The native
implementation will expose and record the selected initialization policy and
will not disguise one source as the other.

For scale-sensitivity studies, the research CLI accepts an explicitly named
**effective** densification threshold. This is not presented as the Choi et al.
default: the run report labels it
`experimental_densification_threshold_over_released_Choi_training`, records
both the configured value and multiplier, and includes final activated scale
and density distributions. The literal released preset remains the default
and is selected with `published`.

The released clone/split rule is preserved as well. A high-gradient Gaussian
is cloned when its largest activated principal scale is at most `0.1` times
the largest physical volume extent; only larger Gaussians are spatially split.
The native research CLI may receive an explicit effective clone/split boundary
in millimetres for controlled resolution studies. Such runs are labeled as an
experimental spatial-split threshold over released Choi training. The report
records the physical volume extent, stored fractional threshold, and effective
threshold in millimetres. Production presets continue to use the released
`0.1` fraction.

For convergence diagnosis only, the research CLI can also stop adaptive
control before the requested training endpoint. This leaves the released
clone, split, prune, loss, and Adam operations unchanged while reserving a
declared number of updates for the final topology to relax. Reports label this
as an experimental adaptive-control schedule and record the exact start and
end iterations. The production Choi preset remains iterations 500 through
15,000 within its 20,000-step run.

Native research reports also include all-view online epoch means and repeated
loss observations for evenly spaced monitor views. Because Choi's released
view scheduler samples without replacement, each complete online epoch covers
every projection once. These values observe a model that changes during the
epoch; they are convergence diagnostics, not a frozen-model validation loss.

## Implemented validation core

The current native research core includes:

- independent CPU implementations of Equations 5, 6, 9, and 12;
- analytic Equation 12 derivatives checked by central finite differences;
- the published 11-by-11, sigma-1.5 SSIM loss with zero padding;
- exact L1+DSSIM image gradients and complete-view gradients through the
  spectral model, projected Gaussian, cone-beam Jacobian, determinant-ratio
  integration term, world covariance, bounded scales, normalized quaternion,
  and Softplus densities;
- the released Adam update, scene-scaled learning rates, bias correction, and
  2,000-iteration response freeze, including the released behavior in which
  response moments accumulate while their learning rate is zero. Separate
  Gaussian and response Adam clocks reproduce the released densification
  behavior: recreated Gaussian tensors skip that iteration's optimizer step,
  while the response optimizer still advances;
- a tiled CUDA forward pass with density-aware anisotropic detector bounds,
  order-independent accumulation, and the released primary-contribution
  cutoff;
- a per-Gaussian CUDA backward pass over the same support, checked component by
  component against the CPU projected-parameter and response gradients;
- a device-resident training state for activated/trainable Gaussians, Adam
  moments, learned response, and densification statistics. Ordinary training
  views perform analytic cone-beam projection, loss, backpropagation, and Adam
  entirely on CUDA; host synchronization occurs only for adaptive topology
  changes, transactional checkpoints, cancellation, and final output;
- an exact separable CUDA implementation of the released 11-by-11 L1+DSSIM
  loss and its adjoint image gradient, checked pixel by pixel against the CPU
  objective;
- the released adaptive-control transaction: visibility-gated screen-gradient
  accumulation, density-conserving clone and split operations, deterministic
  split sampling, primary-density and bounds pruning, and matching Adam-state
  retention/reset behavior. The adaptive statistic reproduces the released
  rasterizer's `W/2` and `H/2` conversion from detector-pixel mean gradients
  to its pseudo screen-coordinate convention. Each control check records the
  observed-gradient and active-scale distributions, effective threshold,
  count-cap state, and clone/split/prune totals;
- Choi et al.'s thresholded FDK initialization with exactly uniform sampling
  without replacement, implemented as deterministic reservoir sampling so a
  large eligible-voxel list is never materialized; isotropic initial scales use
  the released square root of the mean squared distance to the three nearest
  sampled centers, evaluated with an exact k-d-tree search. Its ordered
  slab-streamed form is bitwise identical to resident initialization for a
  fixed seed;
- a separately identified `fact-gs-gradient-magnitude` initialization variant.
  It computes FaCT-GS's 3D Sobel gradient magnitude over the unthresholded FDK
  field, restricts candidates by the same attenuation threshold, and performs
  exact gradient-weighted sampling without replacement. The implementation is
  slab-streamed with three Sobel feature slices rather than a second full
  gradient volume. It changes position sampling only: Choi's dual-density
  initialization mapping, polychromatic projector, loss, optimizer, and
  adaptive-control schedule remain in force. It is therefore documented as a
  FaCT-position/Choi-density hybrid, not as literal FaCT-GS or literal Choi;
- explicit initialization-density factors and random-seed provenance. The
  released configuration (`0.5` for `rho_a`, `0.25` for `rho_b`, after the
  released `0.15` sampled-density rescale) is the default policy; the different
  factors printed in the paper remain available only as a separately named,
  recorded policy once the training driver is integrated;
- exact mapping of Nikon cone-beam, detector-offset, crop, COR, flux-profile,
  and attenuation-unit conventions;
- a bounded Nikon TIFF/scratch projection source that excludes the trailing QA
  frame, validates detector dimensions, bypasses polynomial beam hardening, and
  agrees exactly with direct fixed-WhiteLevel and per-frame flux-normalized
  preprocessing;
- a callback-driven released training loop that selects one projection per
  iteration from a random-without-replacement view stack, supports calibrated
  projections served from RAM, local scratch, or disk, performs adaptive
  control before the optimizer step, and evaluates without stepping the final
  iteration exactly as in the authors' loop;
- an intentionally slow complete-sum CUDA renderer retained as a correctness
  oracle; and
- a memory-bounded CUDA final-volume voxelizer using 3D tile bins, additive
  accumulation without depth sorting, the authors' three-sigma support and
  `1e-6` contribution cutoff, and z slabs delivered by callback for direct
  output streaming. Its anisotropic CUDA output is checked voxel by voxel
  against an independent CPU Equation 9 implementation.
- a no-output streaming CUDA Ram-Lak initializer that feeds internal inverse-
  millimetre slices directly into the exact global reservoir, bypasses Nikon
  polynomial beam hardening, and releases scoped FDK working storage without
  writing a temporary full volume; and
- a recoverable, transactional training checkpoint containing the complete
  Gaussian and response model, Adam moments and clocks, densification state,
  loss/view history, remaining random-without-replacement view pool, serialized
  random engine, immutable training contract, and required geometry,
  projection, and FDK-initialization fingerprints. CRC64 protects the on-disk
  state, and interrupted/resumed training is tested against a byte-identical
  uninterrupted result. Checkpoint format v2 additionally records the eligible
  FDK voxel count and the explicit threshold, density-rescale, density-factor,
  Gaussian-count, and initialization-seed provenance; and
- a research-hidden production coordinator with ordered SHA-256 identities for
  the exact reconstruction TIFFs, exact geometry and initialization
  fingerprints, all-view Ram-Lak initialization, checkpoint resume, and
  transactional slabbed VGI/VOL output. The output is evaluated at the declared
  physical interval midpoint (`normalized energy = 1`), converted from `1/mm`
  with Nikon's output scale once, and reversed from internal ascending Z into
  Nikon storage order once. A CUDA production-path test checks every written
  voxel against the independent CPU Equation 9 oracle after those conversions.

On the RTX 3090 validation system, a 700-step adaptive run using the real Nikon
`2000 x 2000` detector, 2,000 projections, a `512^3` FDK initializer, and
50,000 initial Gaussians takes 22.231 seconds for training, or 0.03176 seconds
per selected view. It reaches the same 0.0145812 final loss and 49,965 final
Gaussians as the preceding staged implementation, which took 31.915 seconds.
The linear 20,000-step training estimate is 10.6 minutes, excluding the
separately reported projection fingerprint and FDK initialization. This is a
hardware- and workload-specific engineering benchmark, not a direct comparison
with the authors' approximately 18-minute A6000 report because their released
summary does not specify an identical detector, Gaussian count, or dataset.

### Native Nikon resolution stress test

A full-detector test uses all 2,000 measured `2000 x 2000` projections from
`SampleData` and the Nikon post-reconstruction frame (`732 x 853 x 1938` at
`0.03334317 mm`). Initialization is full-band Ram-Lak FDK in that frame and
training follows the released 20,000-iteration Choi schedule. Only selected
native-resolution XY planes are voxelized, avoiding an unnecessary 4.7 GiB
output volume while leaving the trained three-dimensional representation and
detector data unchanged.

The original 50,000-Gaussian run finished with 48,845 primitives in 646.0
seconds of training. A capacity stress experiment started at 500,000 and
finished with 491,861 primitives in 1,267.2 seconds. A subsequent source audit
found that these runs accumulated adaptive-control mean gradients in detector
pixels but omitted the released rasterizer's `W/2` and `H/2` screen-coordinate
conversion. They remain useful historical fixed-capacity images, but they are
not validation of Choi's released adaptive-control behavior and are superseded
for method claims. Raising the initial count is an explicitly recorded
hyperparameter experiment, not the authors' default.

At matched native slice Z 969, those historical 50k and 500k models have correlations of
`0.8963` and `0.9080` against same-frame Ram-Lak FDK within pixels where FDK is
greater than `0.005 /mm`. Their spatial-gradient RMS is only `39.3%` and
`46.0%` of the FDK reference. FDK is not ground truth, so these measurements
do not rank quantitative accuracy; together with the fixed-window images they
show that even the 500k-cap model remains substantially smoother than the
native micro-CT reconstruction. This full-size result does not pass a
native-detail gate and the Gaussian method remains research-only.

Run records and selected slices are under
`out/gaussian_native_sampledata_crop_2000v_50k_20k/` and
`out/gaussian_native_sampledata_crop_2000v_500k_20k/`. New corrected runs use
distinct output directories and adaptive-decision diagnostics so stale
checkpoints cannot be mistaken for corrected evidence.

### Corrected adaptive-control pilot

A bounded 600-iteration pilot reaches the first adaptive-control check with the
corrected released screen-gradient convention. All runs use the same scan,
crop, all 2,000 projections, random seeds, selected-view sequence, and output
slice Z 484. The final loss is the objective for the identically selected 600th
view; it is an early-training comparison, not a converged accuracy metric.

| Initializer | Initial/final count | FDK/init (s) | Training (s) | Iteration-600 loss |
| --- | ---: | ---: | ---: | ---: |
| Choi uniform | 50,000 / 49,888 | 44.05 | 21.39 | 0.03845 |
| FaCT-position hybrid | 50,000 / 49,950 | 57.42 | 21.66 | 0.03349 |
| FaCT-position hybrid | 150,000 / 149,856 | 58.22 | 24.42 | 0.03397 |
| FaCT-position hybrid | 500,000 / 499,507 | 59.66 | 34.78 | 0.03430 |

At the 50,000 starting count, gradient-weighted position sampling lowers this
matched early-view loss by about 13 percent and visibly organizes more internal
structure. Increasing to 150,000 and 500,000 adds visible detail in the fixed
attenuation window, but does not lower the selected-view loss at 600 updates.
The 500,000 run starts at the released cap, so adaptive growth is correctly
blocked and only pruning occurs. These observations argue for matched longer
training and independent edge/detail metrics before considering counts above
the released cap; they do not support a billion-Gaussian requirement.

The full diagnostic records and slices are in
`out/gaussian_native_sampledata_crop_50k_densify_fix_600/`,
`out/gaussian_native_sampledata_crop_50k_fact_gradient_600/`,
`out/gaussian_native_sampledata_crop_150k_fact_gradient_600/`, and
`out/gaussian_native_sampledata_crop_500k_fact_gradient_600/`.

### Corrected matched native-detail runs

Longer corrected runs use the FaCT-position/Choi-density hybrid initializer,
all 2,000 measured projections, the same Nikon output frame, and native slice
Z 969. The declared comparison image is the same-frame full-band Ram-Lak FDK
slice. It is converted explicitly from Nikon output `/m` to `/mm` by `0.001`;
it is a resolution reference, not ground truth. Physical NRMSE is computed
without fitting a scale. The least-squares candidate multiplier is reported
separately to expose attenuation-scale disagreement rather than hide it.

| Initial / final Gaussians | Steps | Physical NRMSE | Fitted multiplier | Gradient correlation | Gradient RMS / FDK | Local edge-width / FDK |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 50,000 / 49,319 | 5,000 | 0.5239 | 1.9654 | 0.6800 | 0.3452 | 1.1666 |
| 150,000 / 148,241 | 5,000 | 0.5192 | 1.9539 | 0.7509 | 0.3815 | 1.1298 |
| 500,000 / 494,216 | 5,000 | 0.5193 | 1.9618 | 0.7985 | 0.4272 | 1.0813 |
| 500,000 / 491,197 | 20,000 | 0.4976 | 1.8708 | 0.8354 | 0.4847 | 1.0513 |

Increasing capacity improves spatial-gradient agreement and local edge spread.
Completing the released 20,000-step schedule improves those detail measures
again, but the Gaussian slice still has only about half the FDK gradient RMS
and its physical attenuation requires a diagnostic fitted multiplier of 1.87.
The method therefore remains research-only: the native-detail gap is smaller,
but quantitative attenuation is not calibrated.

The local edge-width statistic is the median derivative spread along normals
to strong edges in the declared reference image. It uses specimen anatomy and
is not a calibrated slanted-edge MTF measurement. The reusable comparison
command records this scope, the unit multiplier, physical and fitted-scale
NRMSE, correlations, background leakage, gradient metrics, and edge-profile
counts in each `fdk-slice-comparison-z969.txt` file.

The full 500k/20k run was interrupted only after a verified transactional
checkpoint and resumed with the complete optimizer and randomized-view state.
Checkpoint cadence was increased from 100 to 1,000 iterations for this large
model; this changes persistence frequency, not the published update schedule.
The completed corrected records are under
`out/gaussian_native_sampledata_crop_50k_fact_gradient_5k/`,
`out/gaussian_native_sampledata_crop_150k_fact_gradient_5k/`,
`out/gaussian_native_sampledata_crop_500k_fact_gradient_5k/`, and
`out/gaussian_native_sampledata_crop_500k_fact_gradient_20k/`.

The trainer reduces the published objective on both a deterministic tiny
cone-beam fitting problem and a six-view, two-material polychromatic phantom.
The latter also improves independently sampled Equation 9 attenuation error at
the declared reference energy and verifies that each view is selected exactly
once per randomized cycle. Full-size performance profiling, physical
calibration, larger controlled metal/air comparisons, FDK/NMAR baselines, and
complete application provenance remain required before this method can
reconstruct user data.

An opt-in CUDA validation also exercises a `128 x 128` detector and
`128 x 128 x 128` output volume with 20 circular cone-beam views and 14
Gaussians representing irregular specimen, dense non-metal material, embedded
metal, and metal beside air. In the deterministic 600-iteration check,
projection loss decreases from `0.0431838` to `0.00193931` and reference-energy
volume RMSE decreases from `0.00810251` to `0.00154038`. This is deliberately
classified as an in-model projector/optimizer recovery test: its projections
are generated from Gaussian truth, so it does not replace the pending
independent voxel-phantom MAR comparison.

A second opt-in validation, `gaussian_splat_voxel_validation`, removes the
in-model **spatial representation** advantage. It constructs hard-edged
`128 x 128 x 128` two-basis voxel labels independently of the Gaussian
representation. Two hundred forty full-circle
cone-beam views are generated by the separately validated CUDA Joseph
ray-driven projector on a `128 x 128` detector, and the two projected basis
line integrals are combined with Choi et al.'s polychromatic Equation 12. This
original `--spectrum choi` mode still shares the fitted model's spectral form;
it is therefore a spatial/projector validation, not an independent spectrum
validation.

Every sixth view is reserved before both FDK initialization and optimization.
The remaining 200 views supply flat-panel FDK cosine preweighting, full-band
Ram-Lak filtering, distance-weighted cone-beam backprojection, published exact
uniform Gaussian sampling, and the full released 20,000-step training schedule.
The 40 untouched views measure projection-space generalization without FDK
leakage. Material means, bias, standard deviation, and RMSE use explicit labels
and one-voxel 26-neighbor interior erosion, so boundary partial volume is not
misreported as homogeneous-material bias.

The `--calibration-inserts` case contains large separated homogeneous specimen,
dense-specimen, and metal spheres. Held-out projection loss falls from
`0.380586` to `0.001933`, and non-air reference-energy NRMSE falls from
`0.42830` to `0.16859`. Interior attenuation bias is `-6.40%` for ordinary
specimen, `-9.60%` for dense specimen, and `-6.43%` for metal. The fitted
response ratio is `0.25950` for truth `0.25`, while the fitted threshold is
`0.19338` for truth `0.24492`. These values are from the deterministic
2026-08-10 rerun of the current implementation.

The complex case contains irregular specimen, cavities, dense non-metal
regions, embedded metal, and metal extending into air. Held-out projection loss
falls from `0.620669` to `0.003571`, but interior attenuation biases are
`+13.28%`, `-14.14%`, and `+31.53%` for specimen, dense specimen, and metal.
This is explicit evidence that a low held-out projection objective does not by
itself establish quantitative attenuation accuracy or identify the learned
spectral response. The quantitative-calibration gate remains open; these
results must not be described as quantitative material recovery.

The validator now also applies six explicitly recorded measurement conditions
before reconstruction: ideal, high-count Poisson, low-count photon starvation,
angle-localized detector saturation, a broad low-frequency additive scatter
surrogate, and their combined stress case. These are separate acquisition
stress models; they do not alter and are not attributed to the Choi algorithm.
The Poisson model samples transmitted intensity before the negative logarithm,
saturated and zero-count samples are counted, and a one-quantum numerical floor
is recorded if a zero occurs. The scatter field is a declared Gaussian
convolution surrogate rather than a Nikon-calibrated scatter simulation.

On the separated inserts, Gaussian non-air NRMSE is `0.1739` for the Poisson
case, `0.3591` for low counts, `0.3203` for angle-localized saturation,
`0.2643` for the scatter surrogate, and `0.3868` for the combined case. The
corresponding combined-case material biases are `-9.38%`, `-34.63%`, and
`-29.94%` for ordinary specimen, dense specimen, and metal. These results pass
only an optimization-regression check; the quantitative calibration gate
remains failed.

Matched Ram-Lak FDK and oracle-segmented Meyer NMAR are run on the same 200
training measurements. This calibration phantom places metal in air, where
Meyer normalization lacks bracketing nonzero prior support; unsupported runs
are reported rather than filled with invented data. A complementary phantom
with fully embedded metal has zero unsupported runs and shows NMAR improving
ideal non-air NRMSE from `1.1381` to `1.0500` and combined NRMSE from `1.0543`
to `0.9563`. Independently calibrated polynomial beam-hardening coefficients
are still required. Complete records are under
`out/gaussian_splat_stress_240v_inserts_128_*_20k/`,
`out/gaussian_splat_stress_240v_embedded_128_*_20k/`, and
`out/gaussian_splat_calibration_240v_complex_128_ideal_20k/`.

The validator now also has a spectrally independent
`--spectrum independent-tungsten` mode. A committed fixture is generated by
SpekPy 2.5.4 and xraylib 4.2.1 using a declared 225 kVp tungsten reflection
target, 12-degree anode angle, 0.25 mm copper filter, and 1 keV bins from 10.5
through 224.5 keV. Water at 1.0 g/cm3, hydroxyapatite (`Ca5P3O13H`) at
1.85 g/cm3, and pyrite (`FeS2`) at 5.01 g/cm3 provide independently tabulated
energy-dependent attenuation. The detector is explicitly idealized as
energy-integrating with unit efficiency. This is a Nikon-relevant synthetic
stress spectrum, not a measured or calibrated Nikon spectrum. The fixture
versions, assumptions, material formulas, densities, reference coefficients,
and SHA-256 identity are written to every result.

The Gaussian solver is unchanged in this mode: it must fit those independent
measurements with Choi et al.'s published 15-component response model. On
separated ideal inserts, FDK, oracle-prior NMAR, and Gaussian non-air NRMSE are
`0.67949`, `0.69566`, and `0.60379`. Although the Gaussian value is lowest,
its metal attenuation bias is `-61.28%`; a low held-out projection loss does
not rescue that quantitative failure. On the fully embedded-metal phantom,
the corresponding NRMSE values are `0.49874`, `0.45792`, and `0.63270`.
Gaussian fitting visibly reduces radial FDK streaks but introduces broad
shading, blurred boundaries, and ordinary-specimen bias of `-58.91%`.
Therefore the current method does not pass the independent metal-artifact or
quantitative-output gate and remains unavailable in the GUI. Results are in
`out/gaussian_independent_spectrum_calibration_128/` and
`out/gaussian_independent_spectrum_embedded_metal_128/`.

The executable retains a deliberately non-published `--projection-only`
initialization ablation for regression and research use. It is not the method
described by Choi et al. and is not a production option.

## Compatible acceleration work

FaCT-GS (Pieta et al., 2026) provides a separately published acceleration path
for CT Gaussian splatting. Its attenuation model is monochromatic and therefore
does not replace Choi et al.'s two-density polychromatic forward model. The
following implementation strategies are algebraically compatible and are the
design target for back-and-forth's production CUDA path:

- density-aware rectangular detector bounds instead of square bounds around an
  anisotropic projected Gaussian;
- per-Gaussian backward accumulation within a tile, reducing global atomic
  updates while preserving the same summed gradients;
- tile grouping without depth ordering, because X-ray attenuation contributions
  are additive and order independent;
- packed position/radius and conic/integration-factor loads (not yet enabled in
  the double-precision parity path); and
- fused CUDA evaluation of the published 2D SSIM loss and its gradient.

Each optimization must retain a double-precision parity mode and pass against
the complete-sum CPU/CUDA oracles before a faster float implementation is used.
The compatible slabbed voxelization and the separately named FaCT-GS
gradient-weighted FDK position sampler are now implemented. The latter remains
an experimental initializer rather than the default because Choi et al.
specify uniform sampling from a thresholded FDK volume. The exact uniform Choi
initializer uses reservoir sampling, which changes storage complexity but not
the sampling distribution. FaCT-GS's monochromatic density model, loss weights,
TV regularizer, and training schedule are not imported into the Choi method.
