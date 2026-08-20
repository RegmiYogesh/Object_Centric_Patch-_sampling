# Object-Centric Patch Sampling

Code accompanying the paper:

> **Mitigating Class Imbalance and False-Negative Supervision in Remote Sensing Semantic
> Segmentation Using Object-Centric Patch Sampling**
> Yogesh Regmi, Sandeep Gautam, Abinash Silwal, Gaurav Parajuli, Roshan Bhandari, Tri Dev Acharya
> *Remote Sensing*, 2026. MDPI.

This is the implementation used in the paper, published so the method can be inspected and applied
to other datasets.

---

## The method

Patch-based training is standard in remote sensing, but the way patches are *sampled* is usually
treated as a preprocessing detail. Sliding-window and random sampling create two problems at the
data construction stage: in sparse-object scenes most patches contain only background, and where
annotation is incomplete, unlabeled objects inside a patch are treated as background.

Object-centric patch sampling anchors each training patch to the geometric centroid of an
annotated object polygon, so every anchored patch contains at least one target instance and
sampling stays close to verified annotations. A controlled proportion of background-only patches
is added back to preserve contextual variety.

The method acts entirely on the training data — no change to the network architecture, loss
function or training schedule is required. Full description and results are in the paper.

---

## Applying it to your own data

You need two things:

1. A georeferenced raster (GeoTIFF).
2. A matching vector layer of annotated objects (Shapefile or GeoJSON) in the same CRS.

`Single_Class_Object_Centric.ipynb` is the cleanest starting point. The patch-generation section
takes the raster and vector paths, computes a centroid per polygon using the discrete Shoelace
formula, and writes image patches with their rasterised masks. Adjust the raster and vector paths,
the patch size and the background ratio, then run that section only — the training cells below it
are specific to the datasets in the paper.

Parameters worth knowing before you change them:

| Parameter               | Value used                                | Note                                                     |
| ----------------------- | ----------------------------------------- | -------------------------------------------------------- |
| Patch size              | 256 px (satellite, aerial), 1024 px (UAV) | large enough to contain a whole object                   |
| Centroid merge distance | 32 px                                     | centroids closer than this merge into one cluster centre |
| Random offset           | ±16 px                                   | adds variety without pushing targets out of the patch    |
| Background allocation   | ~20%                                      | sampled at least one patch-width from any target         |

Near tile edges the centroid is shifted inward so the patch stays inside the raster, which avoids
zero-padding artefacts.

```
python >= 3.9
tensorflow / keras
numpy, scipy
rasterio, geopandas, shapely
scikit-learn
matplotlib
```

---

## Repository layout

The notebooks are the experiment record. Each dataset has one notebook per sampling strategy, so
the three can be compared directly.

**Cotton fields — Sentinel-2, Queensland, Australia**

| Notebook                                     | Sampling strategy |
| -------------------------------------------- | ----------------- |
| `Cotton_Segmentation_Sliding_Window.ipynb` | sliding-window    |
| `Cotton_Segmentation_Random.ipynb`         | random            |
| `Cotton_Segmentation_Centroidbased.ipynb`  | object-centric    |

**Buildings — NAIP aerial imagery, USA**

| Notebook                                        | Sampling strategy |
| ----------------------------------------------- | ----------------- |
| `Building_Segmentation_Sliding_Window.ipynb`  | sliding-window    |
| `Building_Segmentation_Random.ipynb`          | random            |
| `Building_Segmentation_CentroidBasedv2.ipynb` | object-centric    |

**Water bodies — UAV orthophotos, Terai, Nepal**

| Notebook                | Sampling strategy |
| ----------------------- | ----------------- |
| `Pond_Sliding.ipynb`  | sliding-window    |
| `Pond_Centroid.ipynb` | object-centric    |

**Cross-architecture and multi-class**

| Notebook                              | Purpose                                       |
| ------------------------------------- | --------------------------------------------- |
| `Single_Class_Object_Centric.ipynb` | single-class object-centric baseline          |
| `Singleclass_Deeplabv3+.ipynb`      | single-class under DeepLabV3+                 |
| `Multi_class_cotton_Water.ipynb`    | multi-class (cotton + water) patch generation |
| `Multiclass_Deeplabv3+.ipynb`       | multi-class under DeepLabV3+                  |

**Other files**

| Path                          | Contents                                       |
| ----------------------------- | ---------------------------------------------- |
| `BuildingDetectionPatches/` | example patches from the NAIP building dataset |
| `Pond_data/Centroid/`       | centroid-derived patches for the UAV dataset   |
| `Ponds.h5`                  | trained weights for the UAV water body model   |

---

## Data

The datasets are not distributed with this code and carry their own terms:

- **Sentinel-2** — Copernicus open data, European Space Agency.
- **NAIP** — public domain, USDA Farm Service Agency.
- **UAV orthophotos (Nepal)** — collected by the authors.

Annotation layers were produced by the authors. Contact the maintainers for access.

---

## Citation

```bibtex
@article{regmi2026objectcentric,
  title   = {Mitigating Class Imbalance and False-Negative Supervision in Remote Sensing
             Semantic Segmentation Using Object-Centric Patch Sampling},
  author  = {Regmi, Yogesh and Gautam, Sandeep and Silwal, Abinash and
             Parajuli, Gaurav and Bhandari, Roshan and Acharya, Tri Dev},
  journal = {Remote Sensing},
  year    = {2026},
  publisher = {MDPI}
}
```

---

## License

MIT — see [LICENSE](LICENSE). Applies to the code and the trained weights in this repository, not
to the underlying imagery or annotations.

## Maintainers

- Yogesh Regmi — [@RegmiYogesh](https://github.com/RegmiYogesh)
- Abinash Silwal — [@abinashsilwal](https://github.com/abinashsilwal)
