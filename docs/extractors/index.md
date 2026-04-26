# Extractors Overview

## Architecture

All extractors inherit from `BaseFeatureExtractor` and implement two methods:

```python
class BaseFeatureExtractor(ABC):
    def get_required_organs(self) -> List[str]: ...
    def extract(
        self,
        image_data: np.ndarray,          # CT in HU, RAS+, shape (X, Y, Z)
        masks: Dict[str, np.ndarray],    # {"organ_seg": ..., "body": ..., ...}
        spacing: Tuple[float, float, float],  # voxel spacing in mm
        mapping: Dict[int, str],         # {label_id: organ_name}
    ) -> Dict[str, float]: ...
```

## Orientation Convention

All extractors assume **RAS+ orientation** (nibabel `as_closest_canonical()`):

| Axis | Direction | Clinical meaning |
|------|-----------|-----------------|
| 0 (X) | Left → Right | Mediolateral |
| 1 (Y) | Posterior → Anterior | Anteroposterior |
| 2 (Z) | Inferior → Superior | Craniocaudal |

!!! warning "Z axis = craniocaudal"
    Diaphragm separation, vertebral heights, hiatal hernia, and all superior/inferior comparisons are computed along **axis 2**, not axis 0.

## Segmentation Labels

The package works with [TotalSegmentator v2](https://github.com/wasserth/TotalSegmentator) which provides 117 organ labels. The `mapping` dict maps integer label IDs to organ name strings, e.g. `{1: "spleen", 2: "kidney_right", ...}`.

## Utility Methods (Available to all extractors)

| Method | Description |
|--------|-------------|
| `_get_organ_mask(seg, name, mapping)` | Returns boolean mask for one organ |
| `_get_multi_organ_mask(seg, names, mapping)` | Returns combined boolean mask for organ list |
| `_compute_volume_cc(mask, spacing)` | Volume in mL = voxels × ∏(spacing) / 1000 |
| `_compute_mean_hu(image, mask)` | Mean HU within mask |
| `_check_required_mask(masks, name)` | Raises `ValueError` if mask missing |

## Extractor Summary

| Extractor | `fast_mode` | Required masks | Key dependencies |
|-----------|-------------|----------------|-----------------|
| OrganMorphology | — | `organ_seg` | scipy.ndimage |
| Cardiothoracic | — | `organ_seg` | scipy.ndimage |
| Hepatobiliary | — | `organ_seg` | scipy, skimage |
| Vascular | Yes | `organ_seg` | scipy, skimage |
| Bowel | Yes | `organ_seg` | scipy.ndimage |
| FluidGas | — | `organ_seg`, `body` | scipy.ndimage |
| Bone | Yes | `organ_seg` | scipy.ndimage |
| Kidney | — | `organ_seg` | scipy.ndimage |
| Appendix | — | `organ_seg`, `body` | scipy.ndimage |
