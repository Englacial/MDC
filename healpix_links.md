# HEALPix Software Ecosystem - State of Maturity

Research conducted: October 2025

## Overview

HEALPix (Hierarchical Equal Area isoLatitude Pixelization) has evolved from its origins in cosmology and CMB analysis into a broader geospatial and Earth observation tool. The ecosystem is mature and actively maintained, with recent developments focusing on multi-resolution support, integration with modern data science tools, and performance optimization through Rust implementations.

---

## Core Python Libraries

### healpy
**Latest versions:** 1.18.1 (March 2025), 1.18.0 (November 2024)

The canonical Python package for HEALPix operations. Based on the HEALPix C++ library with cfitsio dependencies bundled.

- **Repository:** https://github.com/healpy/healpy
- **PyPI:** https://pypi.org/project/healpy/
- **Documentation:** https://healpy.readthedocs.io/
- **Status:** Actively maintained, mature

### healpix (ntessore)
**Latest version:** 2025.1 (July 2025)

Lean implementation supporting NSIDE parameters up to 2^29. Focuses on core functionality without visualization or spherical harmonic transforms.

- **Repository:** https://github.com/ntessore/healpix
- **PyPI:** https://pypi.org/project/healpix/
- **Status:** Actively maintained, minimalist alternative to healpy

### astropy-healpix
**Latest version:** 1.1.2 (February 2025)

BSD-licensed HEALPix package developed by the Astropy project, based on C code by Dustin Lang (astrometry.net). Provides healpy-compatible interface as subset.

- **Repository:** https://github.com/astropy/astropy-healpix
- **PyPI:** https://pypi.org/project/astropy-healpix/
- **Documentation:** https://astropy-healpix.readthedocs.io/
- **Status:** Actively maintained
- **Note:** Performance is not as good as healpy in most cases; focus has been on functionality

---

## Multi-Resolution and Advanced Extensions

### mhealpy
**Latest version:** 0.3.5

Object-oriented wrapper for handling multi-resolution maps (aka multi-order coverage maps or MOC maps). Supports efficient pixel querying, arithmetic operations, adaptive mesh refinement, plotting, and FITS serialization.

- **Documentation:** https://mhealpy.readthedocs.io/
- **Paper:** "Multi-Resolution HEALPix Maps for Multi-Wavelength and Multi-Messenger Astronomy" ([arXiv:2111.11240](https://arxiv.org/abs/2111.11240), [AJ 163:209](https://iopscience.iop.org/article/10.3847/1538-3881/ac6260))
- **Key feature:** Full multi-level Morton addressing for highly resolved regions
- **Status:** Actively maintained, published in peer-reviewed literature

### mortie
**PyPI:** https://pypi.org/project/mortie/

Morton indexing functions using HEALPix, includes interfaces with vaex.

- **Status:** Available on PyPI

---

## Rust Implementations

### cdshealpix
**Latest version:** 0.6.6 (November 2024)

Primary Rust implementation by Centre de Données astronomiques de Strasbourg (CDS). Supports WebAssembly, Python bindings, and multiple platforms.

- **Repository:** https://github.com/cds-astro/cds-healpix-rust
- **Crates.io:** https://crates.io/crates/cdshealpix
- **Docs:** https://docs.rs/cdshealpix
- **Python wrapper:** https://pypi.org/project/cdshealpix/ (v0.7.1)
- **Key features:**
  - BMOC support (MOC with partial/full coverage flags)
  - Ring scheme with any NSIDE (not just powers of 2)
  - Exact cone and ellipse solutions
  - BMI2 PDEP/PEXT instructions for bit interleaving
  - WebAssembly compilation
  - Used by Aladin Lite for sky visualization
- **Status:** Actively maintained, production ready

### moc (cds-moc-rust)

MOC library implementing v2.0 standard including S-MOCs, T-MOCs, ST-MOCs, F-MOCs (Frequency), and SF-MOCs.

- **Repository:** https://github.com/cds-astro/cds-moc-rust
- **Crates.io:** https://crates.io/crates/moc
- **Dependency:** Built on cdshealpix
- **Status:** Actively maintained

### healpix-rs
**Latest version:** Available on crates.io (June 2025)

Alternative Rust implementation by matt-cornell, focusing on NESTED scheme with cleaner API.

- **Repository:** https://github.com/matt-cornell/healpix-rs
- **Crates.io:** https://crates.io/crates/healpix
- **Note:** Borrows code from cdshealpix but reworked API
- **Status:** Active development

### healpix_fits

Thin wrapper over fitsio crate for HEALPix FITS I/O. Supports f32 and f64, map I/O only.

- **Announcement:** [Rust forum](https://users.rust-lang.org/t/a-simple-and-informal-healpix-fits-i-o-crate/69162)
- **Status:** Lightweight, specialized tool

---

## Geoscience-Specific Tools

### healpix-geo
**Latest version:** 0.0.4 (June 2025)

EOPF (Earth Observation Processing Framework) package integrating cds-healpix-rust and cds-healpix-python for geoscience-specific algorithms.

- **Repository:** https://github.com/EOPF-DGGS/healpix-geo
- **PyPI:** https://pypi.org/project/healpix-geo/
- **Documentation:** https://healpix-geo.readthedocs.io/
- **Dependencies:** Uses cdshealpix, moc, and geodesy Rust crates
- **Status:** Early development for Earth observation applications

---

## Integration with Data Science Tools

### vaex

Big data analytics library with built-in HEALPix convenience functions (uses healpy under the hood).

- **Website:** https://vaex.io/
- **Key functions:**
  - `DataFrame.healpix_count()` - count non-missing values in HEALPix arrays
  - `DataFrame.healpix_plot()` - 2D visualization
- **Default conventions:** Gaia data (source_id/34359738368 gives level 12 index)
- **Example:** https://notebook.community/maartenbreddels/vaex/examples/healpix_plotting
- **Status:** Mature, production ready

### uxarray

Xarray extension for unstructured grids that converts HEALPix to UGRID conventions.

- **PyPI:** https://pypi.org/project/uxarray/
- **Documentation:** https://uxarray.readthedocs.io/
- **HEALPix support:** https://uxarray.readthedocs.io/en/latest/user-guide/healpix.html
- **Constructor:** `ux.Grid.from_healpix(zoom)`
- **Supported formats:** UGRID, MPAS, SCRIP, EXODUS, ESMF, GEOS-CS, ICON, FESOM2, HEALPix
- **Cookbook:** Project Pythia HEALPix Cookbook with uxarray tutorials
- **Status:** Actively developed for climate/weather unstructured data

### xdggs

Xarray extension for Discrete Global Grid Systems, developed at OSGeo/Pangeo BiDS'23 codesprint (November 2023).

- **Repository:** https://github.com/xarray-contrib/xdggs
- **Purpose:** Unified API for DGGS in Xarray ecosystem
- **Supported DGGS:** HEALPix, rHEALPix, Uber H3, DGGRID, Google S2, OpenEAGGR
- **Storage formats:** Zarr, GeoParquet
- **Paper:** "XDGGS: A community-developed Xarray package to support planetary DGGS data cube computations" ([ISPRS Archives](https://isprs-archives.copernicus.org/articles/XLVIII-4-W12-2024/75/2024/))
- **Status:** Prototype/early development, part of xarray-contrib

### healpix-convolution

Convolution operations on xarray objects with HEALPix data.

- **Documentation:** https://healpix-convolution.readthedocs.io/
- **Status:** Specialized tool for signal processing on sphere

---

## Machine Learning and Scientific Computing

### jax-healpy

JAX implementation for differentiable HEALPix operations in machine learning pipelines.

- **Documentation:** https://jax-healpy.readthedocs.io/
- **Status:** Available for ML applications

### NNhealpix / sphericalCNN

PyTorch convolutional neural networks on HEALPix pixelized spherical data.

- **Repository:** https://github.com/aasensio/sphericalCNN
- **Original:** https://github.com/ai4cmb/NNhealpix
- **Status:** Research tool for CMB and spherical ML

---

## Scientific Applications

### HERMES

Computational framework for line-of-sight integration over galactic radiative processes, creating HEALPix-compatible sky maps.

- **Status:** Available for astrophysics applications

### Pynkowski

Python package computing Minkowski Functionals on HEALPix maps.

- **Status:** Specialized analysis tool

---

## Community Resources

### awesome-HEALPix

Curated list by Pangeo community of HEALPix libraries, tools, and resources.

- **Repository:** https://github.com/pangeo-data/awesome-HEALPix
- **Purpose:** Central hub for HEALPix ecosystem discovery
- **Related:** https://github.com/LandscapeGeoinformatics/awesome-discrete-global-grid-systems (broader DGGS list)
- **Status:** Active curation

---

## Support in Common Geospatial Tools

### GDAL / PROJ

**Support level:** ✅ **Supported**

HEALPix and rHEALPix (rotated HEALPix) projections available in PROJ.

- **Usage:** `gdalwarp` with proj string: `+proj=healpix +a=1 +ellps=WGS84 +wktext`
- **Limitation:** Cannot generate single HEALPix patches directly
- **Status:** Mature support via PROJ library

### Xarray

**Support level:** ✅ **Via Extension**

Supported through **xdggs** extension (xarray-contrib).

- **Extension:** https://github.com/xarray-contrib/xdggs
- **Status:** Prototype, under active development
- **Alternative:** uxarray for UGRID-converted HEALPix

### Zarr

**Support level:** ✅ **Via xdggs and GDAL**

- **Path 1:** xdggs can store DGGS-indexed data in Zarr format
- **Path 2:** GDAL Zarr driver supports _ARRAY_DIMENSIONS (xarray compatible)
- **Limitation:** Writing GDAL-compatible Zarr from xarray requires _CRS attributes (currently challenging)
- **Status:** Works but interoperability between tools needs care

---

## Summary and Maturity Assessment

### Highly Mature (Production Ready)
- **healpy** - canonical Python implementation
- **cdshealpix** (Rust + Python) - modern, performant implementation
- **healpix-alchemy** - production database spatial indexing
- **vaex** - big data analytics with HEALPix support
- **GDAL/PROJ** - projection support

### Mature (Active Development)
- **astropy-healpix** - BSD alternative to healpy
- **healpix (ntessore)** - lightweight alternative
- **mhealpy** - multi-resolution maps
- **uxarray** - unstructured grid integration
- **moc (Rust)** - Multi-Order Coverage standard

### Emerging/Prototype
- **xdggs** - unified DGGS interface for xarray
- **healpix-geo** - geoscience-specific tools
- **healpix-rs** - alternative Rust implementation
- **jax-healpy** - ML/differentiable operations

### Specialized/Research Tools
- **NNhealpix/sphericalCNN** - ML on sphere
- **healpix-convolution** - signal processing

### Key Trends

1. **Rust adoption:** High-performance Rust implementations (cdshealpix) with WebAssembly and Python bindings
2. **Multi-resolution:** Growing support for variable-resolution maps (mhealpy, MOC)
3. **Data science integration:** Strong movement toward Xarray/Zarr/Pangeo ecosystem (xdggs, uxarray)
4. **Database support:** Production-ready spatial indexing (healpix-alchemy)
5. **Geoscience expansion:** Moving beyond astronomy into Earth observation (healpix-geo, EOPF)
6. **Standards:** Compliance with OGC DGGS standards, MOC v2.0
7. **Cloud-native:** Focus on Zarr, GeoParquet, and cloud database compatibility

### Gaps and Opportunities

- **xdggs maturity:** Still in prototype phase, needs production hardening
- **Zarr interoperability:** GDAL-xarray compatibility requires better tooling
- **GeoParquet DGGS:** Emerging support for DGGS in GeoParquet format
- **Standardization:** Ongoing work to standardize DGGS across multiple grid types

---

## References

Key papers:
- Górski et al. (2005) "HEALPix: A Framework for High-Resolution Discretization and Fast Analysis of Data Distributed on the Sphere"
- Calabretta & Roukema (2007) "Mapping on the HEALPix grid"
- Teh et al. (2022) "Multiresolution HEALPix Maps for Multiwavelength and Multimessenger Astronomy" ([AJ 163:251](https://iopscience.iop.org/article/10.3847/1538-3881/ac6260))

Official resources:
- HEALPix homepage: https://healpix.sourceforge.io/
- awesome-HEALPix: https://github.com/pangeo-data/awesome-HEALPix
