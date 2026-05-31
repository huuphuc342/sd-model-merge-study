# CUAMIX — Iterative Model Merge Optimization for Stable Diffusion

> A systematic study of checkpoint interpolation methods and merge ratio optimization  
> to maximize anatomical fidelity in photorealistic Stable Diffusion outputs.

---

## Overview

This repository documents the full development process of the **CUAMIX model series** (V2 through V6.1, and UnrealWorld v1–v3) — a multi-stage iterative merge experiment designed to identify optimal weight interpolation ratios across multiple Stable Diffusion checkpoints.

The goal was not simply to produce aesthetically pleasing images, but to **systematically evaluate how different merge strategies affect model behavior** in anatomically complex regions — specifically eyes, hands, and fingers — which represent the highest-frequency detail areas and are most sensitive to weight interference between merged checkpoints.

---

## Motivation

Off-the-shelf Stable Diffusion checkpoints frequently exhibit characteristic failure modes:
- **Hands and fingers**: incorrect digit count, anatomical distortion
- **Eyes**: asymmetry, iris artifacts, loss of specular highlight
- **Skin contrast**: loss of micro-detail at high CFG scales

Existing public merges addressed some of these issues individually. This project investigates whether **iterative ratio-tuned merging** across carefully selected base models can compound improvements across all three failure domains simultaneously.

---

## Methodology

### Controlled Evaluation Protocol

To ensure fair comparison across versions, all evaluations followed a strict controlled protocol:

- **Fixed seed** across all versions — same seed used from V2 through UnrealWorld v2
- **Fixed prompt set** — same positive and negative prompts across all evaluations
- **Fixed sampler settings** — steps, CFG scale, and resolution held constant
- **Pixel-level manual assessment** focused on three anatomical regions:
  - `eyes` — symmetry, iris clarity, specular highlight retention
  - `hands` — digit count accuracy, joint definition
  - `fingers` — tip sharpness, nail detail, separation clarity
- **Contrast and micro-detail** evaluated at 100% zoom on identical crop regions

### Merge Methods Tested

| Method | Description | Used In |
|---|---|---|
| Weighted Sum | Linear interpolation between two checkpoints at ratio α | V2–V6.1 |
| Add Difference | Adds the difference of two models onto a primary base | UnrealWorld v2, v3 |

### Version Tree

```
duchaitenaiart_V453  (base)
    └── CUAMIX_V2   (+ [undocumented], Weighted Sum 0.15)
        └── CUAMIX_V3   (+ majicmixRealistic_v6, Weighted Sum 0.30)
            └── CUAMIX_V4.1  (+ [undocumented], Weighted Sum 0.50)
                └── CUAMIX_V5    (+ [undocumented], Weighted Sum 0.40)
                    └── CUAMIX_V6    (+ picxReal_10, Weighted Sum 0.30)
                        └── CUAMIX_V6.1 = UnrealWorld_v1  ← public rebrand
                                         (+ picxReal_10, Weighted Sum 0.73)
                            └── UnrealWorld_v2  (+ cyberrealistic_v70, Add Difference 0.20)
                                └── UnrealWorld_v3  (+ cyberrealistic_v70, Add Difference 0.20)
```

> **Note on naming:** The internal development series (CUAMIX) was rebranded to **UnrealWorld** at the V6.1 stage upon first public release. All subsequent public versions follow the UnrealWorld naming convention.

> **Note on metadata completeness:** Secondary model references for CUAMIX V2, V4.1, and V5 are partially unresolvable — source files were deleted after merging as part of storage management. Model hashes are preserved in `merge_recipe.json`; names are not recoverable for those versions.

Full merge recipe with model hashes available in [`merge_recipe.json`](./merge_recipe.json)

---

## Results

### Version Comparison

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Side-by-side grid, same seed, V2 / V6 / UnrealWorld_v1 / UnrealWorld_v3 -->
<!-- Recommended: 4-column comparison at 512x768 or 768x1024 -->
<!-- Label each column clearly with version name -->
```
[ INSERT: 4-column visual comparison — V2 vs V6 vs UnrealWorld_v1 vs UnrealWorld_v3 ]
```

---

### Eye Region — Pixel-Level Detail

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Cropped eye region, 100% zoom, same position across versions -->
<!-- Show: V3, V5, V6.1, UnrealWorld_v2 — 4 crops side by side -->
```
[ INSERT: Eye region crop comparison across versions ]
```

**Observation:** Higher weighted sum ratios (0.73 at V6.1) retained specular highlight and iris boundary definition significantly better than lower ratios. Add Difference at 0.20 (UnrealWorld v2) preserved these gains without style regression.

---

### Hand & Finger Anatomy

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Hand/finger crop, 100% zoom, same prompt and seed -->
<!-- Show: V2 (baseline failure), V6 (improvement), UnrealWorld_v2 (final) -->
```
[ INSERT: Hand/finger crop comparison — V2 baseline vs V6.1 vs UnrealWorld_v2 ]
```

**Observation:** Early versions (V2–V3) showed characteristic finger separation artifacts. Progressive ratio tuning reduced these artifacts. Addition of picxReal_10 at V6 stage contributed most significantly to digit count accuracy.

---

### Contrast & Micro-Detail

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Skin/fabric detail crop at 100% zoom -->
<!-- Show: V5 vs V6.1 vs UnrealWorld_v2 -->
```
[ INSERT: Micro-detail crop comparison — V5 vs V6.1 vs UnrealWorld_v2 ]
```

**Observation:** Add Difference injection of cyberrealistic_v70 at 0.20 multiplier (UnrealWorld v2) improved contrast response at high CFG without introducing the oversaturation artifacts typical of direct cyberrealistic merges at higher ratios.

---

## Key Findings

| Finding | Detail |
|---|---|
| Optimal Weighted Sum ratio | **0.73** (V6.1) — best balance of detail retention vs style coherence |
| Add Difference optimal multiplier | **0.20** — effective for style injection without base model degradation |
| Most impactful single merge step | Addition of `picxReal_10` at V6 stage — largest gain in anatomical accuracy |
| Diminishing returns threshold | Ratios above 0.75 introduced color cast artifacts in skin tones |
| Fixed seed validation | Confirmed that observed improvements were model-driven, not prompt/seed variation |

---

## Base Models Referenced

| Model | Role |
|---|---|
| duchaitenaiart_V453 | Initial base (V2) |
| cyberrealistic_v33 / v70 | Style reference, contrast injection |
| majicmixRealistic_v6 | Realism foundation |
| epicphotogasm_x / lastUnicorn | Detail sharpness reference |
| picxReal_10 | Anatomical accuracy contributor |
| realisticVisionV60B1 | Secondary realism reference |

---

## Repository Structure

```
.
├── README.md               ← This document
├── merge_recipe.json       ← Full merge tree with model hashes
└── comparisons/
    ├── full_comparison/    ← ADD: 4-column version comparison images
    ├── eye_crops/          ← ADD: Eye region pixel-level crops
    ├── hand_crops/         ← ADD: Hand/finger anatomy crops
    └── detail_crops/       ← ADD: Micro-detail and contrast crops
```

---

## Public Release

The final model (UnrealWorld v2) was published on CivitAI and reached **7,100+ organic downloads** from users globally, providing large-scale real-world validation of the merge optimization outcomes documented here.

→ [View on CivitAI](https://civitai.com/user/NxtAI)

---

## Author

**Nguyen Huu Phuc**  
B.Sc. Computer Science — Ton Duc Thang University  
[GitHub](https://github.com/huuphuc342) · [CivitAI](https://civitai.com/user/NxtAI) · [LinkedIn](https://linkedin.com/in/phucnguyenhuu-it/)
