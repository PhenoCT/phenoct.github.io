# Hepatobiliary Extractor

`HepatobiliaryExtractor` computes features related to the liver, gallbladder, bile ducts, and pericholecystic fat. It includes specialized algorithms for tracing the common bile duct (CBD) and detecting intrahepatic biliary dilation (IHD).

**Required masks:** `organ_seg`

---

## Methods

### Gallbladder Morphology & Wall

Standard volume and HU statistics are computed for the gallbladder. Additionally, the gallbladder **wall** is isolated by binary erosion:

```
wall = gallbladder_mask XOR binary_erosion(gallbladder_mask)
wall_hu = CT[wall]
```

**Sphericity** of the gallbladder (1.0 = perfect sphere):

$$
\text{sphericity} = \frac{\pi^{1/3} \cdot (6V)^{2/3}}{A}
$$

where V is volume in mm³ and A is the surface area estimated as:

```
A = count(surface_voxels) × spacing_x × spacing_y  (mm²)
```

---

### Pericholecystic Fat Stranding

A **10 mm dilation halo** is computed around the gallbladder mask using binary dilation. Fat stranding is detected as voxels in the halo region with HU between −50 and 0:

```
halo = binary_dilation(gb_mask, 10mm) & ~gb_mask & ~other_organs
stranding_voxels = halo & (-50 ≤ HU ≤ 0)
stranding_fraction = count(stranding_voxels) / count(halo_voxels)
```

| Feature | Unit | Description |
|---------|------|-------------|
| `pericholecystic_fat_stranding_fraction` | fraction | Fraction of 10mm halo in HU −50 to 0 |
| `pericholecystic_fat_mean_hu` | HU | Mean HU of stranding fat |

---

### Common Bile Duct (CBD) Tracing

The CBD is traced automatically between two anatomical anchor regions:

**1. Porta Hepatis (proximal anchor)**
- Defined as the **inferior 10–40% of the liver in Z** (superior axis) restricted to a low-Z slice range
- Right half of the liver in X (medial-to-right)

**2. Pancreas Head (distal anchor)**
- Defined as the **rightmost 25% of the pancreas in X**

**CBD candidate voxels:** HU between −30 and +120 within a dilated bounding box connecting the two anchors.

**Tracing algorithm:**
1. Label connected components in the HU candidate mask
2. Select the largest component spanning from porta hepatis to pancreas head region
3. Skeletonize the component using `skimage.morphology.skeletonize`
4. Apply `distance_transform_edt` to get radius at each skeleton point
5. CBD diameter = 2 × 95th percentile radius

**Normal CBD diameter limits:**
- Young patients: 6 mm
- Elderly patients: 8 mm
- Post-cholecystectomy: 10 mm

| Feature | Unit | Description |
|---------|------|-------------|
| `cbd_diameter_mm` | mm | 95th percentile diameter along CBD skeleton |
| `cbd_length_mm` | mm | Skeleton voxel count × mean spacing |
| `cbd_dilated` | bool | 1 if diameter > 6 mm |

---

### Intrahepatic Biliary Dilation (IHD)

IHD is detected in the **upper half of the liver** (high Z values, toward the head) — where intrahepatic ducts are located.

**Candidate voxels:** HU between −10 and +30 within the upper liver mask.

**EDT measurement:** For each candidate voxel on the skeleton, diameter = 2 × EDT radius. The IHD diameter is reported as the **95th percentile × 2** (robust to noise).

| Feature | Unit | Description |
|---------|------|-------------|
| `ihd_diameter_mm` | mm | 95th percentile IHD diameter |
| `ihd_volume_cc` | mL | Volume of IHD candidate voxels |

---

### Liver Water Fraction

Identifies voxels with HU 0–25 within the liver, excluding the portal vein region. This captures bile and fluid collections (e.g., hepatic cysts, bile lakes):

```
water_fraction = count(liver_voxels where 0 ≤ HU ≤ 25 & not_portal_vein) / total_liver_voxels
```

| Feature | Unit | Description |
|---------|------|-------------|
| `liver_water_fraction` | fraction | Fraction of liver in HU 0–25 |

---

## Full Feature Table

| Feature | Unit | Description |
|---------|------|-------------|
| `gallbladder_volume_cc` | mL | Gallbladder volume |
| `gallbladder_hu_mean` | HU | Mean HU within gallbladder |
| `gallbladder_sphericity` | 0–1 | Shape sphericity (1 = sphere) |
| `gallbladder_wall_mean_hu` | HU | Mean HU of gallbladder wall |
| `gallbladder_wall_std_hu` | HU | Std of wall HU |
| `pericholecystic_fat_stranding_fraction` | fraction | Fraction of 10mm halo in HU −50 to 0 |
| `pericholecystic_fat_mean_hu` | HU | Mean HU of pericholecystic fat |
| `cbd_diameter_mm` | mm | CBD diameter (95th percentile × 2) |
| `cbd_length_mm` | mm | Length of traced CBD skeleton |
| `cbd_dilated` | 0/1 | CBD > 6mm threshold |
| `ihd_diameter_mm` | mm | Intrahepatic duct diameter |
| `ihd_volume_cc` | mL | Volume of IHD candidate region |
| `liver_water_fraction` | fraction | Liver voxels at HU 0–25 |
