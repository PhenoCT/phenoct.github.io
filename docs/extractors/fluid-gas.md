# Fluid & Gas Extractor

`FluidGasExtractor` quantifies pathological free fluid and gas accumulation in the body cavities. Features cover pneumoperitoneum, ascites, pleural effusion, and subcutaneous edema.

**Required masks:** `organ_seg`, `body`

---

## Orientation Note

!!! warning "Z axis is craniocaudal"
    Thorax/abdomen separation is performed along **axis 2 (Z)**, not axis 0.
    In RAS+: THORAX = high Z (toward head), ABDOMEN = low Z (toward feet).

---

## Methods

### Diaphragm Level Detection

The diaphragm Z-level is found using anatomical landmarks:

1. **Preferred:** Maximum Z of the liver = liver top = diaphragm level
2. **Fallback:** Minimum Z of the lungs = lung bottom = diaphragm level

```
z_diaphragm = max(liver_coords[Z])
```

This Z index is used to split the body into thoracic (Z ≥ z_diaphragm) and abdominal (Z < z_diaphragm) compartments.

---

### Cavity Mask

The background cavity (potential space for fluid and gas) is computed by subtracting known organ content from the body:

```
solid_or_bone = isin(seg, solid_organ_ids + bone_ids)
gas_org       = isin(seg, gas_organ_ids)   # bowel, lungs, trachea
cavity_mask   = body & ~(solid_or_bone | gas_org)
```

**Solid organs excluded:** spleen, kidneys, gallbladder, liver, stomach, pancreas, bladder, prostate, heart, aorta.

**Gas organs excluded (after 2-voxel dilation to prevent partial-volume leakage):** all lung lobes, esophagus, trachea, small bowel, duodenum, colon.

---

### Free Air (Pneumoperitoneum)

Free intraperitoneal air is defined as voxels with HU ≤ −600 outside gas-containing organs:

```
free_air = (CT ≤ -600) & body & ~gas_dilated & ~solid_or_bone
```

Gas organs are dilated by 2 voxels before exclusion to avoid partial-volume artifacts at organ boundaries.

| Feature | Unit | Description |
|---------|------|-------------|
| `free_air_volume_cc` | mL | Volume of free air (HU ≤ −600) |

---

### Pleural Effusion

Fluid in the **thoracic** compartment (Z ≥ z_diaphragm):

```
fluid_hu = (-10 ≤ CT < 30)
thorax_roi = cavity_mask & (Z ≥ z_diaphragm)
effusion_voxels = fluid_hu & thorax_roi
```

| Feature | Unit | Description |
|---------|------|-------------|
| `pleural_effusion_volume_cc` | mL | Volume of thoracic fluid |

---

### Ascites

Fluid in the **abdominal** compartment (Z < z_diaphragm), with additional filtering:

**Body-wall shell exclusion:** The body mask is eroded by ~15 mm (to exclude skin, subcutaneous fat, and muscle layers). The tighter mask prevents muscle and fascia from being misidentified as ascites.

**Candidate mask:**
```
eroded_body = binary_erosion(body, ~15mm)
body_wall_shell = body & ~eroded_body

ascites_candidate = eroded_body
    & ~solid_or_bone
    & ~gas_org
    & ~muscle_mask
    & ~body_wall_shell
    & (Z < z_diaphragm)

ascites_voxels = fluid_hu & ascites_candidate
```

**Connected-component filtering:** Components smaller than **5 mL** (5000 / voxel_vol mm³) are removed as noise. The largest remaining component is also reported separately.

| Feature | Unit | Description |
|---------|------|-------------|
| `ascites_volume_cc` | mL | Total ascites volume (components ≥ 5 mL) |
| `ascites_largest_component_cc` | mL | Volume of the single largest component |

---

### Global Fluid Ratio

```
global_fluid_ratio = (effusion_volume + ascites_volume) / total_body_volume
```

| Feature | Unit | Description |
|---------|------|-------------|
| `total_body_volume_cc` | mL | Total body voxel volume |
| `global_fluid_ratio` | fraction | Total fluid / body volume |

---

### Anasarca (Subcutaneous Edema)

Subcutaneous tissue is defined as a **10 mm ring** at the body surface:

```
subcut_roi = body & ~binary_erosion(body, ~10mm)
subcut_fat = subcut_roi & (-150 ≤ CT < 0)
subcutaneous_fat_mean_hu = mean(CT[subcut_fat])
```

Normal subcutaneous fat: −100 to −80 HU. Edematous fat is elevated toward water (0 HU) due to interstitial fluid infiltration.

| Feature | Unit | Description |
|---------|------|-------------|
| `subcutaneous_fat_mean_hu` | HU | Mean HU of subcutaneous fat ring |

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `total_body_volume_cc` | mL | Total body volume |
| `free_air_volume_cc` | mL | Free intraperitoneal air (HU ≤ −600) |
| `pleural_effusion_volume_cc` | mL | Thoracic free fluid |
| `ascites_volume_cc` | mL | Abdominal free fluid (components ≥ 5 mL) |
| `ascites_largest_component_cc` | mL | Largest single ascites component |
| `global_fluid_ratio` | fraction | (Effusion + Ascites) / body volume |
| `subcutaneous_fat_mean_hu` | HU | Mean HU of 10mm subcutaneous fat ring |
