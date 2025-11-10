# Model Data Comparision - Design and Reference

This repository is collates ideas, designs, references, and discussion for
creating a general purpose model-to-data, model-to-model, and data-to-data
tool, initially aimed at serving the cyrosphere and ice sheet modelling
research community.

Start with [why_this_project](./why_this_project.md) for an overview of what
we're trying to achieve and the problems with the current state of things.

We have an outline for some specific tasks that we suspect we should tackle
during our [first sprint](./first_sprint.md) -- but it's probably most helpful
to read our [future sprints](./future_sprints.md) document first, since that
will give a better sense of where this project is going long term.

Other documents to explore on the following pages and directories:

 - [initial_use_cases](./initial_use_cases.md) describes example model-data
   comparisons we're trying to make easy
 - [vision](./vision.md) is another broad document that hints at the cleaner API
   we're hoping to arrive at

For our regridding module, check out the [regridder](./regridder) folder
generally, and these docs in specific:

 - [design_doc](./regridder/design_doc.md) is our working draft design document
 - [healpix.md](./regridder/healpix.md) is an overview of software maturity for
   supporing healpix meshes

The [reference](./reference) folder has both papers and summaries that may be
useful. Moving forward, we expect to add additional folders and docs for the
other two modules that are planned as part of MDC -- vizualisation, and stats.
