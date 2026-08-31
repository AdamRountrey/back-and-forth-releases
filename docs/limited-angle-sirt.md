# Limited-Angle Iterative Reconstruction

## Normalized SIRT

back-and-forth's limited-angle reference solver implements additive SIRT in the normalized algebraic form

```text
x^(k+1) = P[x^k + lambda * C * A^T * R * (b - A x^k)]
```

where:

- `A` is back-and-forth's cone-beam forward projector for the selected reconstruction volume and per-view Nikon geometry.
- `A^T` is the matched residual backprojector using the same detector interpolation weights.
- `b` is the beam-hardening-corrected attenuation projection stack.
- `R = diag(1 / (A 1)_i)` is detector/ray normalization.
- `C = diag(1 / (A^T 1)_j)` is voxel/column sensitivity normalization.
- `lambda` is the relaxation parameter.
- `P` is the optional non-negativity projection.

Back-and-forth computes `R` and `C` implicitly by forward-projecting a unit
volume and backprojecting unit projection data. It does not materialize the
system matrix.

Algebraic solvers compare the modeled volume directly with measured projections.
If the crop omits attenuating material crossed by measured rays, the model is
inconsistent. Cropped iterative and NMAR runs therefore show a default-No
warning. A deliberate override is recorded in QA, reconstruction JSON, and the
custom `[BackAndForth]` XTekCT section; it does not convert the solver into an
interior-tomography method.

## References

- Gilbert, P. "Iterative methods for the three-dimensional reconstruction of an object from projections." Journal of Theoretical Biology, 1972. DOI: `10.1016/0022-5193(72)90180-4`.
- Kak, A. C. and Slaney, M. "Principles of Computerized Tomographic Imaging." IEEE Press, 1988; SIAM reprint, 2001.
- Andersen, A. H. and Kak, A. C. "Simultaneous Algebraic Reconstruction Technique (SART): A Superior Implementation of the ART Algorithm." Ultrasonic Imaging, 1984. DOI: `10.1177/016173468400600107`.
- Wang, G. and Jiang, M. "Ordered-subset simultaneous algebraic reconstruction techniques (OS-SART)." Journal of X-Ray Science and Technology, 2004. DOI: `10.3233/XST-2004-00110`.
- ASTRA Toolbox SIRT documentation, which exposes the same projector/data/reconstruction structure and min/max constraints: https://astra-toolbox.com/docs/algs/SIRT.html
- ASTRA Toolbox SART_CUDA documentation, which documents projection-order control for GPU SART: https://astra-toolbox.com/docs/algs/SART_CUDA.html

## Large Volumes

The streaming CUDA SIRT writer follows the out-of-core pattern used by large-volume iterative toolkits such as TIGRE: the forward projection is still the global operator `A x`, assembled by summing partial forward projections from all volume slabs into full detector residuals, and the backprojection `A^T r` computes each volume slab from all relevant projection residuals. Spatial slabs are therefore storage and scheduling units only; they are not reconstructed as independent CT problems.

The streaming path stores solver volumes as float32 slabs, caches detector
normalization, streams projection batches, and applies one synchronized global
SIRT update per iteration:

```text
x^(k+1) = P[x^k + lambda * C * A^T * R * (b - A x^k)]
```

OS-SART first assembles each subset residual from every volume slab, then
backprojects that fixed global residual into each slab. Streaming CGLS/CGNR
retains global detector residuals and dot products. These are the same global
algorithms scheduled in bounded memory, not independent slab reconstructions.

If full solver state fits safely in GPU memory, the resident CUDA solver is
used. Larger SIRT, OS-SART, and CGLS jobs use host-RAM-backed streaming when
possible and local-disk streaming otherwise.

## OS-SART

The OS-SART option follows Wang and Jiang's ordered-subset SART formulation: projections are partitioned into ordered view subsets, each subset residual is row-normalized, backprojected with the subset voxel-sensitivity normalization, and then applied as one image update. Subset size 1 reduces to projection-wise SART, while larger subsets move toward SIRT-like simultaneous updates with faster practical convergence. The GUI exposes subset size as projection views per subset; a practical starting range is 8-32 views/subset, where smaller subsets update more often and may converge quickly but can amplify noise/limit cycles, while larger subsets are steadier and closer to SIRT behavior. The CUDA resident and streaming implementations use the same Joseph cone-beam projector as the other algebraic solvers; the streaming path assembles each subset residual from all volume slabs before applying any subset update, preserving the global CT operator rather than reconstructing independent chunks.

## Availability and Limits

- Normalized SIRT, SART, OS-SART, CGLS, and TV-CP are available on CPU and
  resident CUDA.
- Normalized SIRT, OS-SART, and CGLS support host-RAM/disk-streamed CUDA output.
- Projection-wise SART and TV-CP require resident solver state for CUDA use.
- Limited-angle reconstruction remains non-unique. Iterations and regularization
  can suppress artifacts but cannot measure missing angular information.
- Stopping settings must be selected and reported for the specimen and study.

