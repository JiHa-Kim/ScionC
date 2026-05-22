# WSD Tuned Baselines

This file records the fixed-LR WSD baselines used for horizon comparisons.
The baseline is the original tuned 2k group-LR ratio, scaled by a scalar `s`
and run with the same relative WSD shape at each horizon:

- warmup: `5%` of optimizer steps
- decay: `15%` of optimizer steps
- schedule floor: `0`
- eval batches: `--eval-iters 20`
- default seeds and model settings

Base peak group LRs at `s = 1`:

| group | peak LR |
|---|---:|
| embed | `0.036793769968644335` |
| hidden | `0.025597902762094994` |
| out | `0.003499994640607136` |

## Tuned Points

| horizon | warmup | decay | tuned `s` | final val | evidence file |
|---:|---:|---:|---:|---:|---|
| 200 | 10 | 30 | `0.8000` | `1.56429` | `_local/fixed_wsd_refine200_s0p8.jsonl` |
| 400 | 20 | 60 | `0.5500` | `1.47801` | `_local/fixed_wsd_refine400_s0p55.jsonl` |
| 800 | 40 | 120 | `0.6324555320` | `1.42073` | `_local/fixed_wsd_xfer800_m0p4.jsonl` |
| 4000 | 200 | 600 | `1.0000` | `1.38187` | `_local/fixed_wsd_fit4k_s1p0.jsonl` |

The 800 scale is `0.4 * sqrt(2000 / 800)`, recorded here as the actual
scalar relative to the 2k base LRs.

## Transfer Notes

The naive inverse-square-root transfer from the 2k peak did not match these
runs. At 800 steps it predicts `s = sqrt(2000 / 800) = 1.5811`, which produced
`1.48200` in `_local/fixed_wsd_xfer800_m1.jsonl`, well behind the tuned
`s = 0.6324555320` result.

Power-law fits were also not clean:

- all tuned short-horizon points plus the 2k source point predicted 4k near
  `s = 0.91`;
- a longer-horizon fit from 400/800/2k predicted 4k near `s = 1.25`.

Both sides were checked at 4k:

| 4k `s` | final val | evidence file |
|---:|---:|---|
| `0.90` | `1.39723` | `_local/fixed_wsd_fit4k_s0p9.jsonl` |
| `1.00` | `1.38187` | `_local/fixed_wsd_fit4k_s1p0.jsonl` |
| `1.10` | `1.39091` | `_local/fixed_wsd_fit4k_s1p1.jsonl` |
| `1.25` | `1.38752` | `_local/fixed_wsd_fit4k_s1p25.jsonl` |

For the current model and WSD protocol, the practical 4k fixed baseline is the
original 2k peak scale, `s = 1.0`.

## Command Template

```powershell
$scale = 0.632455532033676
$embed = 0.036793769968644335 * $scale
$hidden = 0.025597902762094994 * $scale
$out = 0.003499994640607136 * $scale
python -m scionh.train_shakespeare `
  --mode train --no-compile --device cuda `
  --max-iters 800 --eval-interval 400 --eval-iters 20 `
  --warmup-iters 40 --decay-iters 120 `
  --lr-embed $embed --lr-hidden $hidden --lr-out $out `
  --metrics-jsonl _local/fixed_wsd_xfer800_m0p4.jsonl `
  --no-save --skip-sample
```

Use the table above to change `scale`, `max-iters`, `eval-interval`,
`warmup-iters`, `decay-iters`, and `metrics-jsonl` for the other horizons.
