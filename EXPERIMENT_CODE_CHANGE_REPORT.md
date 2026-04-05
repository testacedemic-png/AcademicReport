# VOID — Experiment Code Change Report

This report maps each suggested follow-up experiment to the likely code modifications needed in the VOID repository.

## Scope
- Paper: **VOID: Video Object and Interaction Deletion**
- Repo: `./papers/VOID_Video_Object_and_Interaction_Deletion/code`
- Goal: translate research questions into concrete implementation hooks

---

## 1. Ablate the affected-region mask

### Experiment
Compare:
- primary-object-only mask
- full quadmask
- noisy / imperfect affected-region mask

### Likely code changes
**Files and likely line ranges:**
- `VLM-MASK-REASONER/stage4_combine_masks.py` — lines **20–53**, **101–115**
- `videox_fun/utils/utils.py` — lines **374–413**
- `config/quadmask_cogvideox.py` — lines **31–49**
- new helper script under `scripts/` or `inference/`

**What to change:**
- Add mask-generation modes:
  - `primary_only`
  - `quadmask`
  - `corrupted_quadmask`
- In `stage4_combine_masks.py`, optionally suppress all `127` affected-only regions
- Add corruption transforms:
  - dilation
  - erosion
  - temporal dropout
  - random jitter
- Add a config flag such as:
  - `config.video_model.mask_ablation_mode = "quadmask"`

### Minimal implementation path
- Easiest approach: post-process `quadmask_0.mp4` into variant masks before inference

### Effort
**Low–Medium**

---

## 2. Remove the VLM reasoning stage

### Experiment
Replace Gemini-based affected-object detection with:
- no affected objects
- heuristic motion masks
- simple bbox expansion

### Likely code changes
**Files and likely line ranges:**
- `VLM-MASK-REASONER/stage2_vlm_analysis.py` — lines **256–376**, **665–740**
- `VLM-MASK-REASONER/stage3a_generate_grey_masks_v2.py` — around lines **338–383**
- `VLM-MASK-REASONER/run_pipeline.sh` — around lines **44–66**

**What to change:**
- Add alternative stage-2 backends:
  - `none`
  - `bbox_expand`
  - `motion_heuristic`
- Create synthetic `vlm_analysis.json` outputs for heuristic modes
- Modify `run_pipeline.sh` to choose stage-2 backend by CLI flag

### Minimal implementation path
- Bypass `stage2_vlm_analysis.py` entirely and emit empty `affected_objects`

### Effort
**Medium**

---

## 3. Pass-1 vs Pass-2 study

### Experiment
Compare:
- Pass 1 only
- Pass 1 + Pass 2

### Likely code changes
**Files and likely line ranges:**
- `inference/cogvideox_fun/predict_v2v.py` — lines **131–184**
- `inference/cogvideox_fun/inference_with_pass1_warped_noise.py` — lines **1–16**, **42–136**, **219–239**
- `inference/pass_2_refine.sh` — around lines **25–67**

**What to change:**
- No large code change required
- Add a wrapper script to:
  - run pass 1
  - optionally run pass 2
  - collect outputs in separate folders
- Standardize naming for easier comparison

### Minimal implementation path
- Create `scripts/run_ablation_pass1_vs_pass2.sh`

### Effort
**Low**

---

## 4. Denoising-step discrepancy study

### Experiment
Test 20 / 30 / 50 / 75 denoising steps

### Likely code changes
**Files and likely line ranges:**
- `inference/cogvideox_fun/predict_v2v.py` — lines **172–179**
- `config/quadmask_cogvideox.py` — lines **40–47**

**What to change:**
- Replace hardcoded:
  - `num_inference_steps = 30`
- with:
  - `num_inference_steps = config.video_model.num_inference_steps`
- Then sweep config values

### Minimal implementation path
- Single-line bugfix + experiment runner

### Effort
**Low**

---

## 5. Prompt sensitivity study

### Experiment
Vary:
- minimal prompts
- detailed prompts
- wrong prompts
- empty prompts

### Likely code changes
**Files and likely line ranges:**
- `videox_fun/utils/utils.py` — lines **335–337**
- input data folders containing `prompt.json`
- optional experiment runner script

**What to change:**
- No model code change necessary if editing `prompt.json`
- Optional: add prompt override CLI argument to inference scripts near input parsing and `get_video_mask_input(...)` call sites

### Minimal implementation path
- Duplicate input folders with different `prompt.json` files

### Effort
**Low**

---

## 6. Mask corruption robustness

### Experiment
Inject:
- under-mask
- over-mask
- temporal jitter
- first-frame corruption

### Likely code changes
**Files and likely line ranges:**
- `VLM-MASK-REASONER/stage4_combine_masks.py` — lines **20–53**, **101–115**
- `videox_fun/utils/utils.py` — lines **374–413**
- new utility script under `scripts/`

**What to change:**
- Add perturbation functions for mask videos:
  - morphological shrink/grow
  - framewise shift
  - random dropout
  - frame-0 overwrite
- Save perturbed variants as new quadmask files

### Minimal implementation path
- Create separate offline mask-corruption tool instead of changing inference core

### Effort
**Medium**

---

## 7. Physical interaction category breakdown

### Experiment
Split evaluation by interaction type:
- held object falls
- pushed object stays
- shadow/reflection removal
- collision cascade

### Likely code changes
**Files and likely line ranges:**
- likely no core model change
- add metadata annotation file, e.g. `experiments/interaction_labels.json`
- add evaluation scripts under `scripts/`

**What to change:**
- Build a dataset manifest with per-video labels
- Modify evaluation scripts to aggregate by category

### Minimal implementation path
- Pure evaluation-layer change

### Effort
**Low–Medium**

---

## 8. Cross-domain generalization

### Experiment
Test on different video sources/domains

### Likely code changes
**Files and likely line ranges:**
- likely none in core model
- optional input conversion script
- possibly `videox_fun/utils/utils.py` lines **338–343**, **409–413** if format-handling needs extension

**What to change:**
- Standardize new inputs to expected format:
  - `input_video.mp4`
  - `quadmask_0.mp4`
  - `prompt.json`
- Possibly add frame rate / resize preprocessing

### Minimal implementation path
- Data-format conversion only

### Effort
**Low**

---

## 9. Compare against simpler baselines

### Experiment
Compare VOID vs standard video inpainting/object-removal baselines

### Likely code changes
**Files and likely line ranges:**
- external baseline repos/scripts
- optional shared evaluation script in this repo
- minimal or no direct edits to VOID core files

**What to change:**
- Add common result export format and comparison metrics script

### Minimal implementation path
- Add `scripts/compare_baselines.py`

### Effort
**Medium**

---

## 10. Evaluate the first-frame mask heuristic

### Experiment
Ablate the code path that clears grey pixels in frame 0

### Likely code changes
**Files and likely line ranges:**
- `VLM-MASK-REASONER/stage4_combine_masks.py` — lines **101–115**

**What to change:**
- Add CLI/config flag:
  - `--clear-first-frame-grey true/false`
- Wrap the frame-0 clamp logic in a conditional

### Minimal implementation path
- Tiny edit to one file

### Effort
**Very Low**

---

## 11. Human evaluation on physical plausibility

### Experiment
Collect ratings for realism / causality / temporal consistency

### Likely code changes
**Files and likely line ranges:**
- likely no model changes
- add export script and survey generation assets

**What to change:**
- Create script to package outputs side-by-side
- Generate CSV/HTML annotation sheets

### Minimal implementation path
- New evaluation tooling only

### Effort
**Low–Medium**

---

## 12. Training ablation on quadmask representation

### Experiment
Compare:
- binary mask
- trimask
- quadmask
- quadmask without overlap value `63`

### Likely code changes
**Files and likely line ranges:**
- `scripts/cogvideox_fun/train.py` — lines **1234–1243**
- `scripts/cogvideox_fun/train_warped_noise.py` — lines **1257–1265**
- `videox_fun/utils/utils.py` — lines **374–413**
- `VLM-MASK-REASONER/stage4_combine_masks.py` — lines **20–53**
- `config/default_cogvideox.py` — mask-related config block
- `config/quadmask_cogvideox.py` — lines **33–49**

**What to change:**
- Generalize mask modes into a single enum/config
- Add conversion logic for:
  - binary
  - trimask
  - quadmask
  - quadmask-no-overlap
- Ensure train/inference data preprocessing stays aligned

### Minimal implementation path
- Add preprocessing remap before training, without changing model architecture

### Effort
**Medium–High**

---

## 13. Warped-noise training ablation

### Experiment
Compare:
- no warped noise
- warped noise only at inference
- warped noise in training + inference

### Likely code changes
**Files and likely line ranges:**
- `scripts/cogvideox_fun/train_warped_noise.py` — lines **1257–1287**, **1319–1323**, **1790–1865**
- `inference/cogvideox_fun/inference_with_pass1_warped_noise.py` — lines **75–196**, **219–239**
- `inference/cogvideox_fun/make_warped_noise.py` — likely core generation logic near file head

**What to change:**
- Mostly already supported
- Need reproducible experiment presets and logging
- Add cleaner switches for enabling/disabling warped-noise usage

### Minimal implementation path
- Script orchestration, not major architecture edits

### Effort
**Low–Medium**

---

## 14. Failure-case audit

### Experiment
Curate hard cases:
- long chains
- occlusion
- non-rigid motion
- multiple agents
- camera motion

### Likely code changes
**Files and likely line ranges:**
- probably no core code changes
- add benchmark manifest and evaluation notes

**What to change:**
- Build dataset split annotations
- Add structured logging for failure reasons

### Minimal implementation path
- Pure evaluation/reporting layer

### Effort
**Low**

---

## 15. Cost-efficiency analysis

### Experiment
Measure runtime, memory, latency, quality gain per stage

### Likely code changes
**Files and likely line ranges:**
- `inference/cogvideox_fun/predict_v2v.py` — lines **131–184** plus output-saving region below
- `inference/cogvideox_fun/inference_with_pass1_warped_noise.py` — lines **75–196**, **219–239**
- wrapper benchmark script

**What to change:**
- Add timers
- Add GPU-memory logging if available
- Save structured benchmark JSON per run

### Minimal implementation path
- Add lightweight instrumentation wrappers

### Effort
**Low**

---

# Highest-Value Code Changes to Implement First

## Tier 1 — Very high value, very cheap
1. **Fix denoising-step discrepancy**
   - File: `inference/cogvideox_fun/predict_v2v.py`
   - Lines: **172–179**
   - Replace hardcoded `30` with config value

2. **Add frame-0 heuristic toggle**
   - File: `VLM-MASK-REASONER/stage4_combine_masks.py`
   - Lines: **101–115**
   - Makes a hidden heuristic auditable

3. **Add prompt override support**
   - Files: inference scripts + `videox_fun/utils/utils.py`
   - Lines: **335–337** in prompt loader plus inference arg-parsing locations
   - Enables text-conditioning experiments quickly

## Tier 2 — Best science-per-engineering-hour
4. **Mask ablation generator**
   - Create offline script to derive primary-only / corrupted quadmask variants

5. **Pass-1 vs Pass-2 benchmark wrapper**
   - Standardizes output folders and measurements

6. **VLM bypass mode**
   - Touches `stage2_vlm_analysis.py` lines **256–376**, **665–740** and pipeline shell logic
   - Enables direct test of whether semantic reasoning is essential

## Tier 3 — More ambitious
7. **Unified mask-mode framework for training/inference**
8. **Category-aware evaluation tooling**
9. **Reusable benchmark instrumentation and reporting**

---

# Suggested New Files

To keep the repo clean, these additions would help:

- `scripts/ablate_masks.py`
- `scripts/run_step_sweep.sh`
- `scripts/run_prompt_sweep.sh`
- `scripts/run_pass1_vs_pass2.sh`
- `scripts/benchmark_void.py`
- `experiments/interaction_labels.json`
- `experiments/run_manifest.json`

---

# Summary

The easiest experiments are mostly **configuration, preprocessing, and evaluation-layer changes**, not full architecture rewrites. The most important scientific changes are:

1. making pass-1 step count configurable,
2. exposing hidden heuristics as toggles,
3. creating controlled mask ablations,
4. isolating the contribution of VLM reasoning,
5. benchmarking pass-2 refinement separately.

These would give the strongest reproducibility and ablation story with the least code risk.
