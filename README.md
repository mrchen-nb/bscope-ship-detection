# Position-aware evaluation, DoG self-labeling and radar strip attention for ship detection in shore-based radar imagery

Reproduction package for the manuscript submitted to *Engineering Applications of Artificial
Intelligence*.

This repository contains the code needed to reproduce the reported results. **The raw radar
data are not included** — they are governed by the partner surveillance site and cannot be
released. What is provided is everything required to audit the protocol and to re-derive the
reported statistics on an equivalent dataset.

---

## Contents

```
src/
  rsa_module.py              Radar Strip Attention (RSA) module, full three-component form
  rsa_module_ablation.py     Ablation variants: strip-only, gating, stripconv, range-adaptive
  attention_modules.py      Competing attention baselines: SE / CBAM / ECA / CA / SSA
  strippool_reference.py     Original Strip Pooling operator (Gao et al., CVPR 2021) used as
                             the direct same-carrier control for RSA-Strip
  rtdetr_train_rsa.py        Training entry point (RT-DETR-R50 carrier)
  fast_validate.py           Batch validation on a split, writes fast_validate_results_*.json
  validate_test.py           Per-configuration evaluation over trained checkpoints
  evaluate_splits.py         Paired statistics across the three split protocols
  anisotropy_measurement.py  Two-domain axis-anisotropy (AEI) measurement used to explain
                             why the geometric prior does not transfer to SAR imagery

configs/
  rtdetr_r50_rsa_strip.yaml      flagship configuration (CCFM x 2)
  rtdetr_r50_rsa_strippool.yaml  original Strip Pooling control, identical carrier
  rtdetr_r50_rsa_*.yaml          remaining RSA variants
  rtdetr_r50_{se,cbam,eca,ca,ssa}_*.yaml   attention baselines

scripts/
  1_make_dog_dataset.py   DoG self-labeling pipeline (two-scale smoothing, statistical
                          threshold, morphological closing, per-frame candidate budget)
  loc_split_patch.py      Position-aware split: neighbour-merging cluster on target centres
  batch_split_patch.py    Batch-time split
  run_repro_5seeds.sh     Five-seed paired training driver
  collect_results.py      Aggregate per-seed JSON into mean ± standard deviation

requirements.txt          Pinned environment
```

## Environment

The reported results were produced with:

| Component | Version |
| --- | --- |
| GPU | AMD Instinct MI300X, 192 GB, ROCm 7.2.3 |
| PyTorch | 2.11.0 |
| Ultralytics | 8.4.121 |

```bash
pip install -r requirements.txt
```

`ultralytics==8.4.121` is pinned deliberately. Absolute mAP values differ across
Ultralytics releases (up to several points between 8.4.23 and 8.4.121); only the paired
comparisons reported in the paper were produced with this exact version.

## Protocol

| Setting | Value |
| --- | --- |
| Carrier | RT-DETR-R50 |
| Split | position-disjoint (`locsplit`) unless stated otherwise |
| Epochs | 150, early-stopping patience 60 |
| Batch / input | 64 / 640 × 640 |
| Optimiser | AdamW, lr0 1e-3, cosine schedule, 3-epoch warmup |
| Mosaic | enabled for the full schedule (`close_mosaic=0`) |
| AMP | disabled (fp16 is unstable on this platform) |
| Seeds | 42–46, paired across configurations |

Reproduce the flagship configuration with five paired seeds:

```bash
bash scripts/run_repro_5seeds.sh --config rsa_strip
python src/validate_test.py --config rsa_strip
```

## What cannot be reproduced here, and why

The shore-based radar recordings are not redistributable. Consequently:

- The self-labeling pipeline can be run end to end **on a user's own recordings**, but the
  specific label set used in the paper cannot be regenerated from this repository alone.
- The split metadata (cluster assignments, seed-dependent allocation) is provided as a data
  file rather than as code, so that the reported partitions can be reconstructed exactly.
- Public-data experiments (SSDD, HRSID) run directly from the official distributions.

Per-seed metric records, cluster statistics and checksums accompany the manuscript as
supplementary material S1–S9 and are included here under `results/` after publication.

## Citation

If this code is used in your work, please cite the associated manuscript. A BibTeX entry is
provided in `CITATION.cff`.

## Licence

Code is released under the MIT licence (see `LICENSE`). The method description and figures
are covered by the manuscript's own copyright.
