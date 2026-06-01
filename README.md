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
- **4 reference checkpoints** evaluated per comparison: `Base` (duchaitenaiart_V453) → `CUAMIX_V5` → `UnrealWorld_v1` (=CUAMIX_V6.1) → `UnrealWorld_v3`
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
BASE  (duchaitenaiart_V453)
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
**Base → V5 → UnrealWorld_v1 → UnrealWorld_v3**

---

### Full Portrait Comparison

<!-- 📸 ADD IMAGE HERE -->
| Base | CUAMIX V5 | UnrealWorld v1 | UnrealWorld v3 |
|---|---|---|---|
|<img width="200"  alt="00000-3779303083" src="https://github.com/user-attachments/assets/de594cac-6539-4adc-8197-1f79bbcec22f" /> | <img width="200"  alt="00001-3779303083" src="https://github.com/user-attachments/assets/18a19d7e-5967-4a14-ae02-5c0a02ae1315" /> | <img width="200"  alt="00002-3779303083" src="https://github.com/user-attachments/assets/3c8b6526-096e-48ff-a372-813f07038363" /> | <img width="200"  alt="00003-3779303083" src="https://github.com/user-attachments/assets/ab2cf38b-0fa6-4b1c-ac64-c181dcfcf922">|
| <div align="center">WS baseline</div> | <div align="center">WS 0.40</div> | <div align="center">WS 0.73 + picxReal</div> | <div align="center">Add Diff 0.20</div> |



**Observation:** Weighted Sum phase (Base→V5) improved anatomical coherence and reduced common SD artifacts incrementally. The most significant qualitative shift occurred at UnrealWorld_v1 (CUAMIX_V6.1, WS 0.73 + picxReal_10): skin contrast improved markedly, color tone became more naturalistic, and the characteristic "plastic" appearance common in base Stable Diffusion outputs was substantially reduced. Add Difference stage (UnrealWorld_v3) further enhanced facial realism and depth-of-field separation beyond what ratio tuning alone achieved.

---

### Facial Realism & Depth of Field
| Base | CUAMIX V5 | UnrealWorld v1 | UnrealWorld v3 |
|---|---|---|---|
| <img width="200" alt="00004-493936146" src="https://github.com/user-attachments/assets/d319c006-1f9d-48a8-8c37-03f7c335463c" /> | <img width="200" alt="00005-493936146" src="https://github.com/user-attachments/assets/2f1dbdeb-47af-4f05-864b-63100afd2ba8" /> | <img width="200" alt="00006-493936146" src="https://github.com/user-attachments/assets/bb008d74-2f90-4a50-9d1f-e9a735d75294" /> | <img width="200" alt="00008-493936146" src="https://github.com/user-attachments/assets/663e8983-083f-46bd-81b9-b36ddbfc35ba" /> | 
| <div align="center">WS baseline</div> | <div align="center">WS 0.40</div> | <div align="center">WS 0.73 + picxReal</div> | <div align="center">Add Diff 0.20</div> |


**Observation:** Base model shows strong anime-influenced characteristics — overly smooth skin, plastic rendering, and unnatural facial proportions. CUAMIX V5 (WS 0.40) shifted toward human realism but retained flat skin rendering. The most significant qualitative jump occurred at UnrealWorld_v1 (WS 0.73 + picxReal_10): plastic appearance was substantially eliminated, skin tone became warmer and more naturalistic, facial depth cues (cheekbone shadow, nose bridge definition) emerged, and depth-of-field background separation became visible. UnrealWorld_v3 (Add Difference 0.20) further increased contrast, introduced skin micro-detail, and deepened DoF separation — properties not achievable through Weighted Sum ratio tuning alone.

---

### Eye Region — Pixel-Level Detail

| Base | CUAMIX V5 | UnrealWorld v1 | UnrealWorld v3 |
|---|---|---|---|
|<img width="200" alt="00009-4108508133_cr" src="https://github.com/user-attachments/assets/1ef6d1c5-8d2d-4120-a1a1-2c0af608d31a" /> | <img width="200" alt="00010-4108508133_cr" src="https://github.com/user-attachments/assets/ac451c43-6d59-4fc6-b5cf-7c8949f8b2fa" /> | <img width="200" alt="00011-4108508133_cr" src="https://github.com/user-attachments/assets/a78c2bf2-7b1b-4753-997d-bd58a84b6389" /> | <img width="200" alt="00012-4108508133_cr" src="https://github.com/user-attachments/assets/1337bd17-a711-4513-b707-6df02744e5b1" /> |
| <div align="center">WS baseline</div> | <div align="center">WS 0.40</div> | <div align="center">WS 0.73 + picxReal</div> | <div align="center">Add Diff 0.20</div> |

**Observation:** Base model exhibits anime-characteristic oversized iris with minimal specular complexity. CUAMIX V5 (WS 0.40) normalized iris proportions and introduced basic lash definition. UnrealWorld_v1 
(WS 0.73 + picxReal_10) produced the largest single improvement in eye realism — iris texture became multi-layered, specular highlights gained natural shape and depth, and individual eyelash strands became 
distinguishable. UnrealWorld_v3 (Add Difference 0.20) further increased contrast and sharpness across the eye region, with visible pore texture and natural skin micro-detail emerging around the eye area — properties absent in all Weighted Sum versions.

---

### Hand & Finger Anatomy
| Base | CUAMIX V5 | UnrealWorld v1 | UnrealWorld v3 |
|---|---|---|---|
|<img width="200" alt="00351-745118057_cr" src="https://github.com/user-attachments/assets/4ab8fcb9-f7fe-47f7-be20-f345b9d9305d" /> | <img width="200" alt="00350-745118057_cr" src="https://github.com/user-attachments/assets/55df0621-d972-415d-a36b-184c1c8d057b" /> | <img width="200" alt="00349-745118057_cr" src="https://github.com/user-attachments/assets/4f4215c7-d67d-4f3d-a677-07c0d52aa28d" /> | <img width="200" alt="00347-745118057_cr" src="https://github.com/user-attachments/assets/d88e0104-ed64-4d2d-9c9f-6eaf59a3fee4" /> |
| <div align="center">WS baseline</div> | <div align="center">WS 0.40</div> | <div align="center">WS 0.73 + picxReal</div> | <div align="center">Add Diff 0.20</div> |



**Observation:** Base model rendered 5 digits with flat, plastic skin and minimal surface detail. CUAMIX V5 (WS 0.40) and UnrealWorld_v1 (WS 0.73 + picxReal_10) both exhibited a missing finger artifact — only 4 digits clearly rendered — suggesting that Weighted Sum interpolation at these ratios introduced anatomical regression in digit count despite improving skin tone and surface quality. UnrealWorld_v3 (Add Difference 0.20) restored correct 5-finger anatomy while simultaneously producing the strongest surface detail across all versions — visible knuckle definition, subtle vein structure, and natural skin contrast. This result suggests Add Difference injection operates on a qualitatively different region of weight space, recovering anatomical accuracy that Weighted Sum phase had degraded.


---

## Key Findings

| Finding | Detail |
|---|---|
| Most impactful single step | **UnrealWorld_v1 (WS 0.73 + picxReal_10)** — largest qualitative jump within Weighted Sum phase: eliminated plastic skin rendering, restored natural skin tone, introduced facial depth cues and DoF separation |
| Most impactful method | **Add Difference (0.20)** — produced step-change in contrast, skin micro-detail, pore texture, and eye region sharpness not achievable through ratio tuning alone |
| Unexpected WS regression | CUAMIX V5 and UnrealWorld_v1 both exhibited **missing finger artifact** (4 digits instead of 5) — Weighted Sum interpolation improved surface quality while simultaneously degrading digit count accuracy |
| Add Difference recovery | UnrealWorld_v3 (Add Diff 0.20) **restored correct 5-finger anatomy** while producing strongest surface detail — suggests Add Difference operates on qualitatively different weight-space dimensions than Weighted Sum |
| Add Difference optimal multiplier | **0.20** — sufficient for contrast and detail injection; higher values risk oversaturation in skin tones |
| Plastic appearance elimination | Characteristic SD "plastic" skin rendering was substantially reduced at UnrealWorld_v1 and fully resolved at UnrealWorld_v3 |
| Fixed seed validation | Same seed across all checkpoints confirmed improvements are model-driven, not prompt or seed variation |
---

## Discussion

Three parallel comparisons — full portrait, eye region, and hand anatomy 
— reveal a consistent pattern across all evaluated regions:

**Weighted Sum phase (Base → V5 → UnrealWorld_v1)** produced continuous, 
incremental improvements in skin tone naturalness, facial proportions, 
and stylistic realism. However, this phase also introduced an unexpected 
anatomical regression: both V5 and UnrealWorld_v1 rendered only 4 fingers 
instead of 5, suggesting that linear weight interpolation at these ratios 
degraded digit count accuracy despite improving surface quality. This 
represents a classic property trade-off — consistent with findings in 
weight-space interpolation literature where no single λ universally 
optimizes across all output dimensions simultaneously.

**Add Difference phase (UnrealWorld_v3)** produced qualitatively distinct 
improvements across all three evaluated regions: skin micro-detail and 
pore texture in facial rendering, iris contrast and eye area depth, and 
restoration of correct finger anatomy alongside stronger surface detail. 
These improvements were not reachable through Weighted Sum ratio tuning 
alone — supporting the interpretation that Add Difference, as a directional 
delta injection rather than a linear blend, operates along semantically 
distinct directions in weight space. This is analogous to task vector 
arithmetic in the NLP literature, where directional parameter deltas have 
been shown to preserve or restore model properties that interpolation 
degrades.

**Why Add Difference recovers what Weighted Sum degrades:**

Weighted Sum at any ratio λ produces a linear blend of two weight 
distributions — the result is a weighted average, which inherently 
smooths and compresses the extremes of both models. When applied 
iteratively, this progressive blending can degrade specific encoded 
representations (such as finger anatomy) even while improving others 
(such as skin tone). There is no mechanism within Weighted Sum to 
selectively preserve or restore a degraded property.

Add Difference operates differently. The vector `(B - C) × multiplier` 
encodes a **directional delta** — specifically, what model B has that 
model C does not. 
In this study: 

`result = UnrealWorld_v1 + (cyberrealistic_v70 - UnrealWorld_v1) × 0.20.`

This delta captured contrast response, skin micro-detail, and anatomical 
detail present in `cyberrealistic_v70` but absent or degraded in 
`UnrealWorld_v1`. Injecting 0.20 of this vector transferred those 
properties without re-blending the entire weight distribution — which 
is why finger anatomy was restored while surface quality gains from the 
Weighted Sum phase were preserved rather than overwritten.

This mechanism is conceptually identical to task vector arithmetic 
(Ilharco et al., 2023), referenced in Farn et al. (EMNLP 2025): 
directional parameter deltas encode specific capabilities along 
semantically meaningful directions in weight space, enabling targeted 
property injection rather than global distribution blending.

An open question not explored in this study is whether **SLERP** 
(Spherical Linear Interpolation) would produce different trade-off curves 
— particularly whether it could preserve anatomical accuracy (digit count) 
during the ratio-increase phase where standard Weighted Sum introduced 
regression.

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

## Public Release

The final model series (UnrealWorld v1–v3) was published on CivitAI and reached **7,100+ organic downloads** from global users, providing large-scale real-world validation of the merge optimization outcomes documented here. Accumulated over **17,000,000 views** in 2026 across published works using these models.


---

## Author
**Nguyen Huu Phuc**  
