# Why this project?

## Data-model comparisons are important

Current ensembles of ice sheet models (that simulate the future of the Antarctic and Greenland Ice Sheets) reflect [deep uncertainty about the future of the Antarctic Ice Sheet](https://www.nature.com/articles/s41586-021-03302-y) in the form of diverging estimates of mass loss from the continent. One of the primary challenges with modelling ice sheets is that our observational record is short compared to the long timescales over which these massive systems respond. But even in our short observational record, current ensembles [do not do a good job of matching observations](https://tc.copernicus.org/articles/15/5705/2021/).

One solution is to weight models by how well they fit to observations, and this is certainly a useful step. But results [depend strongly on which data you weight against](https://tc.copernicus.org/articles/17/4661/2023/) and this approach doesn't help us understand *why* models behave differently.

Understanding *why* is the key to improving models. If we discover that model disagreement stems from different process parameterizations, experiments can be designed to learn more about that process. If we discover that mesh sizes are a big part of the explanation, we can invest more in compute power to run higher-resolution models. If we discover that uncertainty in input data is large, we can organize targeted data collection to reduce uncertainty. And so on...

No one model-data comparison is going to solve our problems. Instead, we need to engineer a world in which *model-data comparisons are easy, reproducible, and informative*.

## The technical problems with model-data comparisons

| | Problem | Pitfalls | Our Approach |
| --- | --- | --- | --- |
| **INPUT** | <ul><li>Manual downloading of data is not scaleable or reproducible</li><li>Too many lines of code are spent loading and getting data in the right shape</li></ul> | <ul><li>Waiting on data providers to adopt new formats is too slow</li><li>Re-writing and storing data in large quantitites is [not](https://pangeo-forge.readthedocs.io/en/latest/) [sustainable](https://discourse.pangeo.io/t/us-central1-pangeo-hub-down/4591)</li></ul> | Lightweight, **zero-infrastructure catalogs** that reference data where it already lives using **CF conventions** for interoperability, such as [xOPR](https://github.com/englacial/xopr). <br /> Core technologies: [STAC](https://stacspec.org/en), [Intake](https://github.com/intake/intake), and [VirtualiZarr](https://github.com/zarr-developers/VirtualiZarr) |
| **COMPARE** | Lack of sensible defaults leads to reinventing the wheel, especially in: <ul><li>Propagation of uncertainty distributions</li><li>Sparse data to gridded data comparisons</li></ul> | Forcing all input datasets to a common grid pushes complexity elsewhere and complicates uncertainty quantification. | <ul><li>Automatically capture and propagate units, dimensions, and error/uncertainty fields from input datasets</li><li>Bring data in on the grids/meshes its on. Make [safe remeshing strategies](aggregation.md) the default.</li></ul> |
| **REPRODUCE** | <ul><li>Usually harder to re-run analyses than people claim, and no automated ways to verify reproducibility</li><li>It's difficult to swap out input data sources to extend or remix comparisons</li></ul> | Forcing users into a walled garden adds too much friction to collaboration | Automated workflows should prove reproducibility and facilitate easy data updates. Provide [template repositories](https://github.com/Englacial/xopr-snakemake-demo/), but don't force specific tools on users. |

