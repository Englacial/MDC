# Model Data Comparison (MDC)

This document outlines the design for a MDC tool, which should allow a user to
compare model-to-model, model-to-data, or data-to-data in simple, repeatable,
and modern way.

## Guiding Use Case

The ISMIP6 inter-comparison project is a collection of > a dozen model runs of
icesheet models, forced under various conditions; here's a table showing more
explicitly what we mean:

https://docs.englacial.org/ismip-indexing/index.html

The full output for Antarctica is ~1.2 terabytes, although adding in forcing
and auxiliary datasets moves the total closer to 2TB. This data is an awkward
amount to work with - individual model runs can easily fit into memory, but
comparing across them is cumbersome, especially within a compute or bandwidth
limited setting (i.e., a browser).

Part of the reason for ISMIP6 is to make comparisons easier between the output
datasets; participants agreed to conform to a common set of grids (which nest),
and a common set of experiments. Prior to ISMIP6, each model would typically
have a custom grid specification (i.e., 13.5km for one model, and 20km for
another).  None of the model grids are expected to facilitate comparisons with
observational data, which is defined either on it's own grid, or not gridded at
all.

We'd like to take advantage of this setup to test and develop our MDC tool.
We'll compare some of the models that are in the common grid, and then compare
how our MDC does when comparing against the native resolution; i.e.:

```
    (ISSM @ 8 km, ice surface) - (PISM @ 8 km, ice surface)
```

...versus:


```
   (ISSM @ 13.5 km, ice surface) - (PISM @ 8 km, ice surface)
```

The first comparison is a simple difference (i.e., subtraction) in xarray,
which works because the grids are aligned. The second case should give similar
(or near identical) output, and be just as simple to execute.

The tool we're developing will be cloud native, and work on large datasets.

## Modules and Technology Stack

We expect that our tool will have the following modules:

  1. Gridder / Regridder
  2. Statistics
  3. Visualization

There may be additional features later, but to start, we're focused on data
assimilation for simple comparisons.

Our tool does the following:

```
Input(any data resolution / projection / filetype / etc) --> Common DGGS (in cloud) 
```

```
Output(Common DGGS) --> (any data resolution / projection / filetype / etc) (with aggregation, stats, viz)
```

In other words, we are agnostic as to data inputs or outputs, but highly
opinionated about how we assimilate within our internal data layout model.

Note that there's already a bit of prior art here that's worth looking into:

- [Remapping HEALPix Pipeline](https://github.com/WACCEM/healpix-remapping) - High-performance remapping at NERSC (for input side)
- [xESMF](https://xesmf.readthedocs.io/) - Regridding for Earth System Models (for output side)

We use two internal filetypes for data at rest: 
[zarr v3](https://zarr-specs.readthedocs.io/en/latest/v3/core/v3.0.html)
(for gridded data) or parquet (for non-gridded data), and build on and use
the following core technologies and specifications:

  1. xarray
  2. [xdggs](https://github.com/xarray-contrib/xdggs)
  3. healpix
  4. Apache Hive
  5. Climate and Forecast metadata convention (cf-xarray or similar)

For visualization, we assume some combination of three paths forward:

  1. Native grid based plots, from users within the python kernel called on
     the xarray object (with possible regridding as needed)
  2. Rust / web assembly to read and interpret the zarr and qeoparquet files
     directly from S3 into a web browser (with level of detail zoom and pre-
     built data pyramids). [cdshealpix](https://github.com/cds-astro/cds-healpix-rust) 
     is an example here, but there are several in this space.
  3. [lonboard](https://github.com/developmentseed/lonboard) integration for
     GPU rendering (since it already has support for some of this). Another 
     option in this space is [gridlook](https://github.com/d70-t/gridlook),
     which has a [zarr-based healpix webmap here](https://gridlook.pages.dev/#https://swift.dkrz.de/v1/dkrz_41caca03ec414c2f95f52b23b775134f/reanalysis/v1/ERA5_P1M_7.zarr::varname=100u::camerastate=eyJwb3NpdGlvbiI6WzAuMDAwMDAyNjA0OTA1MTc4NTY3OTgyNCw3LjgzNjUwODY0NjQ0MTU4N2UtNywyLjcyMDEwNjk2MzQyNzU5MTRdLCJxdWF0ZXJuaW9uIjpbMi45ODMyNDU5MDE2NjA5MjRlLTcsNC4wMTI3ODc4NTk3MjIxNjhlLTcsMC44MDI1MjE5MDAzNjcyMzMxLDAuNTk2NjIyNjYwODQyNDQxMl0sImZvdiI6Ny41LCJhc3BlY3QiOjIuMDQwNDY4NTgzNTk5NTc0LCJuZWFyIjowLjEsImZhciI6MTAwMH0).

Statistical operations fall into three categories:

  1. Simple. These are things like nearest / approximate neighbor, mean or std
     per cell, counts, and simple regridding; i.e., things that are natively
     supported by xarray or xdggs.
  2. Domain. External libraries like [GStatSim](https://github.com/GatorGlaciology/GStatSim)
     for geostatistical interpolation, or GDAL for more complex (but still
     standard) types of resampling and aggregation.
  3. Custom. User defined, or specialized metrics that we may define for
     climate model or observational data specifically.

We don't expect a robust custom statistical package for our MVP, but want to
be aware that the design should facilitate having support for one down the line.

**Other References:**
- [xdggs MOC Index PR #151](https://github.com/xarray-contrib/xdggs/pull/151) - Memory-efficient indexing for billion-cell HEALPix datasets
- [ndpyramid](https://github.com/carbonplan/ndpyramid) - Multi-resolution pyramid generation for xarray from carbon plan (pyramids for zarr data not on healpix)
- [nextgems](https://nextgems-h2020.eu) - European 'digital twin' project for climate that uses Healpix
- [nextGEMS HEALPix Implementation](https://nextgems-h2020.eu/catalog.yaml) - Catalog from above effort for production km-scale climate simulations on HEALPix grids
- [mhealpy Documentation](https://mhealpy.readthedocs.io/) - Multi-resolution HEALPix maps with MOC support
- [MOC v2.0 Standard](https://ivoa.net/documents/MOC/) - IVOA Multi-Order Coverage map specification

## Data Model and Philosophy

We use a multi-resolution DGGS to make data assimilation consistent across data
at all scales. The stack is:

  - Healpix (Nested)
    - MoC (Multi-Order-Coverage)
      - Morton indexing (hive ordering)

Our users should not care (or even be required to know) what our internal grid
format is. They should be able to load ISSM and PISM using standard xarray lazy
loading, and operate on the two data sets as if they were aligned-- because
they will be aligned as part of our ingest process.

In general, we should aim to keep to original data as closely as possible
during the ingest-- i.e., keep both the original *value* of the observation,
and the original *location* of the data as close possible. The point of our
internal framework is to make comparison between datasets easy, but we want
to cast to a common output spatial domain explicitly (and as defined by the
user, with sensible default suggestions), rather than implicitly. 

In terms of gridded data, we adopt a simple definition of how to map to our
grid that has two tenants:

  1. Data location is defined by the center of the referring raster cell
  2. Resolution for gridded data is set at least as fine as to not encounter
     observation collisions

Given the above, this means that our gridded data at the lowest leaf nodes
will either be sparse arrays of observations, or unstructured observation
tables.

For our visualization module, we propose some multi-resolution conventions:

  - shards at 3 orders of magnitude
  - chunks at 3 orders of magnitude

So a given resolution level would span 6 orders of resolution, and past 6
orders, we would aggregate to coarser resolution, and write it within the hive
structure.  Aggregated cells should have at a minimum a 'count' value that
describes how many observations contribute to that cell; ideally, there also
would be std stored as well-- although this could be also potentially be
calculated client side.

**References:**
- [mortie PyPI Package](https://pypi.org/project/mortie/) - Morton indexing functions using HEALPix

## Development Goals

For non-gridded point observations, much of the technology stack is already
mature. We can write out partitioned hive style parquet files by doing the
following:

  1. Read the columnar data into a dataframe (i.e., pandas)
  2. Calculate the indices per observation (from lat/lon, i.e., a coordinate
     transform)
  3. Add those indices to the dataframe
  4. Split the indices column to reflect the chunking and sharding schema
  5. Call export_partitioned('s3://bucket/prefix/{subdir}/sensor.parquet, 
                             by=['shards', 'chunks'])

Step 5 above could have additional temporal and spatial chunking, i.e.,

  ```
  by=['shards','chunks','regions','years']
  ```

Step 4 generally looks like this:

    midx                    region       chunk      shard

    5121313434313234232     5121313      434313     234232
    5121313434313234231     5121313      434313     234231
    ...
    4112412432332143114     4112412      432332     143114
    4112412432343213234     4112412      432343     213234

The first two observations are adjacent, and share the same region and chunk;
the last two are further apart and only share the same region. We can
optionally drop the 'midx' index since it can be recreated by concatenating
region-chunk-shard together.

Ultimately, we'd like to be able to do something analogous to above, by going
directly from an xarray in memory, to a partitioned (i.e., chunked and sharded)
zarr store in S3 style object storage. We start with this as an MVP-- going
from in memory to sharded/chunked cloud stored zarr grids. This makes the
simplifying assumption that our gridded data fits into memory on our worker
nodes, and that the parallelism we would exploit will be at the level of either
multiple datasets, or multiple time-slices. However, we are cognizant that
future development should allow for incremental zarr construction that works
from the data leaves up toward the root (i.e., ingesting many small spatial
extent observation tiles, rather than unified large continuous rasters). 

Ideally, we're looking for a fairly simple API that looks something like:

```python
    ds = xr.open_dataset("data.nc")
    moc = xdggs.healpix(ds, resolution=11, indexing_scheme="morton")
    # resolution for terminal leaf node of MOC tree
    moc.to_zarr(...)
    # spatial chunking / sharding by either implicit inspection
    # or by explicit parameter setting
```

Opening the dataset would either be same as a standard xarray zarr store,
with options to change the data resolution once the store is loaded. e.g.,

```python
   dt = open_datatree('moc_pyramid.zarr)
```

...or equivalent to open a zarr DataSet / DataArray

This is different than the NERSC remapping Healpix pipeline, in that we're
trying to get some of this functionality directly into xarray and zarr at
a deeper and more general level.

## Open Technical Questions

While we have a broad vision and specific technical stack in mind, there are
a number of design choices that still need to be made:

### Do we use xarray DataTree?

DataTree was merged into xarray core starting from v2024.10.0+ , and conceptually
can enable much of the multi-resolution functionality we'd like to have. However,
given that it is fairly new, we're still seeing some performance issues (i.e.,
xarray issues #9455 and #9511). More generally, while DataTree is supported in
xarray, it still isn't implemented in xdggs (see xdggs #171), so using DataTrees
would likely involve developing that functionality within xdggs.

A more fundamental question is does DataTree simplify the interface compared to
using DataArray/DataSet? We could implement zarr arrays that are chunked/sharded
in a spatial way without DataTree, and then could provide functions for creating
(on demand, or preprocessed) different resolutions of DataSets/DataArrays at
different resolutions. In other words, what do DataTrees give us, and what are
the cost for implementing them-- and more specifically, what is the marginal
cost and benefit to this design choice compared to avoiding them?

**References:**
- [xarray DataTree Performance Issue #9455](https://github.com/pydata/xarray/issues/9455) - Cloud storage write performance challenges
- [xarray DataTree Deep Tree Performance #9511](https://github.com/pydata/xarray/issues/9511) - Performance with deep hierarchical structures
- [xdggs DataTree Support Request #171](https://github.com/xarray-contrib/xdggs/issues/171) - Integration request for DataTree in xdggs
- [DataTree Documentation](https://docs.xarray.dev/en/latest/user-guide/hierarchical-data.html) - Official xarray hierarchical data guide

### What is our optimal chunking and sharding strategy?

We've adopted a hierarchical spatial partition schema (see morton.md for more
details), but there remains some questions:

  1. How does the morton v1 specification look-- are there changes we should
     make before freezing it?
  2. What are the appropriate resolution conventions? Right now we are looking
     at 6 order magnitude chunks with 3 order magnitude shards. Does this make
     sense? Are these orders globally fixed, or defined relative to the dataset
     ingest?
  3. Do we adopt a time dominate partition strategy? Within the hierarchy, we
     have a run of spatial index labels, but still need to encode time. Is time
     at the base-- S3://bucket/prefix/2022/{region}/{chunk}/{shard}/file.zarr ,
     or the tail-- S3://bucket/prefix/{region}/{chunk}/{shard}/2022/file.zarr?
  4. Do we need to modify the zarr schema at all to do this?

**References:**
- [Zarr v3 Sharding Specification](https://zarr-specs.readthedocs.io/en/latest/v3/codecs/sharding-indexed/) - Hierarchical chunking for reduced file count
- [Zarr Sharding Issue #2346](https://github.com/zarr-developers/zarr-python/issues/2346) - Sharding implementation in zarr v2/v3
- [Pangeo Cloud Data Guide](https://pangeo-data.github.io/pangeo-cmip6-cloud/) - Best practices for chunking climate data 

### Do we store Morton or Nested?

There's already support for Nested MOC across the ecosystem, but Morton
ordering gives us a few advantages:

  1. Single number that encodes both location (by value) and resolution (by
     string length)
  2. Consistent hive partition strategy
  3. Simple operations for coarsening obs / grids
  4. Efficient operations for co-registering data (e.g., point to grid cell)

However, there's also a few downsides:

  1. Inefficient coordinate packing
    - Base healpix nested index has 29 levels to the tree (zoom levels)
    - Morton indexing has 19 levels that fit in a 64-bit integer
  2. Lack of native support in current libraries (plotting, GDAL, xdggs, etc)

Downside 2 can be addressed by either adding support, or by simply wrapping
the coordinate transform from Nested to Morton. Downside 1 can be addressed by
either using morton indices relative to the region / chunk path, or by only
using them to label files in combination with encoding Nested indices and
casting from them as needed.


### How hard do we need to think about global coverage?

We are fortunate in that our data is almost always polar-- we look primarily
at Antarctica, and secondarily at Greenland. The healpix base cell sub-divide
and tessellate nicely into a regular, well behaved grid. Across the base cells
there are cases, such as at each pole, where four base cells meet and would
appear to even be capable of being aggregated up another level in the spatial
tree. The reason that there isn't another level in the tree is because when
a base grid cell that touches the North or South pole meets with base cells
toward the equator, those base cells have an three-way corner intersection.

The question is, how much do we need to think about this, given that our data
won't encounter this for our science domain? Realistically, the answer is
"very little", but also not "zero". We should not solve problems we don't
have for our MVP, but we also should try to avoid decisions that will cause
other communities pain when they try to solve them later!

**References:**
- [HEALPix Geometric Properties](https://healpix.sourceforge.io/html/intro.htm) - Discussion of vertex valence irregularities
- [CF Conventions HEALPix Discussion #433](https://github.com/cf-convention/cf-conventions/issues/433) - Standardization efforts for HEALPix in CF

### Should we consider Uxarray?

My impression is not. Uxarray seems to add complexity, and our grids all have
structure; i.e., both our input model grids, and our internal intermediate
assimilation grid. For unstructured data, parquet with appropriate tiling
seems to already suffice. That said, I have little experience with Uxarray
and would be open to hearing a compelling reason why it applies as a candidate
for our tech stack.

**References:**
- [UXarray HEALPix Support](https://uxarray.readthedocs.io/en/latest/user-guide/healpix.html) - Converting HEALPix to UGRID conventions
- [Project Pythia HEALPix Cookbook](https://projectpythia.org/healpix-cookbook) - Tutorials for HEALPix with uxarray

## Additional Resources

### Community Resources
- [awesome-HEALPix](https://github.com/pangeo-data/awesome-HEALPix) - Curated list of HEALPix resources
- [awesome-discrete-global-grid-systems](https://github.com/LandscapeGeoinformatics/awesome-discrete-global-grid-systems) - Broader DGGS resource list

### Key Papers
- Górski et al. (2005) ["HEALPix: A Framework for High-Resolution Discretization"](https://iopscience.iop.org/article/10.1086/427976) - Foundational HEALPix paper
- Singer et al. (2022) ["HEALPix Alchemy"](https://iopscience.iop.org/article/10.3847/1538-3881/ac5ab8) - Database spatial indexing
- ["XDGGS: Xarray DGGS Support"](https://isprs-archives.copernicus.org/articles/XLVIII-4-W12-2024/75/2024/) - ISPRS paper on xdggs framework
