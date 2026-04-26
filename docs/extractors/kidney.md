# Kidney Extractor

`KidneyExtractor` computes features targeting renal pathology: simple cysts, hydronephrosis, stones, and bilateral asymmetry. It processes left and right kidneys independently and computes asymmetry indices that are key for unilateral pathology detection.

**Required masks:** `organ_seg`, *(optional)* `cyst`

---

## Methods

### Fluid Fraction

Fluid voxels within the kidney are those with HU in the range 0–20 (simple cyst density, hydronephrosis pelvic fluid):

```
fluid_mask = (0 ≤ kidney_voxels ≤ 20)
fluid_fraction = count(fluid_mask) / total_kidney_voxels
```

| Feature | Unit | Description |
|---------|------|-------------|
| `kidney_{side}_fluid_fraction` | fraction | Fraction of kidney at HU 0–20 |

---

### Low HU Density (Water Tail)

A more sensitive version of fluid fraction, using the range −10 to +20 HU to capture all pure-water density regions:

```
water_mask = (-10 ≤ kidney_voxels ≤ 20)
low_hu_density = count(water_mask) / total_kidney_voxels
```

| Feature | Unit | Description |
|---------|------|-------------|
| `kidney_{side}_low_hu_density` | fraction | Fraction of kidney at HU −10 to 20 |

---

### Low and High HU Fractions

| Feature | Threshold | Clinical interpretation |
|---------|-----------|------------------------|
| `kidney_{side}_low_hu_fraction` | HU < 20 | Fluid, cysts |
| `kidney_{side}_high_hu_fraction` | HU > 100 | Calcifications, stones |

---

### Sphericity

Healthy kidneys are bean-shaped (ellipsoidal). Hydronephrotic kidneys become more spherical due to pelvic pressure.

$$
\text{sphericity} = \frac{\pi^{1/3} \cdot (6V)^{2/3}}{A}
$$

where:
- V = volume in mm³ (`count(mask) × ∏(spacing)`)
- A = surface area in mm² estimated as:

```
eroded = binary_erosion(mask)
surface = mask XOR eroded
A = count(surface) × spacing_x × spacing_y
```

A value of 1.0 = perfect sphere. Normal kidneys: ~0.6–0.7. Hydronephrotic: ~0.8–1.0.

| Feature | Unit | Description |
|---------|------|-------------|
| `kidney_{side}_sphericity` | 0–1 | Shape sphericity (1 = sphere) |

---

### Cyst Analysis (Optional)

Requires a separate cyst segmentation mask (`cyst` key in masks dict) with:
- Label 1 = left kidney cysts
- Label 2 = right kidney cysts

```
cyst_volume_cc = count(cyst_mask == label) × voxel_vol_cc
total_volume = kidney_vol + cyst_vol
cyst_burden = cyst_vol / total_volume
```

| Feature | Unit | Description |
|---------|------|-------------|
| `cyst_{side}_volume_cc` | mL | Cyst volume per kidney |
| `kidney_{side}_cyst_burden` | fraction | Cyst / (kidney + cyst) volume |

---

### Bilateral Asymmetry Indices

Computed only when both kidneys are present. Each asymmetry index uses the normalized absolute difference:

$$
\text{asymmetry} = \frac{|L - R|}{L + R}
$$

Applied to:

| Feature | Variable compared |
|---------|-----------------|
| `kidney_volume_asymmetry` | Total kidney volume |
| `kidney_fluid_fraction_asymmetry` | Fluid fraction |
| `kidney_low_hu_density_asymmetry` | Low HU density |
| `kidney_sphericity_asymmetry` | Sphericity |

**HU percentile differences** (absolute, not normalized):

| Feature | Description |
|---------|-------------|
| `kidney_p10_hu_diff` | `|left_p10 − right_p10|` |
| `kidney_p5_hu_diff` | `|left_p5 − right_p5|` |

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `kidney_{side}_fluid_fraction` | fraction | HU 0–20 fraction |
| `kidney_{side}_low_hu_fraction` | fraction | HU < 20 fraction |
| `kidney_{side}_high_hu_fraction` | fraction | HU > 100 fraction |
| `kidney_{side}_low_hu_density` | fraction | HU −10 to 20 fraction (water tail) |
| `kidney_{side}_sphericity` | 0–1 | Morphological sphericity |
| `cyst_{side}_volume_cc` | mL | Cyst volume (if cyst mask provided) |
| `kidney_{side}_cyst_burden` | fraction | Cyst burden fraction |
| `kidney_volume_asymmetry` | 0–1 | Bilateral volume asymmetry |
| `kidney_fluid_fraction_asymmetry` | 0–1 | Bilateral fluid asymmetry |
| `kidney_low_hu_density_asymmetry` | 0–1 | Bilateral water-density asymmetry |
| `kidney_sphericity_asymmetry` | 0–1 | Bilateral sphericity asymmetry |
| `kidney_p10_hu_diff` | HU | Absolute p10 HU difference L vs R |
| `kidney_p5_hu_diff` | HU | Absolute p5 HU difference L vs R |

Side values: `left`, `right`.
