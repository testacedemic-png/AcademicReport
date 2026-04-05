# VOID Experiment 1 Code Change Patch

This patch contains the local code changes for **Experiment 1: affected-region mask ablation**.

## Modified files
- `config/quadmask_cogvideox.py`
- `videox_fun/utils/utils.py`
- `inference/cogvideox_fun/predict_v2v.py`

## Added behavior
- configurable `mask_ablation_mode`
- support for:
  - `quadmask`
  - `primary_only`
  - `drop_affected`
  - `corrupted_affected`
- optional affected-region spatial shift
- optional affected-region dropout
- inference wiring to pass the new config fields into mask loading

## Status
- local code changes made
- syntax/build verified with `py_compile`
- full runtime execution not completed on current host due missing runtime dependencies/GPU environment
