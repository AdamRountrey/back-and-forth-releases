# CUDA Benchmark Notes

## Command

back-and-forth includes a repeatable real-data CUDA benchmark:

```text
back-and-forth-cli cuda-benchmark <scan-folder> [sizes-csv] [iterations] [projection-step] [width-mm]
```

It reconstructs centered cubic volumes using the scan's actual detector geometry and projection TIFFs, then reports CUDA FDK and normalized CUDA SIRT timings for the same size/projection set.

## SampleData Baseline

Machine: NVIDIA GeForce RTX 3090, 24 GiB.

Scan: `SampleData`, 800 usable `2000 x 2000` projections, `359.549 deg` measured coverage.

Scout run: 8 projections (`projection_step=100`), 2 SIRT iterations.

```text
algorithm,size,projections,seconds,seconds_per_iteration
fdk,128,8,0.5709,0
sirt,128,8,0.5032,0.2516
fdk,256,8,0.5406,0
sirt,256,8,0.5249,0.2625
```

Denser run: 40 projections (`projection_step=20`), 2 SIRT iterations.

```text
algorithm,size,projections,seconds,seconds_per_iteration
fdk,128,40,2.3235,0
sirt,128,40,2.3864,1.1932
fdk,256,40,2.4568,0
sirt,256,40,2.6709,1.3354
fdk,384,40,2.2941,0
sirt,384,40,2.9072,1.4536
```

## Interpretation

For these controlled cubes, measured CUDA SIRT time scales roughly linearly
with projection count and more moderately with volume size. Each iteration
performs forward projection, residual formation, and matched residual
backprojection, so runtime also scales approximately linearly with the selected
iteration count.

These timings are hardware- and dataset-specific benchmarks, not image-quality
recommendations. Limited-angle settings still require enough projections and
iterations for the scan geometry and specimen.

