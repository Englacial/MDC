# Specific Tasks Proposed for Dev Seed Sprint 1

We want to start by laying some foundations for data-model comparisons. A lot of this
is pulling lessons from the work that has been done for cloud-based data processing of
the CMIP6 ensembles and bringing those tools to the cryosphere modelling and
observational communities.

Our general strategy is to prioritize high leverage open community problems,
specifically choosing to focus on well defined harder tasks first-- i.e., the
work that is hard, but also well scoped. That is, a few big trees to start on.

The first part of this document has two specific examples of functionality we want to
make easy.

*We're very happy to hear input on the targeted design patterns. Our goal is for these
types of tasks to be simple and elegant for users -- not to follow the specified syntax
exactly or rely on the specific packages mentioned here.*

The second part of the document covers some specific gaps (linking GitHub issues where
relevant) that we think connect to the functionality we're trying to build. These are
not intended as a TODO list, but we'd be very happy to see some of these issues closed
if it advances our goals described in the first part.

# Part 1: Target use cases

## Dataset example: Loading ISMIP6 model outputs

For a running notebook of "how this is done now," see here: https://github.com/englacial/ismip-indexing/blob/main/working_with_ismip6_examples.ipynb

What we want to be able to do would look more like this:

```python
ismip6_cat = intake.open_esm_datastore('gs://ismip6/ismip6.json')
ismip6_dt = ismip6_cat.to_datatree(preprocess=xmip.fix_ismip6)
my_variable = ismip6_dt['JPL1/ISSM']['exp05']['lithk']
```

* ISMIP6 outputs are ~1.1 TB that we moved from Globus to a GCloud storage bucket
* Every variable is a separate NetCDF file, which are nominally CF-compliant but with a scattering of errors
* All of them are aligned regular grids in EPSG:3031, but there are multiple resolutions
* This can be done (https://github.com/englacial/ismip-indexing/blob/main/ismip6_index.py), but it takes too much code. Too much code == bad reproducibility and modularity
* Key issue: Legacy datasets are not always correctly formatted. How do we encode the "fixes" so that everyone doesn't have to do it themselves?

### Regrid children in a DataTree to a common comparison space

We think it's important for there to be a "batteries-included" way to regridding to a common grid.
Regridding should be lazy and eventually support a pretty wide range of possible grids including rectilinear in lat/lon or projected coordiantes, healpix, and unstructued meshes.

[xESMF](https://xesmf.readthedocs.io/en/stable/) is probably the closest to being what we're looking for, though its dependence on the Fortran/C++ ESMF makes it an annoying piece of the stack to include.

Eventually, we'd want regridding to look something like this:

```python
comparison_grid = xr.Dataset({
    'x': (['x'], np.arange(-30400e3, 3040e3, 16e3)),
    'y': (['y'], np.arange(-30400e3, 3040e3, 16e3)),
})
model_outputs = regrid(model_outputs, target=comparison_grid, func=np.mean)
```

### Variance of some subset of models

This part actually already works pretty well once we have a unified DataTree:

```python
lithk_var = xr.concat(
    [child['lithk'] for m in model_outputs.children.values()],
    dim='m'
).var(dim='m')
```

## Dataset example: IceSAT-2 ATL06 (TODO: Shane)

### Load the data with uncertainty encoded

### Aggregate to a grid with uncertainty propagated

### Probabalistic comparison to one ISMIP6 output

# Part 2: Related issues/improvements in open source tools

## General xarray DataTree Enhancements

Xarray DataTree's could solve a lot of our issues, but they're currently missing some
core functionality that would help a lot.

### Support DataTree in apply_ufunc ([xarray #9789](https://github.com/pydata/xarray/issues/9789))

This is a sub-issue of [#9106](https://github.com/pydata/xarray/issues/9106),
and is probably the highest impact of the items described within that issue--
and it would honestly be great to fully close
[#9106](https://github.com/pydata/xarray/issues/9106) if possible. 

There's a lot that we'd like to do across datasets and datatree subtrees /
nodes. This seems to be a high leverage task that will make that working on
those other tasks easier, by providing a cleaner API.

### Parallelize map_over_subtree ([xarray #9502](https://github.com/pydata/xarray/issues/9502))

Much of the later work we envision will involve significant horizontal scaling.
My assumption is that [#9789](https://github.com/pydata/xarray/issues/9789)
will help to enable much of that, but this item is related as well. The hope is
that for grantees that are looking for support with scaling their work, we can
leverage datatree to execute those workflows over collections of datasets.

### DataTree: missing methods ([xarray #10015](https://github.com/pydata/xarray/issues/10015))

This is a large issue, and we don't need to hit all of these--although that
would be amazing! Given other items in this document, we should prioritize which
of these are or are likely to be blockers for other efforts, and address those.

#### Other misc performance fixes for DataTrees

In general terms, we'd like DataTrees to have the same properties as xarray
Dataset and Dataarrays -- that is, we'd like to be be able open larger than
memory DataTrees quickly, with lazy loading on demand.

We don't want to do optimization for optimizations sake, but we do need the
DataTree interface to be generally usable, and backed be an architecture that
can scale appropriately. There's a number of smaller issues that might need to
be addressed:

  1. Performance of deep DataTrees ([xarray #9511](https://github.com/pydata/xarray/issues/9511)).
     We're looking to open dozens of models, and hundreds of model / experiment
     pairs, and it would be nice to do this in a way that doesn't hang. The
     situation for pulling in observational data into datatrees could result in
     thousands of files.
  2. DataTree.to_zarr() is very slow writing to high latency store ([xarray
     #9455](https://github.com/pydata/xarray/issues/9455)). Similar to above on the cloud data access back end.
  3. Implement async support for open_datatree ([xarray #10742](https://github.com/pydata/xarray/issues/10742)). Related,
     possibly a way forward for some of the above?

## Regridder work via xdggs and xarray

Regridding and interpolation remain stubborn problems in our community-- and
problems that **three of our five** grantees will be either addressing directly
through there projects, or requesting help with. Two of our grantees are working on
interpolation and ***still*** want help with regridding as a preprocessing step
to their interpolation methods! We have some thoughts on how to approach this
by using some the astronomic communities healpix framework, which has been
successful in using that technology for data assimilation and integration.
Below are few specific issues that we see related to this in both the xarray
and xdggs libraries.

### Explicit indexers: next steps ([xarray #6293](https://github.com/pydata/xarray/issues/6293))

We have some thoughts on how we could use xdggs to implement a regridder that
functions across DataTree nodes. Some of that work is likely dependant on
tighter / full integration with DataTree in xdggs, and much of that integration
work is tied to the look running index refactoring-- design doc and details
here:

https://github.com/pydata/xarray/blob/main/design_notes/flexible_indexes_notes.md

That work has been occurring in blocks, first with [xarray PR
#5692](https://github.com/pydata/xarray/pull/5692), and then continuing with
open [xarray issue #6293](https://github.com/pydata/xarray/issues/6293). We
don't expect that we'll need to do all of this, but some of what we'd like to
do likely requires addressing some of these open issues.

### Design/Datamodel Decision: Very large datasets ([xdggs #143](https://github.com/xarray-contrib/xdggs/issues/143))

As mentioned above, we need to be able to open datatrees in a way that is
usable. While the issue is presented for datasets rather than datatrees, it
surely impacts both structures.

This is also likely related to [xdggs #155](https://github.com/xarray-contrib/xdggs/issues/155).

### Spatial Partioning in Zarr / geopandas / xarray

We're unsure of the best way to open large amounts of observational data in
datatrees. As mentioned above, we want to be able to do lazy loading, and open a
datatree without loading it. Given the data volumes and density of observational
datasets, this likely means defining some spatial partition of the data.

To actually do that, we likely have two (related) paths forward:

  1. Figure out how to correctly represent spatial sharding in zarr
  2. Use VirtualiZarr to map / save a prior constructed datatree

Doing number 2 might require also doing number 1; [xarray issue #9634](https://github.com/pydata/xarray/issues/9634) seems
relevant here, although there are probably other issues and PRs across zarr,
VirtualiZarr, xarray, etc.

