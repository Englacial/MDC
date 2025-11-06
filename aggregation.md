# Aggregation, regridding, and more

Datasets rarely come on the same grid. In fact, they may not come on a grid at all. Most observational data is not collected on grids but instead along flight paths, at fixed points, or in other structures. Model outputs are more commonly represented on some sort of mesh, but this mesh may not be a regular grid. Many ice sheet models use unstructured triangular meshes to support mesh refinement in areas that are more sensitive to fine-scale physics.

Comparisons generally require that data be referenced to the same space and time. If we assume that the model output is dense (on a mesh containing our entire domain with a value in every cell) and the data is potentially sparse, then there are three basic strategies that data-model comparisons may employ:

1. Interpolate the model to the data points
2. Interpolate, extrapolate, or aggregate the data points to the model mesh
3. Interpolate or aggregate both data and model outputs to a comparison space

### ✅ Model -> data points

*This is generally a safe strategy.* Model outputs should hopefully be slowly-varying within a mesh cell and cell values on all sides of an arbitrary point should exist (unless we happen to be right on the boundary). So an interpolation from the dense model mesh to specific data sampling points is generally going to be well-behaved.

The downside of this approach is that it is highly sensitive to the observational points. Values within a model cell usually represent an average of the value within that cell. If the data has high variability and is sampled at a small interval relative to the model cell size, then the comparison may end up being very noisy.

### ☢️ Data points -> model mesh

Often referred to as "gridding," this is the default approach to bringing data into models. It is also the most dangerous if not done carefully.

Unless the collected data is extremely dense, gridding data points is likely to involve some amount of extrapolation, or at least interpolation over distances greately exceeding the correlation length of the underlying data. It's easy to violate conservation laws with interpolation strategies that aren't physics-aware. And it's difficult to (though not impossible) to propagate high-quality uncertainty estimates far from actual data points.

There are cases when this is the right option and/or the only practical one. But it almost always requires domain knowledge and careful consideration to get this right.

### ✅ Aggregate model and data -> comparison space

While arguably a generalization of the former two, comparison on a neutral common space (probably a sparse grid or mesh) merits separate consideration. In this case, a useful spatial scale for comparison is selected and both model output and observational data points are aggregated onto this mesh. We say "aggregate" because there's no generally no need to interpolate or extrapolate here. We simply take all of the values from each native data product mesh/points that fit into the comparison mesh cell and aggregate them somehow (mean and standard deviation, for example).

Like data gridding approaches, domain knowledge is important here to determine the spatial scale. But, unlikely gridding, tricky interpolation and extrapolation challenges can usually be avoided.

The comparison space does not necessarily need to have uniform cells. While [Healpix](./healpix.md) meshes are our default choice, comparison meshes can also be defined by external boundaries, such as catchement basins or other relevant geographic or temporal units.