# Vascular Extractor

`VascularExtractor` computes comprehensive measurements for three major abdominal vessels: the **aorta**, **portal vein + splenic vein**, and **inferior vena cava (IVC)**. Features support screening for aneurysm, thrombosis, vascular calcification, and contrast phase assessment.

**Required masks:** `organ_seg`

---

## Vessels Analyzed

| Vessel | TotalSegmentator label |
|--------|----------------------|
| Aorta | `aorta` |
| Portal + Splenic vein | `portal_vein_and_splenic_vein` |
| Inferior Vena Cava | `inferior_vena_cava` |

---

## Methods

### Volume & HU Statistics

Standard volume and HU statistics are computed per vessel (mean, std, median, min, max, p10, p25, p75, p90, CV).

**HU histogram (8 bins):** Same bins as OrganMorphology (`hu_hist_0` through `hu_hist_7`), normalized fractions. Used to assess vessel opacification patterns.

**Coefficient of variation:**
```
hu_cv = hu_std / hu_mean   (if hu_mean > 0)
```

---

### Vessel Diameter & Length

Two modes are available:

=== "Standard mode (3D skeletonization)"

    1. Skeletonize the vessel mask with `skimage.morphology.skeletonize`
    2. Compute 3D EDT on the cropped vessel bounding box with anisotropic spacing
    3. Diameter at each skeleton voxel = 2 × EDT radius
    4. Aggregate over all skeleton points

=== "Fast mode (slice-wise 2D EDT)"

    For each axial slice (Z):
    
    1. Extract the 2D cross-section of the vessel mask
    2. Compute 2D EDT using in-plane spacing (X–Y only)
    3. Diameter for this slice = 2 × max(EDT)
    4. Aggregate per-slice diameters
    
    **Length** = number of valid slices × slice thickness × 1.2 (tortuosity correction)
    
    50–100× faster than full 3D skeletonization, minimal accuracy loss.

!!! tip
    Use `VascularExtractor(fast_mode=True)` for batch processing.

**Diameter statistics:**

| Feature | Description |
|---------|-------------|
| `vessel_diam_mean` | Mean diameter across skeleton |
| `vessel_diam_std` | Standard deviation |
| `vessel_diam_p50` | Median diameter |
| `vessel_diam_p75` | 75th percentile |
| `vessel_diam_p90` | 90th percentile |
| `vessel_diam_max` | Maximum diameter |
| `vessel_diam_cv` | CV = std/mean |
| `vessel_length_mm` | Estimated vessel length |

**Aorta-specific:** `max_diam_location_z_mm` — craniocaudal position (in mm) of the maximum diameter point, used for aneurysm localization.

---

### Vessel HU Heterogeneity

Sampled along the skeleton centerline (or approximate centerline in fast mode):

| Feature | Description |
|---------|-------------|
| `hu_heterogeneity_std` | Std of centerline HU values |
| `hu_heterogeneity_range` | Max − min of centerline HU |
| `hu_heterogeneity_cv` | CV of centerline HU |
| `hu_heterogeneity_entropy` | Shannon entropy (50-bin histogram) |

High heterogeneity along the centerline is a marker for focal pathology (thrombus, dissection).

---

### Aortic Calcification

```
calc_mask = (CT > 130 HU) & aorta_mask
aorta_calc_volume_ml = count(calc_mask) × voxel_vol_ml
aorta_calc_fraction  = count(calc_mask) / count(aorta_mask)
```

| Feature | Unit | Description |
|---------|------|-------------|
| `aorta_calc_volume_ml` | mL | Volume of aortic calcifications |
| `aorta_calc_fraction` | fraction | Fraction of aorta with HU > 130 |

---

### Cross-Vessel Features

Computed from the three vessel HU means:

| Feature | Formula | Clinical use |
|---------|---------|-------------|
| `aorta_enhancement` | `max(0, aorta_hu_mean − 50)` | Contrast bolus strength |
| `pv_aorta_hu_ratio` | `pv_hu / aorta_hu` | Portal venous phase quality |
| `ivc_aorta_hu_ratio` | `ivc_hu / aorta_hu` | Venous return opacification |
| `vein_opac_gate` | `1 / (1 + exp((aorta_hu − pv_hu) / 20))` | PV opacification relative to aorta |

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `{vessel}_volume_ml` | mL | Vessel volume |
| `{vessel}_hu_mean` | HU | Mean HU |
| `{vessel}_hu_std` | HU | Std HU |
| `{vessel}_hu_median` | HU | Median HU |
| `{vessel}_hu_min/max` | HU | Min/max HU |
| `{vessel}_hu_p10/p25/p75/p90` | HU | HU percentiles |
| `{vessel}_hu_cv` | — | HU coefficient of variation |
| `{vessel}_hu_hist_0..7` | fraction | 8-bin HU histogram |
| `{vessel}_bbox_x/y/z_mm` | mm | Bounding box dimensions |
| `{vessel}_vessel_length_mm` | mm | Estimated vessel length |
| `{vessel}_vessel_diam_mean/std/p50/p75/p90/max/cv` | mm | Diameter statistics |
| `aorta_max_diam_location_z_mm` | mm | Z position of max aorta diameter |
| `{vessel}_hu_heterogeneity_std/cv/range/entropy` | HU/bits | Centerline heterogeneity |
| `aorta_calc_volume_ml` | mL | Aortic calcification volume |
| `aorta_calc_fraction` | fraction | Aortic calcification fraction |
| `aorta_enhancement` | HU | Aortic enhancement above baseline |
| `pv_aorta_hu_ratio` | — | Portal vein / aorta HU ratio |
| `ivc_aorta_hu_ratio` | — | IVC / aorta HU ratio |
| `vein_opac_gate` | 0–1 | Sigmoid venous opacification gate |

Vessel prefixes: `aorta`, `portal_vein_and_splenic_vein`, `inferior_vena_cava`.
