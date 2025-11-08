
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
