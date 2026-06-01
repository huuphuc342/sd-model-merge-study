# CUAMIX — Iterative Model Merge Optimization for Stable Diffusion

> A systematic empirical study of weight interpolation methods and merge ratio optimization,  
> documenting how different merge strategies affect property preservation in photorealistic Stable Diffusion outputs.

---

## Overview

This repository documents the full development process of the **CUAMIX model series** (V2 through V6.1, and UnrealWorld v1–v3) — a multi-stage iterative merge experiment designed to identify optimal weight interpolation ratios across multiple Stable Diffusion checkpoints.

The central question driving this work:

> **When merging two model checkpoints at ratio λ, which visual properties are preserved, which are degraded — and how does the choice of interpolation method affect this trade-off?**

This question mirrors a broader problem in machine learning: weight-space interpolation as a tool for property preservation. Recent work in LLM safety (e.g., Farn et al., EMNLP 2025) has demonstrated that linear interpolation between aligned and fine-tuned models — `θmerged = (1−λ)θbase + λθft` — can preserve alignment properties without additional data. This study applies the same interpolation principle in a generative vision context, systematically evaluating how ratio selection affects perceptual quality preservation across 7 iterative versions.

---

## Motivation

Off-the-shelf Stable Diffusion checkpoints frequently exhibit characteristic failure modes:
- **Facial realism**: loss of micro-detail, flat skin rendering, unnatural depth-of-field
- **Contrast & depth**: low dynamic range, absent depth cues, shallow DoF rendering
- **Hands and fingers**: incorrect digit count, anatomical distortion
- **Eyes**: asymmetry, iris artifacts, loss of specular highlight

Existing public merges addressed some of these issues individually. This project investigates whether **iterative ratio-tuned merging** across carefully selected base models can compound improvements across all failure domains simultaneously — and whether the Add Difference method produces qualitatively distinct improvements compared to Weighted Sum alone.

---

## Methodology

### Controlled Evaluation Protocol

To ensure fair comparison across versions, all evaluations followed a strict controlled protocol:

- **Fixed seed** across all versions — same seed used from base through UnrealWorld v3
- **Fixed prompt set** — same positive and negative prompts across all evaluations
- **Fixed sampler settings** — steps, CFG scale, and resolution held constant
- **4 reference checkpoints** evaluated per comparison: `duchaitenaiart_V453` (base) → `CUAMIX_V5` → `UnrealWorld_v1` (=CUAMIX_V6.1) → `UnrealWorld_v3`
- **Pixel-level manual assessment** focused on:
  - `facial realism` — skin micro-detail, pore texture, subsurface scattering appearance
  - `depth of field` — background separation, bokeh quality, focal plane clarity
  - `contrast` — dynamic range, shadow detail, highlight retention
  - `eyes` — symmetry, iris clarity, specular highlight retention
  - `hands/fingers` — digit count accuracy, joint definition, tip sharpness

### Merge Methods Tested

| Method | Formula | Used In |
|---|---|---|
| Weighted Sum | `result = A × (1 − α) + B × α` | CUAMIX V2–V6.1, UnrealWorld v1 |
| Add Difference | `result = A + (B − C) × multiplier` | UnrealWorld v2, v3 |

> **On the relationship between methods:** Weighted Sum linearly interpolates between two checkpoints — equivalent to the linear merging formulation in weight-space interpolation literature. Add Difference injects a *directional delta* (the difference between two models) onto a base — analogous to task vector arithmetic. The two methods produce qualitatively distinct outcomes, as documented in the Results section.

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

> **Note on naming:** The internal development series (CUAMIX) was rebranded to **UnrealWorld** at the V6.1 stage upon first public release.

> **Note on metadata completeness:** Secondary model references for CUAMIX V2, V4.1, and V5 are partially unresolvable — source files were deleted after merging as part of storage management. Model hashes are preserved in `merge_recipe.json`; names are not recoverable for those versions.

Full merge recipe with model hashes: [`merge_recipe.json`](./merge_recipe.json)

---

## Results

All comparisons use the same 4 reference checkpoints in consistent order:
**Base (`duchaitenaiart_V453`) → V5 → UnrealWorld_v1 (=V6.1) → UnrealWorld_v3**

---

### Full Portrait Comparison

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: 4-column side-by-side, same seed, same prompt -->
<!-- Columns: duchaitenaiart_V453 | CUAMIX_V5 | UnrealWorld_v1 | UnrealWorld_v3 -->
<!-- Recommended resolution: 512x768 or 768x1024 per column -->
<!-- Label each column with checkpoint name -->
```
[ INSERT: Full portrait — Base vs V5 vs UnrealWorld_v1 vs UnrealWorld_v3 ]
```

**Observation:** Progressive Weighted Sum iterations (V2→V6.1) improved anatomical coherence incrementally. The most visually distinct improvement occurred at the Add Difference stage (UnrealWorld v2→v3): contrast, facial depth-of-field, and skin micro-detail showed step-change improvement not achievable through ratio tuning alone.

---

### Facial Realism & Depth of Field

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Face crop at 100% zoom, same crop region across all 4 versions -->
<!-- Columns: duchaitenaiart_V453 | CUAMIX_V5 | UnrealWorld_v1 | UnrealWorld_v3 -->
<!-- Focus: skin texture, DoF blur on background, facial depth rendering -->
```
[ INSERT: Face crop — Base vs V5 vs UnrealWorld_v1 vs UnrealWorld_v3 ]
```

**Observation:** Add Difference injection of `cyberrealistic_v70` at multiplier 0.20 produced the clearest improvement in facial realism — skin micro-detail became visible, depth-of-field separation sharpened, and facial depth cues (nose bridge, cheekbone shadow) became more pronounced. These improvements were not observed incrementally during the Weighted Sum phase, suggesting Add Difference captures a qualitatively different region of weight space.

---

### Eye Region — Pixel-Level Detail

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Eye crop at 100% zoom, same position across all 4 versions -->
<!-- Columns: duchaitenaiart_V453 | CUAMIX_V5 | UnrealWorld_v1 | UnrealWorld_v3 -->
<!-- Focus: iris clarity, specular highlight, symmetry -->
```
[ INSERT: Eye crop — Base vs V5 vs UnrealWorld_v1 vs UnrealWorld_v3 ]
```

**Observation:** Specular highlight retention and iris boundary definition improved with Weighted Sum ratio increase (V6.1 at 0.73). Add Difference preserved these gains while adding contrast depth to the iris that Weighted Sum alone did not produce.

---

### Hand & Finger Anatomy

<!-- 📸 ADD IMAGE HERE -->
<!-- Format: Hand/finger crop at 100% zoom, same prompt and seed -->
<!-- Columns: duchaitenaiart_V453 | CUAMIX_V5 | UnrealWorld_v1 | UnrealWorld_v3 -->
<!-- Focus: digit count, finger separation, tip sharpness -->
```
[ INSERT: Hand/finger crop — Base vs V5 vs UnrealWorld_v1 vs UnrealWorld_v3 ]
```

**Observation:** Addition of `picxReal_10` at V6 stage contributed most significantly to digit count accuracy. Add Difference stage maintained these gains without regression.

---

## Key Findings

| Finding | Detail |
|---|---|
| Most impactful method | **Add Difference** — produced the largest single-step improvement in contrast, DoF, and facial realism |
| Add Difference optimal multiplier | **0.20** — sufficient for style injection; higher values introduced oversaturation artifacts |
| Optimal Weighted Sum ratio | **0.73** (V6.1) — best balance of detail retention vs style coherence in linear interpolation phase |
| Most impactful WS merge step | Addition of `picxReal_10` at V6 — largest anatomical accuracy gain within WS phase |
| Diminishing returns threshold | Weighted Sum ratios above 0.75 introduced color cast artifacts in skin tones |
| Method distinction | Weighted Sum produced incremental, continuous improvements; Add Difference produced qualitative step-change in contrast and depth rendering |
| Fixed seed validation | Confirmed improvements were model-driven, not prompt or seed variation |

---

## Discussion

The results suggest that the two interpolation methods operate on qualitatively different dimensions of the model's learned representations:

- **Weighted Sum** smoothly interpolates between two checkpoints' weight distributions — effective for fine-tuning stylistic balance and anatomical accuracy incrementally.
- **Add Difference** injects a directional delta vector into the base model — effective for introducing new perceptual properties (contrast response, depth rendering) that were absent in the base and not reachable through ratio tuning alone.

This distinction is consistent with findings in the weight-space interpolation literature, where task vector arithmetic (the conceptual basis of Add Difference) is shown to operate along semantically meaningful directions in parameter space, rather than simply blending two distributions.

An open question not explored in this study is whether **SLERP** (Spherical Linear Interpolation) — which interpolates on a hypersphere rather than linearly — would produce different trade-off curves than standard Weighted Sum, particularly in preserving fine perceptual detail at higher merge ratios.

---

## Base Models Referenced

| Model | Role |
|---|---|
| duchaitenaiart_V453 | Initial base — reference point for all comparisons |
| majicmixRealistic_v6 | Realism foundation (V3) |
| picxReal_10 | Anatomical accuracy contributor (V6, V6.1) |
| cyberrealistic_v70 | Contrast & facial realism injection via Add Difference (v2, v3) |
| cyberrealistic_v33 | Style reference (tested, not used in final chain) |
| epicphotogasm_x / lastUnicorn | Detail sharpness reference (tested, not used in final chain) |
| realisticVisionV60B1 | Secondary realism reference (tested, not used in final chain) |

---

## Repository Structure

```
.
├── README.md               ← This document
├── merge_recipe.json       ← Full merge tree with model hashes
└── comparisons/
    ├── full_portrait/      ← ADD: Full portrait 4-column comparison
    ├── face_dof/           ← ADD: Facial realism & depth-of-field crops
    ├── eye_crops/          ← ADD: Eye region pixel-level crops
    └── hand_crops/         ← ADD: Hand/finger anatomy crops
```

---

## Public Release

The final model series (UnrealWorld v1–v3) was published on CivitAI and reached **7,100+ organic downloads** from global users, providing large-scale real-world validation of the merge optimization outcomes documented here. Accumulated over **17,000,000 views** in 2026 across published works using these models.

→ [View on CivitAI](https://civitai.com/user/NxtAI)

---

## Author

**Nguyen Huu Phuc**  
B.Sc. Computer Science — Ton Duc Thang University  
[GitHub](https://github.com/huuphuc342) · [CivitAI](https://civitai.com/user/NxtAI) · [LinkedIn](https://linkedin.com/in/phucnguyenhuu-it/)
