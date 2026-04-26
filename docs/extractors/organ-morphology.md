# Organ Morphology Extractor

`OrganMorphologyExtractor` computes basic shape, size, and density features for every organ present in the segmentation. It is the most general-purpose extractor and applies to all 117 TotalSegmentator organs.

**Required masks:** `organ_seg`

---

## Methods

### Volume

```
volume_cc = voxel_count × (spacing_x × spacing_y × spacing_z) / 1000
```

Voxel count is determined by `np.count_nonzero(organ_mask)`. The denominator converts mm³ to mL (1 mL = 1000 mm³).

---

### HU Statistics

Raw CT values within the organ mask are extracted and clipped to a per-organ HU range before computing statistics. Clipping removes implant artifacts and air voxels from incompletely segmented regions.

| Statistic | Formula |
|-----------|---------|
| `hu_mean` | `np.mean(clipped_voxels)` |
| `hu_std` | `np.std(clipped_voxels)` |
| `hu_median` | `np.median(clipped_voxels)` |
| `hu_min` | `np.min(clipped_voxels)` |
| `hu_max` | `np.max(clipped_voxels)` |
| `hu_p10`, `hu_p25`, `hu_p75`, `hu_p90` | Percentiles |

---

### HU Histogram (8 bins)

Voxel values are binned into 8 fixed HU intervals:

| Bin | Range |
|-----|-------|
| `hu_hist_0` | −1000 to −500 HU (air) |
| `hu_hist_1` | −500 to −50 HU (fat) |
| `hu_hist_2` | −50 to 0 HU (near-water fat) |
| `hu_hist_3` | 0 to 50 HU (water/fluid) |
| `hu_hist_4` | 50 to 100 HU (soft tissue) |
| `hu_hist_5` | 100 to 200 HU (dense tissue) |
| `hu_hist_6` | 200 to 500 HU (bone-adjacent) |
| `hu_hist_7` | 500 to 1000 HU (cortical bone) |

Each bin value is a **normalized fraction** (0–1) summing to 1 across bins. Output per organ: `{organ}_hu_hist_0` through `{organ}_hu_hist_7`.

---

### Bounding Box

Uses `scipy.ndimage.find_objects()` to find the tight bounding box around each organ label in one pass over the segmentation. Dimensions are reported in mm:

```
bbox_x_mm = (x_max - x_min + 1) × spacing_x    # Left-Right extent
bbox_y_mm = (y_max - y_min + 1) × spacing_y    # Anterior-Posterior extent
bbox_z_mm = (z_max - z_min + 1) × spacing_z    # Cranio-Caudal extent
```

---

### Centroid

Computed with `scipy.ndimage.center_of_mass()`. Output in voxel coordinates and converted to mm:

```
centroid_x_mm = centroid_x_vox × spacing_x
centroid_y_mm = centroid_y_vox × spacing_y
centroid_z_mm = centroid_z_vox × spacing_z
```

---

### Shannon Entropy

Computed only for organs with high internal heterogeneity: **liver**, **kidney_right**, **kidney_left**, **spleen**, **pancreas**.

1. Build a 50-bin histogram of raw HU values within the organ
2. Normalize to a probability distribution p
3. Entropy = −∑ p × log₂(p)

```
entropy = -np.sum(p[p > 0] * np.log2(p[p > 0]))
```

A higher entropy indicates more complex internal texture (e.g., heterogeneous liver parenchyma in steatosis or cirrhosis).

---

### Gallbladder Calcification

For the gallbladder specifically, voxels with HU > 130 are classified as calcification:

```
gallbladder_calc_fraction = count(HU > 130) / total_voxels
```

---

### Derived Features (Liver only)

| Feature | Formula |
|---------|---------|
| `liver_spleen_hu_diff` | `liver_hu_mean − spleen_hu_mean` |
| `liver_density_cv` | `liver_hu_std / liver_hu_mean` (if mean > 0) |
| `liver_occupancy_extent` | `liver_volume_cc / (bbox_x × bbox_y × bbox_z × voxel_vol)` |

---

## Feature Naming

All features are prefixed with the organ name exactly as it appears in the TotalSegmentator mapping:

```
{organ_name}_{feature}
```

Examples: `liver_volume_cc`, `spleen_hu_mean`, `kidney_right_hu_hist_3`, `liver_entropy`.

---

## Feature Table (per organ)

| Feature | Unit | Description |
|---------|------|-------------|
| `{organ}_volume_cc` | mL | Total organ volume |
| `{organ}_hu_mean` | HU | Mean HU (clipped) |
| `{organ}_hu_std` | HU | Standard deviation |
| `{organ}_hu_median` | HU | Median HU |
| `{organ}_hu_min` | HU | Minimum HU |
| `{organ}_hu_max` | HU | Maximum HU |
| `{organ}_hu_p10` | HU | 10th percentile |
| `{organ}_hu_p25` | HU | 25th percentile |
| `{organ}_hu_p75` | HU | 75th percentile |
| `{organ}_hu_p90` | HU | 90th percentile |
| `{organ}_hu_hist_0..7` | fraction | 8-bin normalized HU histogram |
| `{organ}_bbox_x_mm` | mm | Bounding box L-R extent |
| `{organ}_bbox_y_mm` | mm | Bounding box A-P extent |
| `{organ}_bbox_z_mm` | mm | Bounding box C-C extent |
| `{organ}_centroid_x_mm` | mm | Centroid X position |
| `{organ}_centroid_y_mm` | mm | Centroid Y position |
| `{organ}_centroid_z_mm` | mm | Centroid Z position |
| `{organ}_entropy` | bits | Shannon entropy (selected organs) |
| `gallbladder_calc_fraction` | fraction | Fraction of GB voxels > 130 HU |
| `liver_spleen_hu_diff` | HU | Liver mean − spleen mean HU |
| `liver_density_cv` | — | Coefficient of variation of liver HU |
| `liver_occupancy_extent` | fraction | Volume / bounding box volume |
