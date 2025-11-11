# Anticipating Future Sprints

While our first sprint focuses on our Model Data Comparison (MDC) tool, we
expect that a number of our grantees will integrate with the project as well--
or with our other project, [xopr](https://github.com/Englacial/xopr). Thus, to
help with the design portion of the MDC tool (and the first sprint), it's
instructive to look at a preview of what we expect to support in the coming
months.

While what we do to support our grantees will depend on the direction that they
choose, we provide some sense here based on their respective backgrounds, as
well as their proposal and our conversations with them.

## Emma 'Mickey' MacKie

[Mickey](https://www.gatorglaciology.com/people) is the core dev behind the
[GSatSim](https://gatorglaciology.github.io/gstatsimbook/intro.html)
Geostatistics package, and her proposal builds on that expertise to target the
creation of a living data product focused on topographic uncertainty, along
with the pipeline to produce it. This project will almost certainly involve
leveraging her GSatSim library, which is popular and mature in it's own right--
we had **multiple** proposals from other groups with workplans that included
using GSatSim within their own pipelines. For Mickey's pipeline, the plan is to
integrate with a larger planned software called 'GeoBRIDGE', by developing a
topographic module for that project.

Our interview with Mickey indicated that she'd like help with a few things. At
the top of the list is better access to bed data from the cloud without having
to download locally-- which would likely involve directing cycles to make our
xopr radar access tool more cloudy. Other things were focused on regridding
(which our MDC tool will have a module for), and general software expertise to
help with the design of the larger GeoBRIDGE software so that the modules are
setup to interact successfully with each other.

We don't know much about GeoBRIDGE outside of Mickey's topo module, but this is
probably a good place to both stress test and direct some the DataTree work. We
expect that GeoBRIDGE will call GSatSim, and if we can figure out how to get
that call to work through xarray, that will also be helpful for our regridder,
which should ideally have both simple and complex interpolation functionality.
For geostatistics in particular, there good reason to figure out how we can
partition that work load, which could have overlap with spatial chunking.

## Cheng Gong

[Cheng](https://github.com/enigne) is a contributor to the [ISSM](https://issm.jpl.nasa.gov/) 
model (repository [here](https://github.com/ISSMteam/ISSM)), and his project is
focused on [developing a Physically Informed Neural Network (PINN)](https://github.com/ISSMteam/PINNICLE) 
to provide mesh free reference data set for ice sheet model initialization. Effectively,
the PINN will have overlapping 50 by 50 km overlapping chunks, the weights of which are
derived from observational training data, and then the PINN segments can be
sampled to an arbitrary spacing while still preserving the physics boundary
conditions.

Our conversations with Gong indicated that he would like help with tiling and
data prep (again overlapping with our regridder), as well as help with
scaling-- which could potentially involve some horizontal GPU scaling
(dependant on if there's further scaling beyond his current NSF access).

It's worth noting that even though both Mickey and Cheng are developing
sophisticated statistical models capable of outputting a statistically robust
product to an arbitrary spacing from arbitrarily spaced inputs, the both still
mentioned a that they could use help with regridding. This is for two reasons:
first, the statistical models are computationally expensive, so they need
something that can tile inputs to reduce the data volume (especially for models
that have polynomial complexity with respect to inputs), and, secondly there's
still significant data preprocessing and preparation that needs to happen,
often from heterogeneous sources. For Cheng, he mentioned wanting a full time
person just of data tiling and ingest-- our hope is that some of this can be
solved with software that significantly shortens the ingest and preprocessing
process!

## Eliza Dawson

[Eliza](https://elizadawson.github.io/) is developing a depth averaged
temperature product from radar sounding data. As with the majority of our
grantees, this access would likely be through xopr, and we would anticipate
supporting her with both enhancements to that library, and with additional help
for scaling and open science.  Unlike most of our other xopr users, Eliza would
need access to raw sounder data...  while xopr (currently) only provides search
and access to the processed radar flight granules. We'll need to talk more with
Eliza to see how to best support her-- it could be by developing a virtulizarr
mapping to pull in raw radar data, or by defining a new STAC schema for the raw
granules, or by developing horizontal scaling on the current CReSIS HPC that
will help accelerate this work. All of these options are more complex than they
appear; the radar files have significant ancillary files that make both the
first two options non-trivial, and staying on the CReSIS infrastructure doesn't
meet many of our open science requirements (MATLAB based, non-public access,
etc.). However, once the inputs are streamlined, the remainder of the workflow
is more open, and may be a place to focus on depending on what Eliza wants to
prioritize accelerating.

## Leigh Stearns

[Leigh](https://groups.universitylife.upenn.edu/penn-polar/) is another xopr
user, so we'll be coordinating with her to ensure that she has access to the
radar sounding data that she needs to develop her pipeline. Her project is more
exploratory than the others listed above, and she indicated that she would
actually appreciate some help and collaboration on the algorithm development as
well as advice on the Bayesian framework and uncertainty analysis. Leigh is
primarily a modeler, and the output from the project is expected to help and
inform some of her modeling colleagues -- so there's likely some work to be
done on the back-end of this project to see how we can integrate and visualize
the outputs within the Model Data Comparison tool.

## Anna 'Ruthie' Halberstadt

[Ruthie's](https://ruthiehalberstadt.com/) project is somewhat unique compared
to the rest of this list, and focuses on creating metrics for use in model data
comparisons. For our other grantees, we're primarily seeking to see how we can
support their efforts, and their respective libraries-- which may include
feature support in xopr or the MDC tool (i.e., the regridder or visualization
modules).  Ruthie and her student Sara are likely to be users of the regridding
module for their work, but also will likely be developers for the stats module
of the MDC tool. That is, they'll likely be looking to integrate their work
into our MDC tool and framework directly. As with the majority of our grantees,
they're requested help with regridding. We suspect that additional engineering
support needs will become more clear as they spin up; I suspect that there
sprint will be in the second half of sprints that we schedule.
