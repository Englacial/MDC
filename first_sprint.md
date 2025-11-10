# Specific Tasks Proposed for Dev Seed Sprint 1

There's a lot design, research, and visioning throughout this repository. The
classic metaphor is "miss the forest for the trees", but, we also want to make
sure that we avoid the opposite problem of "missing the trees for the forest".

Our general strategy is to prioritize high leverage open community problems,
specifically choosing to focus on well defined harder tasks first-- i.e., the
work that is hard, but also well scoped. That is, a few big trees to start on.

What follows is a list of explicit tasks to target as part of our first sprint,
along with some brief description of why. We probably won't do them all-- but
we'll almost certainly do some of them.

## xarray DataTree Enhancements

We start with a focus on DataTree. While we can and do describe general things
we'd like to see in this ecosystem, it's almost certainly best to start with
existing open issues that leverage the cycles various members of the community
have already spent.

### Support DataTree in apply_ufunc (xarray #9789)

This is a sub-issue of #9106 , and is probably the highest impact of the items
described within that issue-- and it would honestly be great to fully close
#9106 if possible. 

There's a lot that we'd like to do across datasets and datatree subtrees /
nodes. This seems to be a high leverage task that will make that working on
those other tasks easier, by providing a cleaner API.

### Parallelize map_over_subtree (xarray #9502)

Much of the later work we envision will involve significant horizontal
scaling. My assumption is that #9789 will help to enable much of that, but
this item is related as well. The hope is that for grantees that are looking
for support with scaling their work, we can leverage datatree to execute those
workflows over collections of datasets.

### DataTree: missing methods (xarray #10015)

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

  1. Performance of deep DataTrees (xarray #9511). We're looking to open dozens
     of models, and hundreds of model / experiment pairs, and it would be nice
     to do this in a way that doesn't hang. The situation for pulling in
     observational data into datatrees could result in thousands of files.
  2. DataTree.to_zarr() is very slow writing to high latency store (xarray
     #9455). Similar to above on the cloud data access back end.
  3. Implement async support for open_datatree (xarray #10742). Related,
     possibly a way forward for some of the above?

## Regridder work via xddgs and xarray

Regridding and interpolation remain stubborn problems in our community-- and
problems that three of our five grantees will be either addressing directly
through there projects, or requesting help with. We have some thoughts on
how to approach this by using some the astronomic communities healpix
framework, which has been successful in using that technology for data
assimilation and integration. Below are few specific issues that we see related
to this in both the xarray and xddgs libraries.


### Explicit indexers: next steps (xarray #6293)

We have some thoughts on how we could use xddgs to implement a regridder that
functions across DataTree nodes. Some of that work is likely dependant on
tighter / full integration with DataTree in xddgs, and much of that integration
work is tied to the look running index refactoring-- design doc and details
here:

https://github.com/pydata/xarray/blob/main/design_notes/flexible_indexes_notes.md

That work has been occurring in blocks, first with xarray PR #5692, and then
continuing with open xarray issue #6293. We don't expect that we'll need to do
all of this, but some of what we'd like to do likely requires addressing some of
these open issues.


## Dataset example: Loading ISMIP6 model outputs (TODO: Thomas)

What we want to be able to do:

```python
ismip6_outputs = open_datatree('gs://ismip6/')
my_variable = ismip6_outputs[{'model': 'UCIJPL/ISSM', 'experiment': 'exp05', 'variable': 'lithk'}]
```

* ISMIP6 outputs are ~1.1 TB that we moved from Globus to a GCloud storage bucket
* Every variable is a separate NetCDF file, which are vaguely CF-compliant but with lots of errors
* All of them are aligned regular grids, but there are multiple resolutions
* This can be done (https://github.com/englacial/ismip-indexing/blob/main/ismip6_index.py), but it takes too much code. Too much code == bad reproducibility and modularity
* Key point: Legacy datasets are not always correctly formatted. How do we encode the "fixes" so that everyone doesn't have to do it themselves?

```python
ismip6_df = ismip6_index.get_file_index() # This is 200 lines of code
path = ismip6_df[{'model': 'UCIJPL/ISSM', 'experiment': 'exp05', 'variable': 'lithk'}]['path']
my_variable = xr.open_dataset(path)
```

### Variance of some subset of models

* Comparison here is assuming some ability to regrid


### Plots of some subset


## Dataset example: IceSAT-2 ATL06 (TODO: Shane)

### Load the data with uncertainty encoded

### Aggregate to a grid with uncertainty propagated

### Probabalistic comparison to one ISMIP6 output
