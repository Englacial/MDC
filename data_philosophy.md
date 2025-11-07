

This document details the MDC philosophy on data aggregation and interpolation.

## Background

Observational data is typically non-contiguous. There may be portions of
acquisition which are represented as a continuous field-- for example, a
Landsat scene encodes a regular grid of radiances with no gaps. However, there
are invariably gaps between the scenes, and in reality, instruments like MODIS
are acquiring pixels within a single scene over a time period (i.e., 5 minutes).
For some sensors, like ICESat-2, this sparsity is very obvious; despite
incredibly dense along track resolution, there's obvious gaps between the
altimeter tracks.

In contrast, model data tends to be both continuous and contiguous. While models
like NCEP may have a discrete temporal and spatial spacing (i.e., 6 hours, at 1
degree), the output grid is arbitrary-- NOAA could easy change the temporal
spacing or spatial output, which modelers typically will do to investigating
interesting areas or time slices. The point is that within the model there is a
contiguous field that is represented in whatever discretization is convenient
for memory layout or output files.

## Model Data Comparison Options 

When we go to compare models with data, we have basically three options:

  1. We can move the model to the observation locations.
  2. We can move the observations to the model grid.
  3. We can define an intermediate comparison space at neither location.

Option number 1 is far and away the simplest (and most preferred) option.
Observations are taken at specific locations, and by matching the model data to
those locations, we keep our observational data intact instead of executing a
transform. Also, since models represent a continuous field already, we generally
are well justified in interpolating those values to somewhere else-- this is
because we typically have a regular grid, which means that our interpolations
are bounded by neighbors. Since the model grid is an arbitrary mapping of a
continuous field, regridding or interpolating is well defined; since the problem
is bounded, we can be easy confident that we're interpolating and not
extrapolating.

Option number 2 almost always has one of two problems: either too much data that
needs to be aggregated, or not enough data that needs to interpolated. If you're
sensor is ICESat-2, you have both problems at the same time. Because
observational data is typically denser than model data, it is very common that
there are a multitude of observations within a model cell that need to be
aggregated in some way to be meaningful. There are multiple options for this:
take the closest observation to the grid cell center point, define a mean for
the model cell, or fit a continuous geostatistical model to the data points and
then sample at from that field. For cases where the observational data is
sparse, there are real questions about how to interpolate it: if the data isn't
bounded, the interpolation quickly turns into an extrapolation, and using
geostatistical methods on sparse observation may quickly revert to whatever the
prior or mean is for the fitted model field. While geostatistics is generally
most robust, it can also me computationally expensive if executed naively, and
the appropriate aggregation method typically isn't obvious or something that can
be automated. 
