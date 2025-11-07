# Vision for Model Data Comparison

This document outlines a future that doesn't exist yet-- but could!

## The problem

Often times we'd like to compare things; observational data to models, models
to other models, data to other data. Sometimes, this is exploratory-- do areas
of high ice flow occur in glacial ice that is temperate?

However, more often, we want to compare things that are the same. We might have
three different observational estimates of ice velocity. Or a dozen models which
predict surface mass balance on a glacier. Or observations of ice height from an
altimeter that we'd like to compare with a projection that previously modeled
ice height.

Right now, if we want to difference ice velocity from a radar product with a
model run, we might do the following:

```
import xarray as xr

model = xr.open_dataset("./JPL1/ISSM/exp08/xvelmean_AIS_JPL1_ISSM_exp08.nc")
radar = xr.open_dataset("https://its-live-data.s3.amazonaws.com/velocity_mosaic/v2/static/ITS_LIVE_velocity_120m_RGI19A_0000_v02.nc")

difference = radar['velx'] - model['velx']
```

In reality, the above code is considerably more complex. While these are the
same variable, they are almost always represented with different resolutions, 
on grids that don't align or perfectly overlap. The above code would in reality
be much longer, with calls like .interpolate(), .concat(), .resample(), etc.

This is somewhat tolerable for comparing two datasets, but it becomes unwieldy
if we're comparing multiple models. Let's say that we want to compute the
standard deviation of a dozen model outputs, or take the mean of those dozen
models and compare it against an observational dataset. If those data aren't
aligned, we quickly get to repetitive code to harmonize things:


```
model1 = xr.load("...PISM_13km")
model2 = xr.load("...ISSM_4km")
...
radar = xr.load("...radar_5km")

model1_4km = model1.resample(...)
...
```

The above example is truncated, but you can probably imagine that it gets quite
long.

## Working with DataTrees

One of the key things about the example above is that the variable we'd like
to compare is the same across the datasets. At least for things like CMIP6
models, this is true both in the literal sense (we're comparing the x component
of velocity), and in the data space-- all the models have the same variable
name, and using CF conventions, have the same datatype and units.

Similarly, even though the grid resolution and projection/orientation might be
different, we do know where all the data is-- that is, we know both the cell
centers, and the cell bounds for each of the data variables.

What we'd really like to be able to do, is group all the data together in the
same object:


```
ismip_vel = xr.DataTree.open_mfdatatree("./*/*/exp08/xvelmean*.nc")
# or...
ismip_vel = xr.DataTree.open_mfdatatree(["./JPL1/ISSM/exp08/xvelmean_AIS_JPL1_ISSM_exp08.nc",
                                         ...
                                         "./DOE/MALI/exp08/xvelmean_AIS_DOE_MALI_exp08.nc"],
                                         labels=["ISSM", ..., "MALI"]
```

By having all the data in the same object, we can (hopefully), do things like
have provide unified access to plot:

```
# Shows as a panel plot
ismip_vel.plot.imshow()
# or with subset
ismip_veli["ISSM", "MALI"].plot.imshow()
```

Of course, we'd almost certainly like to do the same thing using label based
indexing, such as spatial indices (lat/lon box, dggs grid zone, etc).

And, we'd like a sensible `__repr__` method on the DataTree to let us see what
we're doing--

  - Spatial / temporal extent of tree/subtree/node
  - Resolution, count, projection, grid structure of tree/subtree/node
  - Data chunking, size, location

With similar browsing ability and visualization as current `__repr__`s.

### Using DataTrees to harmonize outputs

The scientific python stack has taught its users to avoid loops, and we'd like
to continue that pattern when working with stacks of data to compare. So, a
resample operation should be able to be defined to a target grid:

```
vel_resample = ismip_vel.interp_like(radar)
# or, more explicitly
vel_resample = ismip_vel.interp(gridspecs, method=function)
```

The user shouldn't care what the underlying grids are, but should be able to
apply the transform in bulk to the tree, optionally by a built-in or custom
function.

Of course, sometimes the inputs will be different. They might be multi-modal:

```
# multiple experiments from multiple models:
ismip_vel = xr.DataTree.open_mfdatatree("./ISSM/*/exp07/xvelmean*.nc",
                                        "./ISSM/*/exp08/xvelmean*.nc",
                                        ...
                                        "./MALI/*/exp07/xvelmean*.nc",
                                        "./MALI/*/exp08/xvelmean*.nc")
```

The above might have some flavor of nested indexing:

  - [model,experiment]
  - [model][experiment]
  - model.experiment

As these trees increase in complexity, we may even start to load using a yaml
template rather than a dictionary-- or a STAC catalog:

```
ismip6 = xr.DataTree.from_yaml("ismip.yml")
multi_radar = xr.DataTree.from_STAC("radar.parquet") # Geoparquet STAC catalog
```

We anticipate a distinct set of indexing for these types of geospatial tree
structures:

  - Parent and leaf node indexing to map to a given subtree or set of leaf nodes
    (inclusive of mapping to a single leaf node)
  - Attribute (i.e., joining on CF conventions for things like temperature)
  - Spatial and temporal indexing (coordinate)
    - Bounds (time and space)
    - Adjacency (DGGS and other indexing trees)
    - Polygon, overlap, etc

There's a lot to do here. We don't need to solve every problem, but we'd like to
prototype some solutions for our problems in a general way that moves both us
and the community forward.

### Uncertainty as a first class citizen

One of the things that's been persistently difficult in our field is
representing and propagating uncertainty. We'd like to leverage some of the
semantic tidiness of using CF conventions: knowing the units and type of
the variable; meaningfully labeled metrics of observation counts, variance, and
error ; a defined schema for adding new labels of uncertainty.

There's a ton a work across the community to be done here-- models may, or may
not, have uncertainty defined as a variable. On the observational side, some data
products have a std or similar per grid cell measure, but some only define a
single std or uncertainty metric at the level of the full product or sensor.
Still, we'd like to lead and lay the groundwork for this to improve in the
future. Here are some things we can do now:

  - Identify if a variable has uncertainty included or not, and push that as a
    node level attribute in the datatree so operators are aware of it.
  - Encode summary statistics in term of observation count and variance; this
    will also be relevant in the next section.
  - Facilitate more use of uncertainty by defining plots and function calls that
    use the uncertainty when it is included.

For our own MDC tool/library, we're consider whether to *require* uncertainty;
at a minimum, we expect to warn loudly when that data isn't included.

### Regridding capability

A common use case for data model comparison is regridding. At times, this can be
fairly naive, and use the existing xarray built in named interpolators.

We expect to have the datatree to expose more elegant and custom options. In
line with uncertainty as a first class citizen, we'd like to be able to have
tighter integration with geostatistics; this could involve bringing in some of
GSatSim capabilities into an xarray accessor package, or it could involve
extending xarray to accept user defined functions for interpolation and
regridding-- likely, it will be a bit of both.

Here are some use cases that we envision:

  - Calling interp() or similar functions such that the output is added to the
    existing datatree (i.e., `datatree.interp(..., addnode=true)`
  - Coupling the datatree object to be able to write out datatrees to cloud
    object storage in a computationally convenient format that is spatially
    chunked (more on that in the next section)
  - Defining datatree structure to ease computation of geostatistics-- both by
    chunking, and by mapping uncertainty and variance information

### Multiscale resolution

There are few use cases for multiscale rasters that are compelling. Comparing
models and data at different resolutions remains a challenge; often the solution
to this is to choose one of the resolutions, and upscale/downscale to that
resolution for the comparison. However, doing so often involves loss of
information, and can cloud the comparison. One option is to define aggregation
functions that summarize the finer resolution data, along with (from above),
useful metrics such as the observation count, variance, or shape of the
distribution for what's been aggregated. 

There are three other use cases that stand out for why multiresolution, via
xdggs and datatree, are appealing:

  1. Computational tractability. Geostatistics typically involves inverting an N
     by N observation matrix. Multiresolution address this is two ways:
      - We can zoom to a patch or area of interest and execute a smaller region
      - For full area coverage, we can define M overlapping tiles with
        observations << N , and compute M replicates of (n_i << N)**2
  2. Web viewing and exploratory data discovery. Implementing multiresolution
     lets us plot and view data across spatial scales. This is relevant for our
     comparison framework.
  3. Storing and training. Modern ML/AI training leans heavily on supervised
     learning, and multiscale rasters open up options to train across
     modalities, and to represent fused observational data.

### Model Data Comparison 

We're a long way from being able to compare models and data in a statistically
robust way that's easy and reproducible. Ultimately, we'd like to be able to
load multiple models to a datatree, and one or more observations to another
datatree, such that both trees have uncertainty encoded in them, and then
define subsets we're interested in:

```
from bayes import prob_data_given_model as PDM

model = open_datatree("s3://...model.zarr")
data = open_datatree("s3://...data.zarr")

subset = dict('models':{...}, 'params':{...}, 'region':{polygon/dggs cell})

evidence_for_model = PDM(data, model, subset, kwargs=...)
```

And then either use the encoded uncertainty from both data and model to assess
the evidence for the model, or, provide a likelihood function to do so.
