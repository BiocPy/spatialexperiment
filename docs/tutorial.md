---
file_format: mystnb
python_version: 3.14
---

# Represent Spatially Resolved transcriptomics data

The `spatialexperiment` package provides the `SpatialExperiment` (SPE) container class, extending `SingleCellExperiment` (SCE) to support spatially resolved transcriptomics (SRT) and spatial-omics datasets. 

It adds dedicated slots for:
1. **Spatial Coordinates**: Location of spots or cells in the spatial grid/coordinate space.
2. **Image Data**: Associated tissue images and metadata, such as scaling factors.

:::{important}
Like other BiocPy containers, the design adheres to the Bioconductor standard where **rows** correspond to features (e.g., genes) and **columns** represent cells/spots.
:::

## Installation

To get started, install the package from [PyPI](https://pypi.org/project/spatialexperiment/):

```bash
pip install spatialexperiment
```

## Construction

The `SpatialExperiment` constructor extends `SingleCellExperiment` with two main arguments:
* `spatial_coords`: A `BiocFrame` or `np.ndarray` of spatial coordinates.
* `img_data`: A `BiocFrame` containing image metadata (`sample_id`, `image_id`, `data` representing `VirtualSpatialImage`, and `scale_factor`).

In addition, the `column_data` must include a `sample_id` column to map columns (spots) to their corresponding images.

Let's generate mock SRT data:

```python
import numpy as np
from biocframe import BiocFrame
from spatialexperiment import SpatialExperiment, construct_spatial_image_class

nrows = 100 # Features
ncols = 50  # Spots/cells

# Mock counts assay
counts = np.random.rand(nrows, ncols)

# Feature metadata
row_data = BiocFrame({
    "gene_ids": [f"gene_{i}" for i in range(nrows)]
})

# Spot/cell metadata
col_data = BiocFrame({
    "cell_id": [f"spot_{i}" for i in range(ncols)],
    "sample_id": ["sample_1"] * 25 + ["sample_2"] * 25
})

# Spatial coordinates
spatial_coords = BiocFrame({
    "x": np.random.uniform(0.0, 100.0, size=ncols),
    "y": np.random.uniform(0.0, 100.0, size=ncols)
})

# Image metadata
img_data = BiocFrame({
    "sample_id": ["sample_1", "sample_2"],
    "image_id": ["lowres", "lowres"],
    "data": [
        construct_spatial_image_class("tests/images/sample_image1.jpg"),
        construct_spatial_image_class("tests/images/sample_image3.jpg"),
    ],
    "scale_factor": [0.5, 0.5]
})
```

Now let's construct the `SpatialExperiment` object:

```python
spe = SpatialExperiment(
    assays={"counts": counts},
    row_data=row_data,
    column_data=col_data,
    spatial_coords=spatial_coords,
    img_data=img_data
)

print(spe)
```

## Spatial Coordinates accessors

Spatial coordinates can be retrieved or modified using functional methods or properties.

### Retrieve Spatial Coordinates

```python
# Get spatial coordinates (returns a BiocFrame)
coords = spe.spatial_coords
print(coords.head(5))
```

### Retrieve Coordinate Names

```python
names = spe.spatial_coords_names
print("Coordinate dimensions:", names)
```

### Modify Spatial Coordinates

Use the functional `set_spatial_coordinates` method to assign new spatial coordinates:

```python
new_coords = BiocFrame({
    "x_new": np.random.uniform(10.0, 50.0, size=ncols),
    "y_new": np.random.uniform(10.0, 50.0, size=ncols)
})

spe = spe.set_spatial_coordinates(new_coords)
print(spe.spatial_coords_names)
```

---

## Image Data Methods

The `img_data` slot contains image metadata, allowing lookup of specific images or scale factors.

### Accessing Image Data

```python
print(spe.img_data)
```

### Fetching Specific Images

The `get_img` method lets you fetch specific images by `sample_id` and/or `image_id`:

```python
img = spe.get_img(sample_id="sample_1", image_id="lowres")
print(type(img))
```

---

## Slicing / Subsetting

`SpatialExperiment` objects can be sliced along either rows (features) or columns (spots/cells). Slicing will automatically subset the spatial coordinates and filter relevant images from `img_data`.

### Slice by Row

```python
# Slice first 10 genes
spe_genes = spe[0:10, :]
print(spe_genes.shape)
```

### Slice by Column

```python
# Slice first 5 spots
spe_spots = spe[:, 0:5]
print(spe_spots.shape)
print(spe_spots.spatial_coords.shape)
```

---

## Combining Experiments

You can combine multiple `SpatialExperiment` objects along their columns (spots/cells) using `combine_columns` or `relaxed_combine_columns`. Spatial coordinates and image metadata will be combined accordingly.

```python
import biocutils as ut

# Combine columns from the same experiment (with warning due to duplicate sample_ids)
combined = ut.combine_columns(spe, spe)
print(combined.shape)
```
