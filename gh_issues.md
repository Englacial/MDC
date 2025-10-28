# GitHub Research: HEALPix, DGGS, Cloud Storage, and Zarr Integration

Research conducted on 2025-10-23 across xdggs, xarray, and zarr-python repositories.

## Executive Summary

This research reveals significant ongoing development in three key areas:
1. **xdggs**: Active development of HEALPix support with lazy/chunked coordinates for cloud-scale datasets
2. **xarray**: Major refactoring of custom indexes and DataTree for hierarchical data
3. **zarr**: Sharding codec and cloud storage optimization efforts

Key finding: The xdggs project has recently implemented MOC (Multi-Order Coverage) indexes for HEALPix that enable working with massive datasets without loading cell IDs into memory - directly relevant to cloud-based DGGS workflows.

---

## 1. xdggs Repository (HEALPix & DGGS Support)

### Repository: https://github.com/xarray-contrib/xdggs

### Critical Issues

#### Issue #143: Design/Datamodel Decision: Very large datasets
**URL**: https://github.com/xarray-contrib/xdggs/issues/143
**Status**: Open
**Significance**: ⭐⭐⭐⭐⭐ **CRITICAL FOR YOUR WORK**

**Problem**: The current implementation faces a hard scaling limit because `cell_ids` coordinates are loaded into memory by default. For global datasets at high zoom levels (e.g., zoom=15), the coordinate dimension alone exceeds typical system memory.

**Example**: At zoom level 15, a HEALPix dataset has 12 × 4^15 cells, making the cell_ids coordinate larger than most system memory.

**Solutions Discussed**:

1. **MOC (Multi-Order Coverage) Index** - Now implemented in PR #151
   - Uses compressed cell ID ranges instead of individual cell IDs
   - Enables lazy/chunked coordinates backed by dask
   - Works with CF conventions' "tie point index mapping"
   - Supports datasets up to level 29 without memory issues

2. **Three Index Types Proposed** (by @benbovy):
   - **Global datasets**: `HealpixIndex.full(level, order)` using `pandas.RangeIndex` (memory efficient)
   - **Small regional datasets**: Standard `PandasIndex` with all cell IDs
   - **Large regional datasets**: Tie-point coordinates using `pandas.IntervalIndex` + `RangeIndex` per interval

3. **CF Conventions Integration**:
   - Compression by gathering (already in CF)
   - Tie-point index mapping for cell ID ranges
   - No need to materialize full cell coordinate arrays

**Key Quotes**:
> "With #151 this works very quickly... The index itself is not really useful right now, though, and the different methods (`cell_centers`, `cell_boundaries`, etc.) don't support `dask` yet." - @keewis

**Implications for Your Work**:
- You can now work with global HEALPix datasets without memory constraints
- Need to ensure chunked/dask support for downstream operations
- MOC index approach is ideal for cloud-based DGGS data

---

#### Issue #155: Chunked version of the methods
**URL**: https://github.com/xarray-contrib/xdggs/issues/155
**Status**: Open

**Problem**: With PR #151's lazy coordinates, methods like `cell_centers`, `cell_boundaries`, `zoom_to` still trigger computation.

**Solution**: Operations are embarrassingly parallel and should be easily parallelizable through `xr.apply_ufunc`.

**Status**: Work needed to add dask support to core methods.

---

#### Issue #171: Support datatree
**URL**: https://github.com/xarray-contrib/xdggs/issues/171
**Status**: Open

**Request**:
1. Select regions by lat/lon across all DataTree nodes in one call
2. Interactive visualization of multiple DataTree nodes on a single map

**Relevance**: DataTree is xarray's structure for hierarchical/multi-scale data - perfect for multi-resolution DGGS pyramids.

---

#### PR #151: healpix moc index
**URL**: https://github.com/xarray-contrib/xdggs/pull/151
**Status**: Merged (⭐ Major advancement)

**Implementation**:
- Uses `RangeMOCIndex` from EOPF-DGGS/healpix-geo
- Enables opening datasets up to level 29 without keeping cell IDs in memory
- Can skip loading cell IDs if dataset has 12 × 4^level cells (global coverage)
- **Example usage**:
```python
import xdggs
import xarray as xr
import dask.array as dsa

zoom = 15
path = f"https://nowake.nicam.jp/files/220m/data_healpix_{zoom}.zarr"

ds = xr.open_dataset(path, engine="zarr", chunks={}).assign_coords(
    cell_ids=lambda ds: ("cell", dsa.arange(
        ds.sizes["cell"],
        chunks=ds.chunks["cell"],
        dtype="uint64"
    ))
)

grid_info_mapping = {
    "grid_name": "healpix",
    "level": zoom,
    "indexing_scheme": "nested"
}

decoded = ds.dggs.decode(grid_info_mapping, index_kind="moc")
```

**Important**: Avoid naming both coordinate and dimension `cell_ids` - causes xarray to create default pandas index and run out of memory.

---

### Other Notable Issues

#### Issue #16: Spatial indexing
**URL**: https://github.com/xarray-contrib/xdggs/issues/16
**Status**: Open
**Discussion**: Need for efficient spatial queries and indexing operations

#### Issue #66: Spatial sorting or shuffling
**URL**: https://github.com/xarray-contrib/xdggs/issues/66
**Status**: Open
**Discussion**: Optimizing data layout for spatial locality (important for cloud storage)

---

## 2. xarray Repository (Custom Indexes & Zarr)

### Repository: https://github.com/pydata/xarray

### Critical Issues

#### Issue #6293: Explicit indexes: next steps
**URL**: https://github.com/pydata/xarray/issues/6293
**Status**: Open
**Significance**: ⭐⭐⭐⭐ **Foundation for custom DGGS indexes**

**Purpose**: Tracking issue for completing xarray's custom index refactoring (started in #5692).

**Key Remaining Work**:
- Many Dataset/DataArray methods need index compatibility checks
- Public API for assigning/resetting indexes
- Documentation and examples
- 3rd party index plugin system (entrypoints)

**Relevant Points**:
- Custom indexes are now first-class citizens in xarray's data model
- `DGGSIndex` in xdggs builds on this foundation
- Plan to deprecate pandas.MultiIndex special cases

**Status**: Ongoing refactoring, but core functionality is merged and working.

---

#### Issue #5376: Multi-scale datasets and custom indexes
**URL**: https://github.com/pydata/xarray/issues/5376
**Status**: Open
**Significance**: ⭐⭐⭐⭐ **Relevant for multi-resolution DGGS**

**Proposal**: Use custom indexes to handle multi-scale/pyramidal datasets.

**API Concept**:
```python
# Set an index for x/y/z coordinates
xyz_dataset.set_index(
    ('x', 'y', 'z'),
    ImagePyramidIndex,
    reduction=np.mean,
    pre_compute_scales=(2, 2),
)

# Get a slice at appropriate resolution
xyz_slice = xyz_dataset.sel_and_rescale(
    x=slice(...),
    y=slice(...),
    z=slice(...)
)
```

**Discussion Points**:
- Index approach vs. hierarchical Dataset groups (DataTree)
- Dynamic downsampling vs. pre-computed pyramids
- Integration with visualization tools
- Both approaches might be complementary

**Implications**: Could provide elegant API for DGGS multi-resolution data.

---

#### Issue #10569: Cannot read Zarr v2 data from SWIFT without consolidated metadata
**URL**: https://github.com/pydata/xarray/issues/10569
**Status**: Open
**Significance**: ⭐⭐⭐ **Important for cloud storage**

**Problem**: Reading Zarr v2 from object storage via HTTPS requires consolidated metadata because HTTP doesn't provide directory listing operations.

**Key Points**:
- HTTPS protocol doesn't support file listings
- Object stores like SWIFT/S3 don't auto-generate index pages
- Consolidated metadata (`.zmetadata`) is **required** for HTTPS access
- Writing v2 with `dimension_separator='/'` to SWIFT fails to create consolidated metadata

**Resolution**:
- Consolidated metadata is necessary for cloud object storage access
- This is by design, not a bug
- fsspec cannot infer directory contents without either HTML index or consolidated metadata

**Implications for Your Work**:
- Always use `consolidated=True` when writing Zarr to cloud storage
- Budget for the metadata consolidation step in workflows
- Consider Zarr v3 which has better metadata handling

---

#### Issue #10579: `open_dataset` creates default indexes sequentially, causing latency in cloud stores
**URL**: https://github.com/pydata/xarray/issues/10579
**Status**: Open

**Problem**: Default index creation happens sequentially, multiplying latency when opening datasets from high-latency cloud stores.

**Impact**: Slow dataset opening from S3/GCS/Azure.

---

#### Issue #9455: `DataTree.to_zarr()` is very slow writing to high latency store
**URL**: https://github.com/pydata/xarray/issues/9455
**Status**: Open

**Problem**: Writing DataTree to cloud storage is very slow due to sequential operations.

**Relevance**: Important for hierarchical DGGS data stored in cloud.

---

#### Other Zarr-Related Issues

##### Issue #2300: zarr and xarray chunking compatibility and `to_zarr` performance
**URL**: https://github.com/pydata/xarray/issues/2300
**Status**: Closed

**Problem**: `da.store` between zarr stores with differing chunks is extremely slow.

**Discussion**: Rechunking strategies and chunk alignment between source and destination.

##### Issue #7987: Existing chunks not being respected on to_zarr()
**URL**: https://github.com/pydata/xarray/issues/7987
**Status**: Closed

**Problem**: Chunks specified in dask arrays not preserved when writing to Zarr.

---

### DataTree Issues (Hierarchical Data)

Multiple open issues around DataTree functionality:
- #10800: Backend documentation doesn't mention DataTree
- #10812: DataTree doesn't inherit non-dimension coordinates
- #9502: Parallelize `map_over_subtree`
- #9106: Support DataTree in top-level functions
- #9511: Performance of deep DataTrees
- #9634: Per-node DataTree chunking

**Relevance**: DataTree is the natural structure for multi-resolution DGGS pyramids.

---

## 3. zarr-python Repository (Cloud Storage & Sharding)

### Repository: https://github.com/zarr-developers/zarr-python

### Critical Issues

#### Consolidated Metadata Issues

##### Issue #3466: Support consolidated metadata in v2 > v3 conversion CLI
**URL**: https://github.com/zarr-developers/zarr-python/issues/3466
**Status**: Open

**Problem**: v2 to v3 conversion doesn't handle consolidated metadata properly.

##### Issue #3313: consolidate_metadata does not scale well
**URL**: https://github.com/zarr-developers/zarr-python/issues/3313
**Status**: Open

**Problem**: Metadata consolidation becomes very slow with large numbers of arrays/groups.

##### Issue #2979: Broken `fill_value` encoding in `consolidated_metadata` when writing format v2 from zarr-python 3
**URL**: https://github.com/zarr-developers/zarr-python/issues/2979
**Status**: Open

##### Issue #2937: Consolidated metadata preferences on a Store-specific basis
**URL**: https://github.com/zarr-developers/zarr-python/issues/2937
**Status**: Open

**Discussion**: Different stores may have different requirements for consolidated metadata.

---

#### Sharding Issues

##### Issue #2346: Sharding in zarr v2
**URL**: https://github.com/zarr-developers/zarr-python/issues/2346
**Status**: Open
**Significance**: ⭐⭐⭐ **Important optimization**

**Proposal**: Since sharding is implemented as a codec, it can be used with v2 arrays.

**Benefits**:
- Reduce number of small files
- Improve performance with hierarchical chunking
- Better cloud storage efficiency

**Status**: Needs testing with zarr-python v3.

##### Issue #3014: Incremental writing to a sharded array slower than without sharding
**URL**: https://github.com/zarr-developers/zarr-python/issues/3014
**Status**: Open

**Problem**: Sharding degrades performance for incremental writes.

##### Issue #3421: Example that reveals inefficient sharded writes
**URL**: https://github.com/zarr-developers/zarr-python/issues/3421
**Status**: Open

**Problem**: Writing patterns that cause excessive shard rewrites.

##### Issue #2834: Bug with setitem with oindex and sharding
**URL**: https://github.com/zarr-developers/zarr-python/issues/2834
**Status**: Open

##### Issue #3085: Extreme Slowness/Timeouts with Large Dataset (Chunked, Multi-Band Zarr Store)
**URL**: https://github.com/zarr-developers/zarr-python/issues/3085
**Status**: Open

**Problem**: Performance degradation with large multi-band datasets.

---

#### Cloud Storage & Performance

##### Issue #2988: [v3] Caching from `fsspec` doesn't work with `FSSpecStore`
**URL**: https://github.com/zarr-developers/zarr-python/issues/2988
**Status**: Open

**Problem**: fsspec's caching mechanisms don't integrate properly with Zarr v3 stores.

**Impact**: Loss of potential performance gains from caching.

##### Issue #2710: Performance regression in V3
**URL**: https://github.com/zarr-developers/zarr-python/issues/2710
**Status**: Open

**Problem**: Zarr v3 shows performance regressions vs v2 in some workloads.

##### Issue #1758: [v3] Request coalescing
**URL**: https://github.com/zarr-developers/zarr-python/issues/1758
**Status**: Open

**Feature**: Combine multiple small requests into fewer larger requests for cloud storage efficiency.

---

## 4. Key Web Search Findings

### HEALPix & DGGS with xarray

**Source**: ISPRS Archives (https://isprs-archives.copernicus.org/articles/XLVIII-4-W12-2024/75/2024/)

**Key Points**:
- xdggs provides unified API for HEALPix, H3, DGGRID ISEA7H
- Uses custom Xarray-compatible DGGSIndex
- Promotes FAIR data practices
- Integrates with Dask for cloud scalability

---

### Zarr Cloud Optimization Best Practices

**Source**: Pangeo Discourse

**Key Recommendations**:
1. **Chunk Size**:
   - Zarr Tutorial recommends ≥1MB
   - Amazon S3 Best Practices: 8-16MB for byte-range requests

2. **Consolidated Metadata**: Essential for cloud performance
   - Reduces number of HTTP requests
   - Critical for HTTPS access to object storage

3. **Rechunking**:
   - Use rechunker or pangeo-forge recipes
   - Plan chunk layout for access patterns

4. **Sharding** (Zarr v3):
   - Reduces file count
   - Improves small-chunk performance
   - Typical shard size: ~1GB

---

### Zarr v3 Sharding Performance

**Sources**:
- zarr_benchmarks repo (https://github.com/zarrs/zarr_benchmarks)
- Zarr documentation

**Findings**:
- Sharding provides 2-5x improvement for small random reads
- Can reduce 100,000 chunks to 100 sharded files
- Slower for large sequential I/O
- Best for small chunks with random access patterns

**Benchmarks**: Available at zarrs/zarr_benchmarks repository

---

## 5. Code Examples and Solutions

### Working with Large HEALPix Datasets (from Issue #143)

```python
import xdggs
import xarray as xr
import dask.array as dsa

# Open large HEALPix dataset without memory issues
zoom = 15
path = f"https://nowake.nicam.jp/files/220m/data_healpix_{zoom}.zarr"

# Load with lazy coordinates
ds = xr.open_dataset(path, engine="zarr", chunks={})

# Create lazy cell_ids coordinate (important: different name than dimension!)
cell_ids = xr.DataArray(
    dsa.arange(ds.sizes["cell"], chunks=ds.chunks["cell"], dtype="uint64"),
    dims=["cell"],
    name="cell_ids"
)
ds = ds.assign_coords({"cell_ids": cell_ids})

# Decode with MOC index
ds.cell_ids.attrs = {
    "grid_name": "healpix",
    "level": zoom,
    "indexing_scheme": "nested",
}

decoded = ds.dggs.decode(
    {"grid_name": "healpix", "level": zoom, "indexing_scheme": "nested"},
    index_kind="moc"
)
```

**Critical**: Use different names for coordinate (`cell_ids`) and dimension (`cell`) to avoid automatic pandas index creation.

---

### CF-Compliant HEALPix with Tie Points (Proposed)

```python
# CDL notation for compressed regional dataset
dimensions:
    time = 100;
    cell = 15234;      # actual data dimension
    tp_cell = 44;      # tie points

variables:
    float time(time);
        time:units = "days since 2024-09-04";

    float temp(time, cell);
        temp:standard_name = "air_temperature";
        temp:units = "K";
        temp:grid_mapping = "crs: cell";

    char l_interpolation;
        l_interpolation:interpolation_name = "linear";
        l_interpolation:tie_point_mapping = "cell: cell_indices tp_cell";

    int cell(tp_cell);               # compressed cell IDs
    int cell_indices(tp_cell);       # indices into full cell space

    int crs;
        crs:grid_mapping_name = "healpix";
        crs:refinement_level = 7;
        crs:healpix_order = "nested";

data:
    cell_indices = 0, 20, 21, 25, 26, ...;
    cell = 1114, 1134, 2564, 2568, 6581, ...;
```

This approach uses CF's tie-point interpolation to represent cell ID ranges compactly.

---

## 6. Recommendations for Your Work

### Immediate Actions

1. **Use xdggs with MOC Index**:
   - Upgrade to latest xdggs (includes PR #151)
   - Use `index_kind="moc"` for large datasets
   - Ensure cell coordinate and dimension have different names

2. **Zarr Best Practices**:
   - Always use `consolidated=True` for cloud storage
   - Target 8-16MB chunk sizes for S3/GCS
   - Consider Zarr v3 with sharding for many small chunks

3. **Cloud Storage**:
   - Budget for consolidation step in write workflows
   - Use HTTPS with consolidated metadata
   - Consider request coalescing patterns

### Medium-Term Considerations

1. **DataTree for Multi-Resolution**:
   - Monitor DataTree development for hierarchical DGGS
   - Consider pyramidal storage structure
   - Watch issues #171, #9455, #9634

2. **Chunking Strategy**:
   - Design chunks for your access patterns
   - Balance between chunk count and chunk size
   - Consider sharding for high-resolution levels

3. **Custom Index Development**:
   - Follow xarray issue #6293 for custom index API stability
   - Consider contributing DGGS-specific optimizations
   - Explore coordinate transform API for lat/lon <-> cell_id

### Long-Term Strategic

1. **CF Conventions**:
   - Monitor HEALPix CF standardization (https://github.com/cf-convention/cf-conventions/issues/433)
   - Plan for CF-compliant metadata
   - Consider compression-by-gathering patterns

2. **Integration Points**:
   - xpublish for serving DGGS data
   - Integration with visualization tools (lonboard, hvplot)
   - Pangeo ecosystem compatibility

3. **Performance Optimization**:
   - Benchmark your specific access patterns
   - Consider spatial sorting (issue #66)
   - Explore bounding box indexing for regional queries

---

## 7. Key Contacts and Contributors

- **@benbovy**: xdggs lead, xarray custom index architect
- **@keewis**: xdggs core contributor, MOC index implementation
- **@jbusecke**: Cloud-scale dataset use cases
- **@d70-t**: HEALPix CF conventions expertise
- **@tinaok**: Large-scale HEALPix workflows

---

## 8. Related Resources

### Documentation
- xdggs: https://xdggs.readthedocs.io/
- xarray custom indexes: https://docs.xarray.dev/en/stable/internals/how-to-create-custom-index.html
- Zarr v3 spec: https://zarr-specs.readthedocs.io/en/latest/v3/core/v3.0.html
- Sharding codec: https://zarr-specs.readthedocs.io/en/latest/v3/codecs/sharding-indexed/

### Community
- Pangeo Discourse: https://discourse.pangeo.io/
- CF Conventions HEALPix discussion: https://github.com/cf-convention/cf-conventions/issues/433
- awesome-HEALPix: https://github.com/pangeo-data/awesome-HEALPix

### Benchmarks
- zarr_benchmarks: https://github.com/zarrs/zarr_benchmarks
- Zarr performance guide: https://zarr.readthedocs.io/en/stable/user-guide/performance.html

---

## 9. Summary of Critical Findings

### ⭐⭐⭐⭐⭐ Must Know

1. **xdggs PR #151** (Merged): MOC index enables massive HEALPix datasets without memory constraints
2. **Consolidated metadata is required** for cloud object storage access via HTTPS
3. **Chunk sizes**: 8-16MB recommended for cloud storage (S3 best practices)

### ⭐⭐⭐⭐ Very Important

4. **xarray Issue #6293**: Custom index refactoring ongoing - API may evolve
5. **Zarr v3 sharding**: Major performance improvement for many small chunks
6. **DataTree**: Natural structure for multi-resolution DGGS pyramids

### ⭐⭐⭐ Important

7. **CF HEALPix standardization**: In progress, will affect metadata conventions
8. **Spatial sorting** (xdggs #66): Can improve cloud storage performance
9. **Request coalescing** (zarr #1758): Reduces latency for cloud reads

---

## Conclusion

The ecosystem for working with HEALPix DGGS data in the cloud is rapidly maturing:

- **xdggs** has solved the memory scaling problem with MOC indexes
- **xarray** provides the foundation through custom indexes and DataTree
- **zarr** offers cloud-optimized storage with sharding in v3

The main remaining challenges are:
1. Adding dask support to xdggs methods (issue #155)
2. Optimizing DataTree performance for cloud storage
3. Finalizing CF conventions for HEALPix metadata

Your timing is excellent - the core capabilities are now available, and you can contribute to the remaining optimization work.
