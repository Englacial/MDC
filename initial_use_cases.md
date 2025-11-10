# Guiding model-data comparison use cases

This document contains some guiding use cases of the types of data-model comparisons we want to facilitate. All of these comparisons are scientifically important, but the specific set here is desgined to stretch the technical capabilities of the tool.

## Basal temperature

*Compare sparse, probabalistic constraints on plausible temperatueres at the ice-bedrock interface against models that resolve thermal state.*

**Data product:** Probability distribution of temperature relative to pressure melting point, sampled along radar flight lines

**Model inputs:** Basal temperature for grounded ice (ISMIP variable: `litempbotgr`), modified by ice thickness (ISMIP variable: `lithk`) to determine temperature relative to pressure melting point.

**Processing needed:**
* Conversion from absolute temperature to temperature relative to pressure melting point
* [Aggregation](regridder/data_aggregation.md) of data and model inputs to the same mesh
* Calculation of a probability from an observed point and the parameters of a statistical distribution

**Statistical comparison:** Map of pointwise likelihood of the model’s predicted bed temperature given data

**What’s the impact:** Modeled temperature fields differ both from internal treatment of thermodynamics in the model and choice of input geothermal heat flux map. Models [diverage a lot in estimated basal temperature](https://models.englacial.org/app?var=litempbotgr&models=NCAR%2FCISM%2CDOE%2FMALI%2CAWI%2FPISM1%2CLSCE%2FGRISLI2&exps=exp05&cmap=auto&nan=0). If we can show in a data-driven way that some models are getting it right and some are getting it wrong, we will hopefully learn from both what physics and what input datasets work.

## Grounding line location

*Compare estimated locations of the grounding line (the contour of where the ocean is partially lifting up the ice in a marine-terminating glacier) through time with observationally-derived estimates.*

**Data product:** Mask of the probability of seawater in each cell over Greenland’s marine-terminating glaciers, derived from a combination of radar and surface altimetery.

**Model inputs:** Percentage of floating ice in the grid cell (ISMIP variable: `sftflf`)

**Processing needed:**
* [Aggregation](regridder/data_aggregation.md) of data and model inputs to the same mesh (in this case, likely to the model's native mesh because re-gridding of any sort will lose too much information)
* Calculation of a probability from an observed point and the parameters of a statistical distribution

**Statistical comparison:** Pointwise probability of the percentage floating measurement (relative to the grid it refers to) given the radar-derived mask of seawater probability.

**What's the impact:** Grounding zones are dynamic, not fully understood, and poorly resolved in most models. Beacuse grounding lines can vary over short timescales, they are a useful representation of the ability of models to capture dynamic changes. Most of our constraints on grounding line locations are indirect, but we have some (sparse) direct measurements from radar reflecitvity that tell us about the presence of seawater. These measurements can be used to validate modelled advance or retreat of grounding lines.

## Surface altimetry

*Compare the model-resolved ice surface against satellite altimetry, directly using the IceSAT-2 land ice height product.*

**Data product:** IceSAT-2's [ATL06](https://nsidc.org/data/atl06/versions/6) data product of geolocated land ice height measurements.

**Model inputs:** Surface elevation (ISMIP variable: `orog`)

**Processing needed:**
* Masking of ATL06 data product to select only over the relevant ice sheet
* [Aggregation](regridder/data_aggregation.md) of data and model inputs to the same mesh

**Statistical comparison:** Probability of the resolved surface elevation given the uncertainty estimate from the ATL06 data product.

**What's the impact:** Changes in surface elevation capture the most fundamental dynamics of ice sheet evolution. Since 2018, we have access to dense laser altimetry at 90 day repeat intervals. This data can tell us if our models are capturing the currently-observed dynamics.

## Surface velocity distribution comparison

*Compare the distribution of velocities found within catchement-scale regional boundaries between models and observationally-estimated data products.*

**Data product:** MEaSUREs ITS_LIVE optical and SAR-derived surface velocity maps

**Model inputs:** Surface velocity (ISMIP variables: `xvelsurf` and `yvelsurf`)

**Processing needed:**
* Aggregation on a catchement by catchement basis of model and observational data
* Bayesian estimation of the distribution parameters representing each (model and data) velocity distribution

**Statistical comparison:** Confidence intervals of the parameters describing the modelled and observed surface velocity distributions for each catchement

**What's the impact:** The ability to reconstruct the types of dynamics we observe is likely a better signal that reconstructing exactly where we see specific velocities. In many cases, we have only a partial understanding of why ice streams occur in the places that they occur, so a model that correctly reconstructs an ice stream somewhere is valuable, even if it doesn't place the fastest flowing ice in exactly the right spot. This should also be a fairer comparison for models that are initialized as paleo spinups.