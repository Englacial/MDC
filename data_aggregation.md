# Data Aggregation and Model-Data Comparison

## Background

Datasets rarely come on the same grid. In fact, they may not come on a grid at all.

Observational data is typically non-contiguous and collected along flight paths, at fixed points, or in other structures. While portions of acquisition may be represented as continuous fields (e.g., a Landsat scene encodes a regular grid of radiances), there are invariably gaps between scenes. For some sensors like ICESat-2, this sparsity is obvious—despite incredibly dense along-track resolution, there are clear gaps between altimeter tracks.

In contrast, model data tends to be both continuous and contiguous. Models represent continuous fields that are discretized at whatever resolution is convenient for memory layout or output files. Many ice sheet models use unstructured triangular meshes to support mesh refinement in areas sensitive to fine-scale physics. The key point is that within models, there is a contiguous field represented in whatever discretization is chosen.

## Comparison Strategies

Comparisons generally require that data be referenced to the same space and time. If we assume model output is dense (on a mesh containing our entire domain with values in every cell) and data is potentially sparse, there are three basic strategies:

### 1. ✅ Model → Data Points (Recommended)

**Interpolate the model to the observation locations.**

This is generally the safest and simplest strategy. By matching model data to observation locations, we keep observational data intact instead of executing a transform. Since models represent continuous fields on regular grids, interpolation is well-defined—values are bounded by neighbors, ensuring we're interpolating rather than extrapolating. Model outputs should be slowly-varying within mesh cells, and cell values on all sides of an arbitrary point should exist (unless at boundaries).

**Limitations:** This approach is highly sensitive to observational point distribution. If data has high variability and is sampled at small intervals relative to model cell size, comparisons may be noisy. Model cell values typically represent averages within that cell, which may not align well with point measurements.

### 2. ☢️ Data Points → Model Mesh (Use with Caution)

**Interpolate, extrapolate, or aggregate data points to the model mesh.**

Often referred to as "gridding," this is the default approach for bringing data into models, but also the most dangerous if not done carefully. Unless collected data is extremely dense, gridding likely involves extrapolation or interpolation over distances greatly exceeding the correlation length of underlying data.

**Challenges:**
- When observational data is denser than model resolution, multiple observations within a model cell need aggregation (mean, median, closest to center, or geostatistical fitting)
- When data is sparse, interpolation quickly becomes extrapolation
- It's easy to violate conservation laws with physics-unaware interpolation strategies
- Difficult to propagate high-quality uncertainty estimates far from actual data points
- Geostatistical methods on sparse observations may revert to priors or means

While sometimes the only practical option, this approach almost always requires domain knowledge and careful consideration to implement correctly.

### 3. ✅ Aggregate Both → Comparison Space

**Interpolate or aggregate both data and model outputs to a neutral comparison space.**

This approach merits separate consideration as a useful compromise. A spatial scale for comparison is selected (probably a sparse grid or mesh), and both model output and observational data are aggregated onto this mesh. "Aggregate" is used intentionally—there's generally no need to interpolate or extrapolate. Simply take all values from each native data product that fit into comparison mesh cells and aggregate them appropriately (e.g., mean and standard deviation).

Like data gridding, domain knowledge is important for determining spatial scale. However, unlike gridding, tricky interpolation and extrapolation challenges can usually be avoided.

The comparison space need not have uniform cells. While [Healpix](./healpix.md) meshes are our default choice, comparison meshes can also be defined by external boundaries such as catchment basins or other relevant geographic or temporal units.

