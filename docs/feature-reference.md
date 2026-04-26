# Feature Reference

Complete listing of all Image-Derived Phenotypes organized by extractor. Use your browser's search (**Ctrl+F** / **Cmd+F**) or the site search bar to find specific features.

---

## Organ Morphology

Applies to all 117 TotalSegmentator organs. Shown here for key organs; pattern repeats for all organs present in the segmentation.

| Feature | Unit | Description |
|---------|------|-------------|
| `{organ}_volume_cc` | mL | Organ volume |
| `{organ}_hu_mean` | HU | Mean Hounsfield unit (clipped) |
| `{organ}_hu_std` | HU | HU standard deviation |
| `{organ}_hu_median` | HU | Median HU |
| `{organ}_hu_min` | HU | Minimum HU |
| `{organ}_hu_max` | HU | Maximum HU |
| `{organ}_hu_p10` | HU | 10th percentile HU |
| `{organ}_hu_p25` | HU | 25th percentile HU |
| `{organ}_hu_p75` | HU | 75th percentile HU |
| `{organ}_hu_p90` | HU | 90th percentile HU |
| `{organ}_hu_hist_0` | fraction | Fraction of voxels in −1000 to −500 HU |
| `{organ}_hu_hist_1` | fraction | Fraction in −500 to −50 HU |
| `{organ}_hu_hist_2` | fraction | Fraction in −50 to 0 HU |
| `{organ}_hu_hist_3` | fraction | Fraction in 0 to 50 HU |
| `{organ}_hu_hist_4` | fraction | Fraction in 50 to 100 HU |
| `{organ}_hu_hist_5` | fraction | Fraction in 100 to 200 HU |
| `{organ}_hu_hist_6` | fraction | Fraction in 200 to 500 HU |
| `{organ}_hu_hist_7` | fraction | Fraction in 500 to 1000 HU |
| `{organ}_bbox_x_mm` | mm | Bounding box Left-Right extent |
| `{organ}_bbox_y_mm` | mm | Bounding box Anterior-Posterior extent |
| `{organ}_bbox_z_mm` | mm | Bounding box craniocaudal extent |
| `{organ}_centroid_x_mm` | mm | Centroid X (Left-Right) |
| `{organ}_centroid_y_mm` | mm | Centroid Y (Anterior-Posterior) |
| `{organ}_centroid_z_mm` | mm | Centroid Z (craniocaudal) |
| `{organ}_entropy` | bits | Shannon entropy — liver, kidneys, spleen, pancreas only |
| `gallbladder_calc_fraction` | fraction | Gallbladder voxels with HU > 130 |
| `liver_spleen_hu_diff` | HU | Liver mean HU − spleen mean HU |
| `liver_density_cv` | — | Liver HU coefficient of variation |
| `liver_occupancy_extent` | fraction | Liver volume / bounding box volume |

---

## Cardiothoracic

| Feature | Unit | Description |
|---------|------|-------------|
| `total_lung_volume_cc` | mL | Total volume all 5 lung lobes |
| `total_atelectasis_volume_cc` | mL | Total atelectasis (HU −200 to +150) |
| `total_atelectasis_fraction` | fraction | Atelectasis / total lung volume |
| `{lobe}_atelectasis_volume_cc` | mL | Per-lobe atelectasis volume |
| `{lobe}_atelectasis_fraction` | fraction | Per-lobe atelectasis fraction |
| `{lobe}_atelectasis_mean_hu` | HU | Mean HU of atelectatic voxels |
| `mediastinum_soft_tissue_volume_cc` | mL | Mediastinal soft tissue volume |
| `mediastinum_soft_tissue_mean_hu` | HU | Mean HU mediastinal soft tissue |
| `mediastinum_soft_tissue_std_hu` | HU | Std HU mediastinal soft tissue |

Lobe names: `lung_upper_lobe_left`, `lung_lower_lobe_left`, `lung_upper_lobe_right`, `lung_middle_lobe_right`, `lung_lower_lobe_right`.

---

## Hepatobiliary

| Feature | Unit | Description |
|---------|------|-------------|
| `gallbladder_volume_cc` | mL | Gallbladder volume |
| `gallbladder_hu_mean` | HU | Mean HU |
| `gallbladder_sphericity` | 0–1 | Gallbladder shape sphericity |
| `gallbladder_wall_mean_hu` | HU | Wall HU mean |
| `gallbladder_wall_std_hu` | HU | Wall HU std |
| `pericholecystic_fat_stranding_fraction` | fraction | 10mm halo voxels at HU −50 to 0 |
| `pericholecystic_fat_mean_hu` | HU | Mean HU of pericholecystic fat |
| `cbd_diameter_mm` | mm | CBD diameter (95th percentile × 2) |
| `cbd_length_mm` | mm | CBD skeleton length |
| `cbd_dilated` | 0/1 | CBD > 6mm |
| `ihd_diameter_mm` | mm | Intrahepatic duct diameter |
| `ihd_volume_cc` | mL | IHD candidate volume |
| `liver_water_fraction` | fraction | Liver voxels at HU 0–25 |

---

## Vascular

| Feature | Unit | Description |
|---------|------|-------------|
| `{vessel}_volume_ml` | mL | Volume |
| `{vessel}_hu_mean` | HU | Mean HU |
| `{vessel}_hu_std` | HU | Std HU |
| `{vessel}_hu_median` | HU | Median HU |
| `{vessel}_hu_min` | HU | Min HU |
| `{vessel}_hu_max` | HU | Max HU |
| `{vessel}_hu_p10/p25/p75/p90` | HU | Percentiles |
| `{vessel}_hu_cv` | — | HU coefficient of variation |
| `{vessel}_hu_hist_0..7` | fraction | 8-bin HU histogram |
| `{vessel}_bbox_x/y/z_mm` | mm | Bounding box |
| `{vessel}_vessel_length_mm` | mm | Vessel length |
| `{vessel}_vessel_diam_mean` | mm | Mean diameter |
| `{vessel}_vessel_diam_std` | mm | Diameter std |
| `{vessel}_vessel_diam_p50` | mm | Median diameter |
| `{vessel}_vessel_diam_p75` | mm | 75th percentile diameter |
| `{vessel}_vessel_diam_p90` | mm | 90th percentile diameter |
| `{vessel}_vessel_diam_max` | mm | Maximum diameter |
| `{vessel}_vessel_diam_cv` | — | Diameter CV |
| `aorta_max_diam_location_z_mm` | mm | Z position of max aorta diameter |
| `{vessel}_hu_heterogeneity_std` | HU | Centerline HU std |
| `{vessel}_hu_heterogeneity_cv` | — | Centerline HU CV |
| `{vessel}_hu_heterogeneity_range` | HU | Centerline HU range |
| `{vessel}_hu_heterogeneity_entropy` | bits | Centerline HU entropy |
| `aorta_calc_volume_ml` | mL | Aortic calcification volume |
| `aorta_calc_fraction` | fraction | Aortic calcification fraction |
| `aorta_enhancement` | HU | Aortic HU above 50 baseline |
| `pv_aorta_hu_ratio` | — | Portal vein / aorta HU ratio |
| `ivc_aorta_hu_ratio` | — | IVC / aorta HU ratio |
| `vein_opac_gate` | 0–1 | Sigmoid venous opacification |

Vessel prefixes: `aorta`, `portal_vein_and_splenic_vein`, `inferior_vena_cava`.

---

## Bowel

| Feature | Unit | Description |
|---------|------|-------------|
| `sb_max_diam_mm` | mm | Small bowel max diameter |
| `colon_max_diam_mm` | mm | Colon max diameter |
| `sb_to_colon_diam_ratio` | — | SBO ratio |
| `hiatal_hernia_volume_cc` | mL | Herniated stomach volume |
| `hiatal_hernia_percent` | fraction | Fraction of stomach herniated |
| `stomach_total_volume_cc` | mL | Total stomach volume |
| `colon_surr_fat_mean_hu` | HU | Pericolonic fat mean HU |
| `colon_surr_fat_p90_hu` | HU | Pericolonic fat p90 HU |
| `small_bowel_wall_mean_hu` | HU | SB wall mean HU |
| `small_bowel_wall_std_hu` | HU | SB wall HU std |
| `small_bowel_wall_p10_hu` | HU | SB wall HU p10 |
| `small_bowel_wall_p90_hu` | HU | SB wall HU p90 |
| `colon_wall_mean_hu` | HU | Colon wall mean HU |
| `colon_wall_std_hu` | HU | Colon wall HU std |
| `colon_wall_p10_hu` | HU | Colon wall HU p10 |
| `colon_wall_p90_hu` | HU | Colon wall HU p90 |

---

## Fluid & Gas

| Feature | Unit | Description |
|---------|------|-------------|
| `total_body_volume_cc` | mL | Total body volume |
| `free_air_volume_cc` | mL | Free air (HU ≤ −600) |
| `pleural_effusion_volume_cc` | mL | Thoracic free fluid |
| `ascites_volume_cc` | mL | Abdominal free fluid |
| `ascites_largest_component_cc` | mL | Largest ascites component |
| `global_fluid_ratio` | fraction | Total fluid / body volume |
| `subcutaneous_fat_mean_hu` | HU | Subcutaneous fat mean HU |

---

## Bone

| Feature | Unit | Description |
|---------|------|-------------|
| `{vertebra}_mean_hu` | HU | Trabecular HU mean |
| `{vertebra}_std_hu` | HU | Trabecular HU std |
| `{vertebra}_height_mm` | mm | Vertebral height (Z extent) |
| `{vertebra}_compression_ratio` | — | Height / avg neighbor height |
| `lumbar_hu_mean` | HU | L1–L4 pooled trabecular HU mean |
| `lumbar_hu_std` | HU | L1–L4 pooled trabecular HU std |
| `lumbar_hu_p10` | HU | L1–L4 10th percentile |
| `lumbar_hu_p25` | HU | L1–L4 25th percentile |
| `lumbar_hu_p50` | HU | L1–L4 median |
| `lumbar_hu_p75` | HU | L1–L4 75th percentile |
| `lumbar_hu_p90` | HU | L1–L4 90th percentile |
| `min_compression_ratio` | — | Minimum across spine |
| `mean_compression_ratio` | — | Mean across spine |
| `std_compression_ratio` | — | Std across spine |

Vertebra names: `vertebrae_T1`–`vertebrae_T12`, `vertebrae_L1`–`vertebrae_L5`.

---

## Kidney

| Feature | Unit | Description |
|---------|------|-------------|
| `kidney_{side}_fluid_fraction` | fraction | HU 0–20 fraction |
| `kidney_{side}_low_hu_fraction` | fraction | HU < 20 fraction |
| `kidney_{side}_high_hu_fraction` | fraction | HU > 100 fraction |
| `kidney_{side}_low_hu_density` | fraction | HU −10 to 20 (water tail) |
| `kidney_{side}_sphericity` | 0–1 | Shape sphericity |
| `cyst_{side}_volume_cc` | mL | Cyst volume |
| `kidney_{side}_cyst_burden` | fraction | Cyst / total volume |
| `kidney_volume_asymmetry` | 0–1 | Bilateral volume asymmetry |
| `kidney_fluid_fraction_asymmetry` | 0–1 | Bilateral fluid asymmetry |
| `kidney_low_hu_density_asymmetry` | 0–1 | Bilateral water density asymmetry |
| `kidney_sphericity_asymmetry` | 0–1 | Bilateral sphericity asymmetry |
| `kidney_p10_hu_diff` | HU | |p10_left − p10_right| |
| `kidney_p5_hu_diff` | HU | |p5_left − p5_right| |

Side: `left`, `right`.

---

## Appendix / RLQ

| Feature | Unit | Description |
|---------|------|-------------|
| `appendix_loc_confidence` | 0–1 | Localization confidence |
| `appendix_loc_method` | string | Localization method used |
| `center_x/y/z_vox` | voxels | RLQ center (voxels) |
| `center_x/y/z_mm` | mm | RLQ center (mm) |
| `box_size_x/y/z_mm` | mm | VOI box size |
| `rlq_fat_mean_hu` | HU | RLQ fat mean HU |
| `rlq_fat_std_hu` | HU | RLQ fat HU std |
| `rlq_fat_p10_hu` | HU | RLQ fat HU p10 |
| `rlq_fat_p90_hu` | HU | RLQ fat HU p90 |
| `rlq_stranding_volume_cc` | mL | Fat stranding volume (HU > −70) |
| `rlq_stranding_fraction` | fraction | Stranding fraction |
| `rlq_to_subcut_fat_ratio` | HU | RLQ fat HU minus subcutaneous fat HU |
| `rlq_fluid_volume_cc` | mL | Free fluid in RLQ |
| `rlq_calcification_volume_cc` | mL | Appendicolith volume |
| `rlq_calcification_max_hu` | HU | Max HU of calcification |
| `rlq_bowel_wall_mean_hu` | HU | Bowel wall mean HU in RLQ |
| `rlq_bowel_wall_std_hu` | HU | Bowel wall HU std in RLQ |
