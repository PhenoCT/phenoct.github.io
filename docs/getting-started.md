# Getting Started

## Installation

```bash
git clone https://github.com/PhenoCT/PhenoCT.git
cd PhenoCT
pip install -e .
```

**Requirements:** Python ≥ 3.8, numpy, scipy, scikit-image, nibabel, pandas.

---

## Input Requirements

Each case requires:

| Input | Description |
|-------|-------------|
| `image.nii.gz` | CT scan in HU values, any spacing |
| `organ_seg.nii.gz` | TotalSegmentator v2 output (117-label segmentation) |
| `body.nii.gz` | Body mask (0/1 or 0/255) — required by Fluid/Gas and Appendix extractors |
| `cyst.nii.gz` | *(Optional)* Kidney cyst mask with labels 1=left, 2=right |

!!! note "Orientation"
    The package loads all files with `nibabel.as_closest_canonical()`, which reorients to **RAS+**. You do not need to pre-orient your files.

---

## Single Case

```python
from phenoct.batch import process_single_case
from phenoct.extractors import (
    OrganMorphologyExtractor,
    CardiothoracicExtractor,
    HepatobiliaryExtractor,
    VascularExtractor,
    BowelExtractor,
    FluidGasExtractor,
    BoneExtractor,
    KidneyExtractor,
    AppendixExtractor,
)

extractors = [
    OrganMorphologyExtractor(),
    CardiothoracicExtractor(),
    HepatobiliaryExtractor(),
    VascularExtractor(fast_mode=True),  # (1)!
    BowelExtractor(fast_mode=True),
    FluidGasExtractor(),
    BoneExtractor(fast_mode=True),
    KidneyExtractor(),
    AppendixExtractor(),
]

features = process_single_case(
    case_id   = "patient_001",
    image_path = "/data/ct.nii.gz",
    mask_paths = {
        "organ_seg": "/data/organ_seg.nii.gz",
        "body":      "/data/body.nii.gz",
    },
    extractors = extractors,
)
# features is a flat dict: {"liver_volume_cc": 1423.0, "aorta_vessel_diam_mean": 22.1, ...}
```

1. `fast_mode=True` uses slice-wise 2D EDT instead of full 3D skeletonization — 50–100× faster with minimal accuracy loss. Recommended for batch processing.

---

## Batch Processing

```python
import pandas as pd
from phenoct.batch import process_batch

cases = [
    {
        "case_id":   "patient_001",
        "image":     "/data/001/ct.nii.gz",
        "organ_seg": "/data/001/seg.nii.gz",
        "body":      "/data/001/body.nii.gz",
    },
    # ... more cases
]

df = process_batch(cases, extractors=extractors, n_workers=8)
df.to_csv("features.csv", index=False)
```

---

## Output

The result is a **flat dictionary** (or DataFrame row) with one value per feature. Feature names follow the pattern:

```
{organ}_{measurement}_{unit_suffix}
```

For example:

| Feature | Value | Unit |
|---------|-------|------|
| `liver_volume_cc` | 1423.5 | mL |
| `aorta_vessel_diam_mean` | 22.1 | mm |
| `lumbar_hu_mean` | 142.3 | HU |
| `ascites_volume_cc` | 0.0 | mL |
| `kidney_left_fluid_fraction` | 0.04 | fraction |
