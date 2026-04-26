# Cardiothoracic Extractor

`CardiothoracicExtractor` quantifies pulmonary and mediastinal pathology from the thoracic CT. It focuses on lung parenchyma atelectasis, mediastinal soft tissue, and related thoracic measurements.

**Required masks:** `organ_seg`

---

## Methods

### Atelectasis (Per Lobe)

Atelectasis appears as dependent lung consolidation with HU between −200 and +150. For each lung lobe individually:

1. Extract voxels within the lobe mask
2. Apply HU range: −200 ≤ HU ≤ +150
3. The atelectatic sub-region is defined by this threshold

```
atelectasis_voxels = lobe_mask & (HU >= -200) & (HU <= 150)
```

**Per lobe features** (prefix = lobe name, e.g. `lung_lower_lobe_left`):

| Feature | Unit | Description |
|---------|------|-------------|
| `{lobe}_atelectasis_volume_cc` | mL | Volume of atelectatic voxels |
| `{lobe}_atelectasis_fraction` | fraction | Atelectatic / total lobe voxels |
| `{lobe}_atelectasis_mean_hu` | HU | Mean HU of atelectatic voxels |

Lobes processed: `lung_upper_lobe_left`, `lung_lower_lobe_left`, `lung_upper_lobe_right`, `lung_middle_lobe_right`, `lung_lower_lobe_right`.

---

### Mediastinum ROI

The mediastinum is defined as the thoracic region excluding lungs, major vessels, and solid organs. Construction:

```
mediastinum = body_mask & ~major_organs & ~lung_mask
```

Where `major_organs` includes heart, aorta, esophagus, trachea, and pulmonary vessels.

**Mediastinal soft tissue** is further filtered to HU −50 to +150 (excludes fat and bone):

| Feature | Unit | Description |
|---------|------|-------------|
| `mediastinum_soft_tissue_volume_cc` | mL | Volume of soft tissue in mediastinum |
| `mediastinum_soft_tissue_mean_hu` | HU | Mean HU of mediastinal soft tissue |
| `mediastinum_soft_tissue_std_hu` | HU | Std of mediastinal soft tissue HU |

---

### Total Lung Volume

Both lungs combined:

| Feature | Unit | Description |
|---------|------|-------------|
| `total_lung_volume_cc` | mL | Sum of all 5 lobe volumes |
| `total_atelectasis_volume_cc` | mL | Total atelectatic volume across all lobes |
| `total_atelectasis_fraction` | fraction | Total atelectatic / total lung voxels |

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `total_lung_volume_cc` | mL | Total lung volume (all 5 lobes) |
| `total_atelectasis_volume_cc` | mL | Total atelectasis volume |
| `total_atelectasis_fraction` | fraction | Atelectasis fraction of total lung |
| `{lobe}_atelectasis_volume_cc` | mL | Lobe-specific atelectasis volume |
| `{lobe}_atelectasis_fraction` | fraction | Lobe-specific atelectasis fraction |
| `{lobe}_atelectasis_mean_hu` | HU | Mean HU in atelectatic region |
| `mediastinum_soft_tissue_volume_cc` | mL | Mediastinal soft tissue volume |
| `mediastinum_soft_tissue_mean_hu` | HU | Mean HU mediastinal soft tissue |
| `mediastinum_soft_tissue_std_hu` | HU | Std HU mediastinal soft tissue |
