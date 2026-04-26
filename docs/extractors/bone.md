# Bone Extractor

`BoneExtractor` quantifies vertebral bone health for opportunistic osteoporosis screening and vertebral compression fracture detection. It processes all thoracic and lumbar vertebrae (T1–L5) from the TotalSegmentator segmentation.

**Required masks:** `organ_seg`

---

## Vertebrae Processed

Vertebrae are iterated in **craniocaudal order** (T1 most superior → L5 most inferior):

```
T1 → T2 → ... → T12 → L1 → L2 → L3 → L4 → L5
```

!!! note "Vertebral height = Z axis"
    In RAS+, vertebral HEIGHT is the extent along **axis 2 (Z)**, not axis 0. The craniocaudal extent of each vertebra is `(z_max − z_min) × spacing_z`.

---

## Methods

### Trabecular HU (BMD Proxy)

Cortical bone is excluded by **binary erosion** of the vertebra mask:

```
erosion_iter   = max(1, round(3mm / avg_spacing))
trabecular_mask = binary_erosion(vertebra_mask, iterations=erosion_iter)
# if < 50 voxels remain after erosion, use full mask
trabecular_voxels = clip(CT[trabecular_mask], -200, 1500)
```

Clipping to −200 to 1500 HU removes air artifacts from partial segmentation and extreme implant values.

**Per-vertebra output:**

| Feature | Unit | Description |
|---------|------|-------------|
| `{vertebra}_mean_hu` | HU | Mean trabecular HU |
| `{vertebra}_std_hu` | HU | Std of trabecular HU |

---

### Global Lumbar BMD (L1–L4)

Trabecular voxels from L1, L2, L3, and L4 are **pooled** into a single array for overall lumbar assessment:

| Feature | Unit | Description |
|---------|------|-------------|
| `lumbar_hu_mean` | HU | Mean HU across L1–L4 trabecular bone |
| `lumbar_hu_std` | HU | Std across L1–L4 |
| `lumbar_hu_p10` | HU | 10th percentile |
| `lumbar_hu_p25` | HU | 25th percentile |
| `lumbar_hu_p50` | HU | Median |
| `lumbar_hu_p75` | HU | 75th percentile |
| `lumbar_hu_p90` | HU | 90th percentile |

!!! info "Interpreting HU as BMD"
    Lumbar trabecular HU correlates with bone mineral density measured by DXA. Values below ~100–120 HU suggest osteopenia; below ~80 HU suggests osteoporosis (thresholds vary by scanner calibration).

---

### Vertebral Height

Height is measured as the craniocaudal (Z-axis) extent of each vertebra in the RAS+ frame:

```
height_mm = (z_max - z_min) × spacing_z
```

| Feature | Unit | Description |
|---------|------|-------------|
| `{vertebra}_height_mm` | mm | Craniocaudal height of vertebra |

---

### Compression Ratio

For each vertebra (T2 through L4), the height is compared to the average of its immediate neighbors:

```
avg_neighbor_height = (prev_height + next_height) / 2
compression_ratio   = current_height / avg_neighbor_height
```

A ratio < ~0.80 indicates compression fracture (the vertebra is shorter than its neighbors). The first (T1) and last (L5) vertebrae are excluded as they lack both neighbors.

**Global compression statistics** (across all T2–L4):

| Feature | Unit | Description |
|---------|------|-------------|
| `{vertebra}_compression_ratio` | — | Height ratio vs neighbors |
| `min_compression_ratio` | — | Minimum across all vertebrae (most severe) |
| `mean_compression_ratio` | — | Mean compression ratio |
| `std_compression_ratio` | — | Std of compression ratios |

---

### Fast Mode

With `fast_mode=True`, a single `ndimage.find_objects()` pass pre-computes bounding boxes for all labels. Each vertebra is then processed on a small cropped sub-volume (with 10 mm padding to avoid erosion boundary effects), rather than eroding the full image. Substantially reduces memory and CPU use on high-resolution scans.

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `{vertebra}_mean_hu` | HU | Mean trabecular HU (T1–L5) |
| `{vertebra}_std_hu` | HU | Std trabecular HU (T1–L5) |
| `{vertebra}_height_mm` | mm | Craniocaudal height (T1–L5) |
| `{vertebra}_compression_ratio` | — | Height / avg neighbor height (T2–L4) |
| `lumbar_hu_mean` | HU | Pooled L1–L4 trabecular HU mean |
| `lumbar_hu_std` | HU | Pooled L1–L4 trabecular HU std |
| `lumbar_hu_p10` | HU | L1–L4 10th percentile |
| `lumbar_hu_p25` | HU | L1–L4 25th percentile |
| `lumbar_hu_p50` | HU | L1–L4 50th percentile (median) |
| `lumbar_hu_p75` | HU | L1–L4 75th percentile |
| `lumbar_hu_p90` | HU | L1–L4 90th percentile |
| `min_compression_ratio` | — | Minimum compression ratio (most severe) |
| `mean_compression_ratio` | — | Mean compression ratio across spine |
| `std_compression_ratio` | — | Std of compression ratios |

Vertebra names: `vertebrae_T1` through `vertebrae_T12`, `vertebrae_L1` through `vertebrae_L5`.
