# Bowel Extractor

`BowelExtractor` extracts features relevant to gastrointestinal pathology including small bowel obstruction, hiatal hernia, colitis/diverticulitis, and bowel wall edema.

**Required masks:** `organ_seg`

---

## Methods

### Small Bowel Obstruction (SBO) — Maximum Diameter

Uses **bounding-box-cropped EDT** to estimate the maximum diameter of the small bowel (small_bowel + duodenum) and colon:

1. Find the tight bounding box around the organ mask
2. Run `scipy.ndimage.distance_transform_edt` on the cropped sub-volume with anisotropic spacing
3. Diameter = 2 × 99th percentile radius (robust to outliers)

```
max_diameter = 2 × np.percentile(EDT[mask], 99)
```

**SBO indicator:** Ratio of small bowel to colon diameter — a ratio > 1 (dilated small bowel with normal or collapsed colon) is suspicious for obstruction.

```
sb_to_colon_diam_ratio = sb_max_diam_mm / colon_max_diam_mm
```

!!! note "Fast mode"
    With `fast_mode=True`, the EDT is computed per axial slice (2D) instead of 3D, which is 10–50× faster and uses only in-plane spacing (X–Y axes).

| Feature | Unit | Description |
|---------|------|-------------|
| `sb_max_diam_mm` | mm | Maximum diameter of small bowel + duodenum |
| `colon_max_diam_mm` | mm | Maximum diameter of colon |
| `sb_to_colon_diam_ratio` | — | Small bowel / colon diameter ratio |

---

### Hiatal Hernia

Detects stomach herniation above the diaphragm (liver dome).

**Algorithm (RAS+ correct):**
1. Find the **maximum Z coordinate** of the liver — this is the liver top (most superior point = diaphragm level)
2. Count stomach voxels where Z > liver_top_z (stomach voxels superior to diaphragm)

```
z_liver_top = max(liver_coords[Z])
herniated = count(stomach_z > z_liver_top)
hiatal_hernia_volume_cc = herniated × voxel_vol_cc
hiatal_hernia_percent   = herniated / total_stomach_voxels
```

| Feature | Unit | Description |
|---------|------|-------------|
| `hiatal_hernia_volume_cc` | mL | Volume of stomach above liver dome |
| `hiatal_hernia_percent` | fraction | Fraction of stomach herniated |
| `stomach_total_volume_cc` | mL | Total stomach volume |

---

### Pericolonic Fat Stranding (Colitis / Diverticulitis)

Creates a **5 mm shell** around the colon via binary dilation and measures fat within the shell:

1. Dilate colon mask by `round(5.0 / avg_spacing)` iterations
2. Shell = dilated region & ~colon_mask & background (seg == 0)
3. Filter shell to fat HU range: −150 to −30 HU

```
shell = binary_dilation(colon, 5mm) & ~colon & background
fat_in_shell = shell_voxels where -150 ≤ HU ≤ -30
colon_surr_fat_mean_hu = mean(fat_in_shell)
colon_surr_fat_p90_hu  = percentile(fat_in_shell, 90)
```

!!! tip
    Elevated pericolonic fat HU (less negative, closer to 0) indicates fat stranding — inflammation raising fat density toward water.

| Feature | Unit | Description |
|---------|------|-------------|
| `colon_surr_fat_mean_hu` | HU | Mean fat HU in 5mm pericolonic shell |
| `colon_surr_fat_p90_hu` | HU | 90th percentile fat HU in shell |

---

### Bowel Wall HU (Submucosal Edema)

The bowel wall is isolated via **binary erosion**:

```
eroded = binary_erosion(bowel_mask, iterations=1)
wall = bowel_mask & ~eroded
wall_hu = clip(CT[wall], -50, 150)
```

Elevated wall HU indicates submucosal edema (e.g., ischemic colitis, Crohn's). Wall HU values are clipped to −50 to +150 to exclude air and calcium artifacts.

Computed for both **small bowel** (small_bowel + duodenum) and **colon**:

| Feature | Unit | Description |
|---------|------|-------------|
| `small_bowel_wall_mean_hu` | HU | Mean HU of small bowel wall |
| `small_bowel_wall_std_hu` | HU | Std of small bowel wall HU |
| `small_bowel_wall_p10_hu` | HU | 10th percentile |
| `small_bowel_wall_p90_hu` | HU | 90th percentile |
| `colon_wall_mean_hu` | HU | Mean HU of colon wall |
| `colon_wall_std_hu` | HU | Std of colon wall HU |
| `colon_wall_p10_hu` | HU | 10th percentile |
| `colon_wall_p90_hu` | HU | 90th percentile |

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `sb_max_diam_mm` | mm | Small bowel maximum diameter |
| `colon_max_diam_mm` | mm | Colon maximum diameter |
| `sb_to_colon_diam_ratio` | — | SBO indicator ratio |
| `hiatal_hernia_volume_cc` | mL | Herniated stomach volume |
| `hiatal_hernia_percent` | fraction | Fraction of stomach herniated |
| `stomach_total_volume_cc` | mL | Total stomach volume |
| `colon_surr_fat_mean_hu` | HU | Pericolonic fat mean HU |
| `colon_surr_fat_p90_hu` | HU | Pericolonic fat 90th percentile HU |
| `small_bowel_wall_mean_hu` | HU | Small bowel wall mean HU |
| `small_bowel_wall_std_hu` | HU | Small bowel wall HU std |
| `small_bowel_wall_p10_hu` | HU | Small bowel wall HU p10 |
| `small_bowel_wall_p90_hu` | HU | Small bowel wall HU p90 |
| `colon_wall_mean_hu` | HU | Colon wall mean HU |
| `colon_wall_std_hu` | HU | Colon wall HU std |
| `colon_wall_p10_hu` | HU | Colon wall HU p10 |
| `colon_wall_p90_hu` | HU | Colon wall HU p90 |
